# 局部更新文档

如图所示，update API 结合了先前说明的读取和写入模式。

![局部更新文档](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/elas_0404.png?raw=true)

以下是部分更新一个文档的步骤：

1. 客户端向 Node 1 发送更新请求。

1. 它将请求转发到主分片所在的 Node 3 。

1. Node 3 从主分片检索文档，修改 _source 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试步骤 3 ，超过 retry_on_conflict 次后放弃。

1. 如果 Node 3 成功地更新文档，它将新版本的文档并行转发到 Node 1 和 Node 2 上的副本分片，重新建立索引。 一旦所有副本分片都返回成功， Node 3 向协调节点也返回成功，协调节点向客户端返回成功。

update API 还接受在 新建、索引和删除文档 章节中介绍的 routing 、 replication 、 consistency 和 timeout 参数。

> **基于文档的复制**
> 
> 当主分片把更改转发到副本分片时， 它不会转发更新请求。 相反，它转发完整文档的新版本。请记住，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。 如果Elasticsearch仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档。

