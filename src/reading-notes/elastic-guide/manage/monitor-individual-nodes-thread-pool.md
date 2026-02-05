# 线程池部分

Elasticsearch 在内部维护了线程池。这些线程池相互协作完成任务，有必要的话相互间还会传递任务。通常来说，你不需要配置或者调优线程池，不过查看它们的统计值有时候还是有用的，可以洞察你的集群表现如何。

这有一系列的线程池，但以相同的格式输出：

```json

"thread_pool": {
        "analyze": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "auto_complete": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "azure_event_loop": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "ccr": {
          "threads": 5,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 5,
          "completed": 5
        },
        "cluster_coordination": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 345064
        },
        "fetch_shard_started": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "fetch_shard_store": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 4,
          "completed": 166
        },
        "flush": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 1576
        },
        "force_merge": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 13
        },
        "generic": {
          "threads": 17,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 17,
          "completed": 6725195
        },
        "get": {
          "threads": 2,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 2,
          "completed": 11157
        },
        "management": {
          "threads": 2,
          "queue": 0,
          "active": 1,
          "rejected": 0,
          "largest": 2,
          "completed": 13397354
        },
        "ml_datafeed": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "ml_job_comms": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "ml_utility": {
          "threads": 3,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 3,
          "completed": 1765085
        },
        "refresh": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 35331528
        },
        "repository_azure": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "rollup_indexing": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "search": {
          "threads": 4,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 4,
          "completed": 2424657
        },
        "search_coordination": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 394423
        },
        "search_throttled": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "searchable_snapshots_cache_fetch_async": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "searchable_snapshots_cache_prewarming": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "security-crypto": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 48
        },
        "security-token-key": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "snapshot": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 259458
        },
        "snapshot_meta": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 6,
          "completed": 44161
        },
        "system_critical_read": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 1110706
        },
        "system_critical_write": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 431
        },
        "system_read": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 3467466
        },
        "system_write": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 1064432
        },
        "vector_tile_generation": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "warmer": {
          "threads": 1,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 1,
          "completed": 7231797
        },
        "watcher": {
          "threads": 0,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 0,
          "completed": 0
        },
        "write": {
          "threads": 2,
          "queue": 0,
          "active": 0,
          "rejected": 0,
          "largest": 2,
          "completed": 2219375
        }
      },

```

每个线程池会列出已配置的线程数量（ threads ），当前在处理任务的线程数量（ active ），以及在队列中等待处理的任务单元数量（ queue ）。

如果队列中任务单元数达到了极限，新的任务单元会开始被拒绝，你会在 rejected 统计值上看到它反映出来。这通常是你的集群在某些资源上碰到瓶颈的信号。因为队列满意味着你的节点或集群在用最高速度运行，但依然跟不上工作的蜂拥而入。

> **批量操作的被拒绝数**
>
>如果你碰到了队列被拒，一般来说都是批量索引请求导致的。通过并发导入程序发送大量批量请求非常简单。越多越好嘛，对不？
>
>事实上，每个集群都有它能处理的请求上限。一旦这个阈值被超过，队列会很快塞满，然后新的批量请求就被拒绝了。
>
>这是一件 好事情 。队列的拒绝在回压方面是有用的。它们让你知道你的集群已经在最大容量了。这比把数据塞进内存队列要来得好。增加队列大小并不能增加性能，它只是隐藏了问题。当你的集群只能每秒钟处理 10000 个文档的时候，无论队列是 100 还是 10000000 都没关系——你的集群还是只能每秒处理 10000 个文档。
>
>队列只是隐藏了性能问题，而且带来的是真实的数据丢失的风险。在队列里的数据都是还没处理的，如果节点挂掉，这些请求都会永久的丢失。此外，队列还要消耗大量内存，这也是不理想的。
>
>在你的应用中，优雅的处理来自满载队列的回压，才是更好的选择。当你收到拒绝响应的时候，你应该采取如下几步：
>
>1. 暂停导入线程 3–5 秒。
>1. 从批量操作的响应里提取出来被拒绝的操作。因为可能很多操作还是成功的。响应会告诉你哪些成功，哪些被拒绝了。
>1. 发送一个新的批量请求，只包含这些被拒绝过的操作。
>1. 如果依然碰到拒绝，再次从步骤 1 开始。
>
>通过这个流程，你的代码可以很自然的适应你集群的负载，做到自动回压。
>
>拒绝不是错误：它们只是意味着你要稍后重试。

这里的一系列的线程池，大多数你可以忽略，但是有一小部分还是值得关注的：

1. indexing 普通的索引请求的线程池

1. bulk 批量请求，和单条的索引请求不同的线程池

1. get Get-by-ID 操作

1. search 所有的搜索和查询请求

1. merging 专用于管理 Lucene 合并的线程池





