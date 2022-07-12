# 添加索引

我们往 Elasticsearch 添加数据时需要用到 索引 —— 保存相关数据的地方。 索引实际上是指向一个或者多个物理 分片 的 逻辑命名空间 。

一个 分片 是一个底层的 工作单元 ，它仅保存了全部数据中的一部分。 在分片内部机制中，我们将详细介绍分片是如何工作的，而现在我们只需知道一个分片是一个 Lucene 的实例，以及它本身就是一个完整的搜索引擎。 我们的文档被存储和索引到分片内，但是应用程序是直接与索引而不是与分片进行交互。

Elasticsearch 是利用分片将数据分发到集群内各处的。分片是数据的容器，文档保存在分片内，分片又被分配到集群内的各个节点里。 当你的集群规模扩大或者缩小时， Elasticsearch 会自动的在各节点中迁移分片，使得数据仍然均匀分布在集群里。

一个分片可以是 主 分片或者 副本 分片。 索引内任意一个文档都归属于一个主分片，所以主分片的数目决定着索引能够保存的最大数据量。

> **🦉 Note**
>
> 技术上来说，一个主分片最大能够存储 Integer.MAX_VALUE - 128 个文档，但是实际最大值还需要参考你的使用场景：包括你使用的硬件， 文档的大小和复杂程度，索引和查询文档的方式以及你期望的响应时长。


一个副本分片只是一个主分片的拷贝。副本分片作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。

在索引建立的时候就已经确定了主分片数，但是副本分片数可以随时修改。

让我们在包含一个空节点的集群内创建名为 blogs 的索引。 索引在默认情况下会被分配5个主分片， 但是为了演示目的，我们将分配3个主分片和一份副本（每个主分片拥有一个副本分片）：

```json

PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}

{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "blogs"
}

```

我们的集群现在, 所有3个主分片都被分配在 Node 1 。

![3个分片在一个节点](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/elas_0202.png?raw=true)


⚠️ 最新版本中，创建索引默认主分片为1，副本也是1，下面是elastic cloud示例

```json

PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 2
   }
}

{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "blogs"
}

```

![3个分片在2个节点](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/basic/es-basic-create-an-index.jpg?raw=true)

如果我们现在查看集群健康，我们将看到如下内容：

```json

GET _cluster/health

{
  "cluster_name": "a2ff16d9aa2645dc87ab1714e6e16a84",
  "status": "yellow", 0️⃣
  "timed_out": false,
  "number_of_nodes": 3,
  "number_of_data_nodes": 2,
  "active_primary_shards": 95,
  "active_shards": 190,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 3, 1️⃣
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 98.44559585492227
}

```

* 0️⃣ 集群 status 值为 yellow

* 1️⃣ 没有被分配到任何节点的副本数

集群的健康状况为 yellow 则表示全部 主 分片都正常运行（集群可以正常服务所有请求），但是 副本 分片没有全部处在正常状态。 实际上，所有3个副本分片都是 unassigned —— 它们都没有被分配到任何节点。 在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上的所有副本数据。

当前我们的集群是正常运行的，但是在硬件故障时有丢失数据的风险。
