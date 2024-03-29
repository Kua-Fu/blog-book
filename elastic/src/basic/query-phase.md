# 查询阶段

在初始 查询阶段 时， 查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 每个分片在本地执行搜索并构建一个匹配文档的 优先队列。

> **优先队列**
>
>一个 优先队列 仅仅是一个存有 top-n 匹配文档的有序列表。
>
>优先队列的大小取决于分页参数 from 和 size 。
>
>例如，如下搜索请求将需要足够大的优先队列来放入100条文档。
> ```
> GET /_search
> {
>   "from": 90,
>   "size": 10
> }
> ```

这个查询阶段的过程如图 “查询过程分布式搜索” 所示。

![查询过程分布式搜索](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/elas_0901.png?raw=true)

查询阶段包含以下三个步骤:

1. 客户端发送一个 search 请求到 Node 3 ， Node 3 会创建一个大小为 from + size 的空优先队列。

1. Node 3 将查询请求转发到索引的每个主分片或副本分片中。每个分片在本地执行查询并添加结果到大小为 from + size 的本地有序优先队列中。

1. 每个分片返回各自优先队列中所有文档的 ID 和排序值给协调节点，也就是 Node 3 ，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。

当一个搜索请求被发送到某个节点时，这个节点就变成了协调节点。 这个节点的任务是广播查询请求到所有相关分片并将它们的响应整合成全局排序后的结果集合，这个结果集合会返回给客户端。

第一步是广播请求到索引中每一个节点的分片拷贝。就像 document GET requests 所描述的， 查询请求可以被某个主分片或某个副本分片处理， 这就是为什么更多的副本（当结合更多的硬件）能够增加搜索吞吐率。 协调节点将在之后的请求中轮询所有的分片拷贝来分摊负载。

每个分片在本地执行查询请求并且创建一个长度为 from + size 的优先队列—也就是说，每个分片创建的结果集足够大，均可以满足全局的搜索请求。 分片返回一个轻量级的结果列表到协调节点，它仅包含文档 ID 集合以及任何排序需要用到的值，例如 _score 。

协调节点将这些分片级的结果合并到自己的有序优先队列里，它代表了全局排序结果集合。至此查询过程结束。

> 🦉**Note**
>
> 一个索引可以由一个或几个主分片组成， 所以一个针对单个索引的搜索请求需要能够把来自多个分片的结果组合起来。
>
> 针对 multiple 或者 all 索引的搜索工作方式也是完全一致的—​仅仅是包含了更多的分片而已。
