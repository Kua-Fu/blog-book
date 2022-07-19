# 等待中的任务

有一些任务只能由主节点去处理，比如创建一个新的索引或者在集群中移动分片。由于一个集群中只能有一个主节点，所以只有这一节点可以处理集群级别的元数据变动。在 99.9999% 的时间里，这不会有什么问题。元数据变动的队列基本上保持为零。

在一些 罕见 的集群里，元数据变动的次数比主节点能处理的还快。这会导致等待中的操作会累积成队列。

等待中的任务 API 会给你展示队列中（如果有的话）等待的集群级别的元数据变更操作：

```json
GET _cluster/pending_tasks

{
   "tasks": []
}

```

通常，响应都是像这样的, 这意味着没有等待中的任务。如果你有一个罕见的集群在主节点出现瓶颈了，等待中的任务列表可能会像这样：

```json

{
   "tasks": [
      {
         "insert_order": 101,
         "priority": "URGENT",
         "source": "create-index [foo_9], cause [api]",
         "time_in_queue_millis": 86,
         "time_in_queue": "86ms"
      },
      {
         "insert_order": 46,
         "priority": "HIGH",
         "source": "shard-started ([foo_2][1], node[tMTocMvQQgGCkj7QDHl3OA], [P],
         s[INITIALIZING]), reason [after recovery from gateway]",
         "time_in_queue_millis": 842,
         "time_in_queue": "842ms"
      },
      {
         "insert_order": 45,
         "priority": "HIGH",
         "source": "shard-started ([foo_2][0], node[tMTocMvQQgGCkj7QDHl3OA], [P],
         s[INITIALIZING]), reason [after recovery from gateway]",
         "time_in_queue_millis": 858,
         "time_in_queue": "858ms"
      }
  ]
}

```

可以看到任务都被指派了优先级（ 比如说 URGENT 要比 HIGH 更早的处理 ），任务插入的次序、操作进入队列多久，以及打算处理什么。在上面的列表中，有一个 创建索引(create-index) 和两个 启动分片(shard-started) 的操作在等待。

> **什么时候应该担心等待中的任务?**
>
>就像曾经提到过的，主节点很少会成为集群的瓶颈。唯一可能成为瓶颈的是集群状态非常大 而且 更新频繁。
>
>例如，如果你允许客户按照他们的意愿创建任意的动态字段，而且每个客户每天都有一个独立索引，那么你的集群状态会变得非常大。集群状态包括 ( 但不限于 ) 所有索引及其类型，以及每个索引的全部字段。
>
>所以如果你有 100000 客户，然后每个客户平均有 1000 个字段，而且数据有 90 天的保留期—这就有九十亿个字段需要保存在集群状态中。不管它何时发生变更，所有的节点都需要被通知。
>
>主节点必须处理这些变动，这需要不小的 CPU 开销，加上推送更新的集群状态到所有节点的网络开销。
>
>这就是那些可以看到集群状态操作队列上涨的集群。没有简单的办法可以解决这个问题，不过你有三个选择：
>
>1. 使用一个更强大的主节点。不幸的是，这种垂直扩展只是延迟这种必然结果出现而已。
>
>1. 通过某些方式限定文档的动态性质来限制集群状态的大小。
>
>1. 到达某个阈值后组建另外一个集群。
