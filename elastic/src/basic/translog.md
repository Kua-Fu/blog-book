# 持久化变更

如果没有用 fsync 把数据从文件系统缓存刷（flush）到硬盘，我们不能保证数据在断电甚至是程序正常退出之后依然存在。为了保证 Elasticsearch 的可靠性，需要确保数据变化被持久化到磁盘。

在 动态更新索引，我们说一次完整的提交会将段刷到磁盘，并写入一个包含所有段列表的提交点。Elasticsearch 在启动或重新打开一个索引的过程中使用这个提交点来判断哪些段隶属于当前分片。

即使通过每秒刷新（refresh）实现了近实时搜索，我们仍然需要经常进行完整提交来确保能从失败中恢复。但在两次提交之间发生变化的文档怎么办？我们也不希望丢失掉这些数据。

Elasticsearch 增加了一个 translog ，或者叫事务日志，在每一次对 Elasticsearch 进行操作时均进行了日志记录。通过 translog ，整个流程看起来是下面这样：

1. 一个文档被索引之后，就会被添加到内存缓冲区，并且 追加到了 translog ，正如下图, “新的文档被添加到内存缓冲区并且被追加到了事务日志” 描述的一样。

	![新的文档被添加到内存缓冲区并且被追加到了事务日志](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/elas_1106.png?raw=true)

1. 刷新（refresh）使分片处于 “刷新（refresh）完成后, 缓存被清空但是事务日志不会” 描述的状态，分片每秒被刷新（refresh）一次：

	* 这些在内存缓冲区的文档被写入到一个新的段中，且没有进行 fsync 操作。

	* 这个段被打开，使其可被搜索。
	
	* 内存缓冲区被清空。
	
	![刷新（refresh）完成后, 缓存被清空但是事务日志不会](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/elas_1107.png?raw=true)
	
1. 这个进程继续工作，更多的文档被添加到内存缓冲区和追加到事务日志（“事务日志不断积累文档” ）

	![事务日志不断积累文档](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/elas_1108.png?raw=true)

1. 每隔一段时间—​例如 translog 变得越来越大—​索引被刷新（flush）；一个新的 translog 被创建，并且一个全量提交被执行（“在刷新（flush）之后，段被全量提交，并且事务日志被清空” ）：

	* 所有在内存缓冲区的文档都被写入一个新的段。
	
	* 缓冲区被清空。
	
	* 一个提交点被写入硬盘。
	
	* 文件系统缓存通过 fsync 被刷新（flush）。
	
	* 老的 translog 被删除。
	
	![在刷新（flush）之后，段被全量提交，并且事务日志被清空](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/elas_1109.png?raw=true)
	
translog 提供所有还没有被刷到磁盘的操作的一个持久化纪录。当 Elasticsearch 启动的时候， 它会从磁盘中使用最后一个提交点去恢复已知的段，并且会重放 translog 中所有在最后一次提交后发生的变更操作。

translog 也被用来提供实时 CRUD 。当你试着通过ID查询、更新、删除一个文档，它会在尝试从相应的段中检索之前， 首先检查 translog 任何最近的变更。这意味着它总是能够实时地获取到文档的最新版本。

## flush API

这个执行一个提交并且截断 translog 的行为在 Elasticsearch 被称作一次 flush 。 分片每30分钟被自动刷新（flush），或者在 translog 太大的时候也会刷新。请查看 translog 文档 来设置，它可以用来 控制这些阈值：

flush API 可以被用来执行一个手工的刷新（flush）:

```json

POST /blogs/_flush 0️⃣

POST /_flush?wait_if_ongoing 1️⃣

```

0️⃣ 刷新（flush） blogs 索引
	
1️⃣ 刷新（flush）所有的索引并且并且等待所有刷新在返回前完成。

你很少需要自己手动执行 flush 操作；通常情况下，自动刷新就足够了。

这就是说，在重启节点或关闭索引之前执行 flush 有益于你的索引。当 Elasticsearch 尝试恢复或重新打开一个索引， 它需要重放 translog 中所有的操作，所以如果日志越短，恢复越快。

> Translog 有多安全?
>
> translog 的目的是保证操作不会丢失。这引出了这个问题： Translog 有多安全？
>
>在文件被 fsync 到磁盘前，被写入的文件在重启之后就会丢失。默认 translog 是每 5 秒被 fsync 刷新到硬盘， 或者在每次写请求完成之后执行(e.g. index, delete, update, bulk)。这个过程在主分片和复制分片都会发生。最终， 基本上，这意味着在整个请求被 fsync 到主分片和复制分片的translog之前，你的客户端不会得到一个 200 OK 响应。
>
>在每次请求后都执行一个 fsync 会带来一些性能损失，尽管实践表明这种损失相对较小（特别是bulk导入，它在一次请求中平摊了大量文档的开销）。
>
>但是对于一些大容量的偶尔丢失几秒数据问题也并不严重的集群，使用异步的 fsync 还是比较有益的。比如，写入的数据被缓存到内存中，再每5秒执行一次 fsync 。
>
>这个行为可以通过设置 durability 参数为 async 来启用：
> ```
>PUT /my_index/_settings
>{
>    "index.translog.durability": "async",
>    "index.translog.sync_interval": "5s"
>}
> ```
>
>这个选项可以针对索引单独设置，并且可以动态进行修改。如果你决定使用异步 translog 的话，你需要 保证 在发生crash时，丢失掉 sync_interval 时间段的数据也无所谓。请在决定前知晓这个特性。
>
>如果你不确定这个行为的后果，最好是使用默认的参数（ "index.translog.durability": "request" ）来避免数据丢失。
