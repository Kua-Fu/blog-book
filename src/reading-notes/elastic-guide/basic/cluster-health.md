# 集群健康

Elasticsearch 的集群监控信息中包含了许多的统计数据，其中最为重要的一项就是 集群健康 ， 它在 status 字段中展示为 green 、 yellow 或者 red 。

```json

GET /_cluster/health

{
  "cluster_name": "a2ff16d9aa2645dc87ab1714e6e16a84",
  "status": "green",  0️⃣
  "timed_out": false,
  "number_of_nodes": 3,
  "number_of_data_nodes": 2,
  "active_primary_shards": 91,
  "active_shards": 182,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}

```

 0️⃣ status 字段是我们最关心的
 
 status 字段指示着当前集群在总体上是否工作正常。它的三种颜色含义如下：
 
 * green 所有的主分片和副本分片都正常运行
 
 * yellow 所有的主分片都正常运行，但不是所有的副本分片都正常运行
 
 * red 有主分片没能正常运行
 
 在本章节剩余的部分，我们将解释什么是 主 分片和 副本 分片，以及上面提到的这些颜色的实际意义。
