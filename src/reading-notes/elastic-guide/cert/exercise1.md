# 真题1

## 一、参考

> [真题 1 详解](https://www.yuque.com/deep_elasticsearch/tzcm9n/wkimk6)

## 二、集群健康状态

主要是把集群从yellow 状态变为 green, 方法是修改副本数量

[解决集群变红或者变黄的问题](https://www.yuque.com/deep_elasticsearch/tzcm9n/zc45pu#af6f4cff)

```
GET _cat/indices?v&health=yellow

health status index         uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   yztest-1      BdTfAzOSRJm_YSecM0zoyg   1   1          1            0      5.2kb          5.2kb
yellow open   yztest-1-dest uPvZY-eiSeSgGtdRz5pHIQ   1   1          1            0      5.7kb          5.7kb


PUT yztest-1,yztest-1-dest/_settings
{
  "number_of_replicas": 0
}


GET _cluster/health

{
  "cluster_name" : "elasticsearch",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 13,
  "active_shards" : 13,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

```
