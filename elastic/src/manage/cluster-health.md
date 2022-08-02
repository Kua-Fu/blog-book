# 集群健康

一个 Elasticsearch 集群至少包括一个节点和一个索引。或者它可能有一百个数据节点、三个单独的主节点，以及一小打客户端节点——这些共同操作一千个索引（以及上万个分片）。

不管集群扩展到多大规模，你都会想要一个快速获取集群状态的途径。Cluster Health API 充当的就是这个角色。你可以把它想象成是在一万英尺的高度鸟瞰集群。它可以告诉你安心吧一切都好，或者警告你集群某个地方有问题。

让我们执行一下 cluster-health API 然后看看响应体是什么样子的：

```json

GET _cluster/health

{
  "cluster_name": "a2ff16d9aa2645dc87ab1714e6e16a84",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 3,
  "number_of_data_nodes": 2,
  "active_primary_shards": 97,
  "active_shards": 194,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 3,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 98.47715736040608
}

```

和 Elasticsearch 里其他 API 一样，cluster-health 会返回一个 JSON 响应。这对自动化和告警系统来说，非常便于解析。响应中包含了和你集群有关的一些关键信息：

1. green 所有的主分片和副本分片都已分配。你的集群是 100% 可用的。

1. yellow 所有的主分片已经分片了，但至少还有一个副本是缺失的。不会有数据丢失，所以搜索结果依然是完整的。不过，你的高可用性在某种程度上被弱化。如果 更多的 分片消失，你就会丢数据了。把 yellow 想象成一个需要及时调查的警告。

1. red 至少一个主分片（以及它的全部副本）都在缺失中。这意味着你在缺少数据：搜索只能返回部分数据，而分配到这个分片上的写入请求会返回一个异常。

green/yellow/red 状态是一个概览你的集群并了解眼下正在发生什么的好办法。剩下来的指标给你列出来集群的状态概要：

* number_of_nodes 和 number_of_data_nodes 这个命名完全是自描述的。

* active_primary_shards 指出你集群中的主分片数量。这是涵盖了所有索引的汇总值。

* active_shards 是涵盖了所有索引的_所有_分片的汇总值，即包括副本分片。

* relocating_shards 显示当前正在从一个节点迁往其他节点的分片的数量。通常来说应该是 0，不过在 Elasticsearch 发现集群不太均衡时，该值会上涨。比如说：添加了一个新节点，或者下线了一个节点。

* initializing_shards 是刚刚创建的分片的个数。比如，当你刚创建第一个索引，分片都会短暂的处于 initializing 状态。这通常会是一个临时事件，分片不应该长期停留在 initializing 状态。你还可能在节点刚重启的时候看到 initializing 分片：当分片从磁盘上加载后，它们会从 initializing 状态开始。

* unassigned_shards 是已经在集群状态中存在的分片，但是实际在集群里又找不着。通常未分配分片的来源是未分配的副本。比如，一个有 5 分片和 1 副本的索引，在单节点集群上，就会有 5 个未分配副本分片。如果你的集群是 red 状态，也会长期保有未分配分片（因为缺少主分片）。

## 钻更深点：找到问题索引

想象一下某天碰到问题了， 而你发现你的集群健康状态看起来像是这样：

```json

{
   "cluster_name": "elasticsearch_zach",
   "status": "red",
   "timed_out": false,
   "number_of_nodes": 8,
   "number_of_data_nodes": 8,
   "active_primary_shards": 90,
   "active_shards": 180,
   "relocating_shards": 0,
   "initializing_shards": 0,
   "unassigned_shards": 20
}

```

好了，从这个健康状态里我们能推断出什么来？嗯，我们集群是 red ，意味着我们缺数据（主分片 + 副本分片）了。我们知道我们集群原先有 10 个节点，但是在这个健康状态里列出来的只有 8 个数据节点。有两个数据节点不见了。我们看到有 20 个未分配分片。

这就是我们能收集到的全部信息。那些缺失分片的情况依然是个谜。我们是缺了 20 个索引，每个索引里少 1 个主分片？还是缺 1 个索引里的 20 个主分片？还是 10 个索引里的各 1 主 1 副本分片？具体是哪个索引？


要回答这个问题，我们需要使用 level 参数让 cluster-health 答出更多一点的信息：

```json

GET _cluster/health?level=indices

{
  "cluster_name": "a2ff16d9aa2645dc87ab1714e6e16a84",
  "status": "yellow",
  "timed_out": false,
  "number_of_nodes": 3,
  "number_of_data_nodes": 2,
  "active_primary_shards": 97,
  "active_shards": 194,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 3,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 98.47715736040608,
  "indices": {
    ".ent-search-actastic-app_search_document_positions_v3": {
      "status": "green",
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "active_primary_shards": 1,
      "active_shards": 2,
      "relocating_shards": 0,
      "initializing_shards": 0,
      "unassigned_shards": 0
    }
  }
}

```

这个参数会让 cluster-health API 在我们的集群信息里添加一个索引清单，以及有关每个索引的细节（状态、分片数、未分配分片数等等）

一旦我们询问要索引的输出，哪个索引有问题立马就很清楚了：v2 索引。我们还可以看到这个索引曾经有 10 个主分片和一个副本，而现在这 20 个分片全不见了。可以推测，这 20 个索引就是位于从我们集群里不见了的那两个节点上。

level 参数还可以接受其他更多选项：

```json

GET _cluster/health?level=shards

```

shards 选项会提供一个详细得多的输出，列出每个索引里每个分片的状态和位置。这个输出有时候很有用，但是由于太过详细会比较难用。如果你知道哪个索引有问题了，本章讨论的其他 API 显得更加有用一点。

## 阻塞等待状态变化

当构建单元和集成测试时，或者实现和 Elasticsearch 相关的自动化脚本时，cluster-health API 还有另一个小技巧非常有用。你可以指定一个 wait_for_status 参数，它只有在状态达标之后才会返回。比如：

```json

GET _cluster/health?wait_for_status=green

```

这个调用会 阻塞 （不给你的程序返回控制权）住直到 cluster-health 变成 green ，也就是说所有主分片和副本分片都分配下去了。这对自动化脚本和测试非常重要。

如果你创建一个索引，Elasticsearch 必须在集群状态中向所有节点广播这个变更。那些节点必须初始化这些新分片，然后响应给主节点说这些分片已经 Started 。这个过程很快，但是因为网络延迟，可能要花 10–20ms。

如果你有个自动化脚本是 (a) 创建一个索引然后 (b) 立刻写入一个文档，这个操作会失败。因为索引还没完全初始化完成。在 (a) 和 (b) 两步之间的时间可能不到 1ms —— 对网络延迟来说这可不够。

比起使用 sleep 命令，直接让你的脚本或者测试使用 wait_for_status 参数调用 cluster-health 更好。当索引完全创建好，cluster-health 就会变成 green ，然后这个调用就会把控制权交还给你的脚本，然后你就可以开始写入了。

有效的选项是： green 、 yellow 和 red 。这个调回会在达到你要求（或者『更高』）的状态时返回。比如，如果你要求的是 yellow ，状态变成 yellow 或者 green 都会打开调用。
