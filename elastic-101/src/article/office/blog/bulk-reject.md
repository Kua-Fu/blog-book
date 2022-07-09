# bulk 被拒绝

[原文: Why am I seeing bulk rejections in my Elasticsearch cluster?](https://www.elastic.co/cn/blog/why-am-i-seeing-bulk-rejections-in-my-elasticsearch-cluster)

Elasticsearch supports a wide range of use-cases across our user base, and more and more of these rely on fast indexing to quickly get large amounts of data into Elasticsearch. Even though Elasticsearch is fast and index performance is continually improved, it is still possible to overwhelm it. At that point you typically see parts of bulk requests getting rejected. In this blog post we will look at the causes and how to avoid it.

在我们的用户群了 elasticsearch 有广泛的使用管理，越来越多的客户依赖大量数据的快速索引。尽管 elasticsearch 速度很快，索引性能也在不断提高，仍然可能达到峰值。此时，通常可以看到部分写入请求被拒绝。下文将探讨原因和如何避免请求被拒绝。

This is the second installment in a series of blog posts where we look at and discuss your common questions. The first installment discussed and provided guidelines around "How many shards one should aim to have in an Elasticsearch cluster?"

这是一系列文章中的第二篇。第一部分文章可以详见 [How many shards one should aim to have in an Elasticsearch cluster?](https://www.elastic.co/cn/blog/how-many-shards-should-i-have-in-my-elasticsearch-cluster)

## What happens when a bulk indexing request is sent to Elasticsearch?

当一个 bulk 请求发送到elasticsearch, 请求后会发生什么?

Let’s start at the beginning and look at what happens behind the scenes when a bulk indexing request is sent to Elasticsearch.

让我们从头开始，看看当批量索引请求发送到 elasticsearch 时候，后台会发生什么?

When a bulk request arrives at a node in the cluster, it is, in its entirety, put on the bulk queue and processed by the threads in the bulk thread pool. The node that receives the request is referred to as the coordinating node as it manages the life of the request and assembles the response. This can be a node dedicated to just coordinating requests or one of the data nodes in the cluster.

当 bulk 请求到达集群中的某个节点时候，请求被添加到 bulk请求队列中，并且该请求会被 bulk线程池中的线程处理。接收请求的节点被称为协调节点，因为它会管理请求的生命周期并且响应客户端请求。协调节点可以是一个专有节点，也可以是集群中的一个数据节点，承担着协调功能。

A bulk request can contain documents destined for multiple indices and shards. The first processing step is therefore to split it up based on which shards the documents need to be routed to. Once this is done, each bulk sub-request is forwarded to the data node that holds the corresponding primary shard, and it is there enqueued on that node’s bulk queue. If there is no more space available on the queue, the coordinating node will be notified that the bulk sub-request has been rejected.

The bulk thread pool processes requests from the queue and documents are forwarded to replica shards as part of this processing. Once the sub-request has completed, a response is sent to the coordinating node.

Once all sub-requests have completed or been rejected, a response is created and returned to the client. It is possible, and even likely, that only a portion of the documents within a bulk request might have been rejected.


一个 bulk 请求可以包含指向多个索引多个分片的文档。bulk 请求

（1）第一个步骤会把请求拆分到不同的分片中，根据文档信息，计算路由到的分片；

（2）第二个步骤会把被拆分的子请求分发到主分片节点，子请求会被添加到对应节点的bulk请求队列；

（3）如果子请求所对应的节点 bulk请求队列已经占满，则协调节点会收到拒绝请求通知；

（4）主分片节点存在大容量的线程池，用于处理 bulk子请求，并且会将文档转发到副本节点，副本节点文档bulk也是bulk子请求的一部分，当bulk子请求处理完成，协调节点将收到响应；

（5）当所有的子请求都完成或者拒绝，客户端将接收到bulk响应。有可能的场景是，部分文档写入被拒绝，部分文档写入成功。

The reason Elasticsearch is designed with request queues of limited size is to protect the cluster from being overloaded, which increases stability and reliability. If there were no limits in place, clients could very easily bring a whole cluster down through bad or malicious behaviour. The limits that are in place have been set based on our extensive experience supporting Elasticsearch for different types of use-cases.

设计bulk请求队列，并且限制队列长度，是为了防止请求过多，导致集群过载，可以提高集群稳定性和可靠性。如果没有长度限制的请求队列机制，客户端很容易由于不合适操作，或者恶意操作，使得集群崩溃🐴。根据现有客户的实际使用场景经验，我们设置了现有的队列长度。

When using the HTTP interface, requests that results in at least a partial rejection will return with response code 429, 'Too many requests'. The principle also applies when the transport protocol is used, although the protocol and interface naturally is different. Applications and clients may report these errors back to the user in different ways, and some may even attempt to handle this automatically by retrying any rejected documents.

当通过 http协议发送bulk请求时，如果bulk请求中存在部分被拒绝的子请求，bulk请求的返回状态码是429，表示请求太多。当通过 tcp协议发送bulk请求时，这个原则也是一样，但是协议和接口会有不同表现。bulk请求的客户端（可能是某个应用）会将429报错以不同方式报告给用户。客户端程序可能会在接收到429报错后，自动重试，继续上传写入失败文档。

## How can we test this in practice?

如果实际测试bulk请求？

In order to illustrate the practical impact of this behaviour, we devised a simple test where we use our benchmarking tool Rally to run bulk indexing requests against a couple of Elastic Cloud clusters with varying number of data nodes. Configuration and instructions on how to run Rally is available in this gist.

The same indexing workload was run against three different Elastic Cloud clusters. We have been indexing with one replica shard configured wherever possible. The clusters consisted of one, two and three data nodes respectively, with each data node having 8GB RAM (4GB heap for Elasticsearch, 4GB native memory). Invoking the GET /_nodes/thread_pool API we could see that each data node by default had a fixed bulk thread pool size of two with a queue size of 200:

```json

%> curl -XGET http://<es_url>:<es_port>/_nodes/thread_pool</es_port></es_url>
"bulk": {
"type": "fixed",
"min": 2,
"max": 2,
"queue_size": 200
}

```

下面我们设计了一个简单的测试，测试bulk性能，测试工具是 [rally](https://github.com/elastic/rally)，测试多个具有不同节点的集群。

在三个不同配置的ES实例，我们会执行相同的bulk请求测试。三个ES实例，分别是1个节点、2个节点、3个节点，其中每个节点配置都是相同的（ 8GB 内存，4G设置为ES 堆内存，4G设置为节点内存）。可以通过接口`GET /_nodes/thread_pool` 接口查看

```json

GET _nodes/thread_pool

{
  "_nodes": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "cluster_name": "a2ff16d9aa2645dc87ab1714e6e16a84",
  "nodes": {
    "juVnJa6TQ7a2c3v065Gz3w": {
      "name": "instance-0000000000",
      "transport_address": "10.42.10.154:19303",
      "host": "10.42.10.154",
      "ip": "10.42.10.154",
      "version": "8.3.1",
      "build_flavor": "default",
      "build_type": "docker",
      "build_hash": "b9a6b2867996ba92ceac66cb5bafc6db25e7910e",
      "roles": [
        "data_content",
        "data_hot",
        "ingest",
        "master",
        "remote_cluster_client",
        "transform"
      ],
      "attributes": {
        "logical_availability_zone": "zone-0",
        "server_name": "instance-0000000000.a2ff16d9aa2645dc87ab1714e6e16a84",
        "availability_zone": "us-central1-a",
        "xpack.installed": "true",
        "data": "hot",
        "instance_configuration": "gcp.es.datahot.n2.68x10x45",
        "region": "unknown-region"
      },
      "thread_pool": {
        "force_merge": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "search_coordination": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "searchable_snapshots_cache_fetch_async": {
          "type": "scaling",
          "core": 0,
          "max": 6,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "ml_datafeed": {
          "type": "scaling",
          "core": 1,
          "max": 512,
          "keep_alive": "1m",
          "queue_size": -1
        },
        "snapshot_meta": {
          "type": "scaling",
          "core": 1,
          "max": 6,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "fetch_shard_started": {
          "type": "scaling",
          "core": 1,
          "max": 4,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "rollup_indexing": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "search": {
          "type": "fixed",
          "size": 4,
          "queue_size": 1000
        },
        "cluster_coordination": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "security-crypto": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "ccr": {
          "type": "fixed",
          "size": 32,
          "queue_size": 100
        },
        "flush": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "fetch_shard_store": {
          "type": "scaling",
          "core": 1,
          "max": 4,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "ml_utility": {
          "type": "scaling",
          "core": 1,
          "max": 2048,
          "keep_alive": "10m",
          "queue_size": -1
        },
        "get": {
          "type": "fixed",
          "size": 2,
          "queue_size": 1000
        },
        "system_read": {
          "type": "fixed",
          "size": 1,
          "queue_size": 2000
        },
        "system_critical_read": {
          "type": "fixed",
          "size": 1,
          "queue_size": 2000
        },
        "write": {
          "type": "fixed",
          "size": 2,
          "queue_size": 10000
        },
        "watcher": {
          "type": "fixed",
          "size": 10,
          "queue_size": 1000
        },
        "security-token-key": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "system_critical_write": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1500
        },
        "refresh": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "repository_azure": {
          "type": "scaling",
          "core": 0,
          "max": 5,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "vector_tile_generation": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "system_write": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "generic": {
          "type": "scaling",
          "core": 4,
          "max": 128,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "warmer": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "auto_complete": {
          "type": "fixed",
          "size": 1,
          "queue_size": 100
        },
        "azure_event_loop": {
          "type": "scaling",
          "core": 0,
          "max": 1,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "management": {
          "type": "scaling",
          "core": 1,
          "max": 2,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "analyze": {
          "type": "fixed",
          "size": 1,
          "queue_size": 16
        },
        "searchable_snapshots_cache_prewarming": {
          "type": "scaling",
          "core": 0,
          "max": 16,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "ml_job_comms": {
          "type": "scaling",
          "core": 4,
          "max": 2048,
          "keep_alive": "1m",
          "queue_size": -1
        },
        "snapshot": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "search_throttled": {
          "type": "fixed",
          "size": 1,
          "queue_size": 100
        }
      }
    },
    "snEBN-aVTcKiJfoT87LgKA": {
      "name": "instance-0000000001",
      "transport_address": "10.42.0.133:19375",
      "host": "10.42.0.133",
      "ip": "10.42.0.133",
      "version": "8.3.1",
      "build_flavor": "default",
      "build_type": "docker",
      "build_hash": "b9a6b2867996ba92ceac66cb5bafc6db25e7910e",
      "roles": [
        "data_content",
        "data_hot",
        "ingest",
        "master",
        "remote_cluster_client",
        "transform"
      ],
      "attributes": {
        "data": "hot",
        "server_name": "instance-0000000001.a2ff16d9aa2645dc87ab1714e6e16a84",
        "instance_configuration": "gcp.es.datahot.n2.68x10x45",
        "region": "unknown-region",
        "availability_zone": "us-central1-b",
        "logical_availability_zone": "zone-1",
        "xpack.installed": "true"
      },
      "thread_pool": {
        "force_merge": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "search_coordination": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "searchable_snapshots_cache_fetch_async": {
          "type": "scaling",
          "core": 0,
          "max": 6,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "ml_datafeed": {
          "type": "scaling",
          "core": 1,
          "max": 512,
          "keep_alive": "1m",
          "queue_size": -1
        },
        "snapshot_meta": {
          "type": "scaling",
          "core": 1,
          "max": 6,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "fetch_shard_started": {
          "type": "scaling",
          "core": 1,
          "max": 4,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "rollup_indexing": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "search": {
          "type": "fixed",
          "size": 4,
          "queue_size": 1000
        },
        "cluster_coordination": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "security-crypto": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "ccr": {
          "type": "fixed",
          "size": 32,
          "queue_size": 100
        },
        "flush": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "fetch_shard_store": {
          "type": "scaling",
          "core": 1,
          "max": 4,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "ml_utility": {
          "type": "scaling",
          "core": 1,
          "max": 2048,
          "keep_alive": "10m",
          "queue_size": -1
        },
        "get": {
          "type": "fixed",
          "size": 2,
          "queue_size": 1000
        },
        "system_read": {
          "type": "fixed",
          "size": 1,
          "queue_size": 2000
        },
        "system_critical_read": {
          "type": "fixed",
          "size": 1,
          "queue_size": 2000
        },
        "write": {
          "type": "fixed",
          "size": 2,
          "queue_size": 10000
        },
        "watcher": {
          "type": "fixed",
          "size": 10,
          "queue_size": 1000
        },
        "security-token-key": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "system_critical_write": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1500
        },
        "refresh": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "repository_azure": {
          "type": "scaling",
          "core": 0,
          "max": 5,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "vector_tile_generation": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "system_write": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "generic": {
          "type": "scaling",
          "core": 4,
          "max": 128,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "warmer": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "auto_complete": {
          "type": "fixed",
          "size": 1,
          "queue_size": 100
        },
        "azure_event_loop": {
          "type": "scaling",
          "core": 0,
          "max": 1,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "management": {
          "type": "scaling",
          "core": 1,
          "max": 2,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "analyze": {
          "type": "fixed",
          "size": 1,
          "queue_size": 16
        },
        "searchable_snapshots_cache_prewarming": {
          "type": "scaling",
          "core": 0,
          "max": 16,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "ml_job_comms": {
          "type": "scaling",
          "core": 4,
          "max": 2048,
          "keep_alive": "1m",
          "queue_size": -1
        },
        "snapshot": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "search_throttled": {
          "type": "fixed",
          "size": 1,
          "queue_size": 100
        }
      }
    },
    "4MZABTmcQSG3PvMWtK_Sjg": {
      "name": "tiebreaker-0000000002",
      "transport_address": "10.42.5.215:19463",
      "host": "10.42.5.215",
      "ip": "10.42.5.215",
      "version": "8.3.1",
      "build_flavor": "default",
      "build_type": "docker",
      "build_hash": "b9a6b2867996ba92ceac66cb5bafc6db25e7910e",
      "roles": [
        "master",
        "voting_only"
      ],
      "attributes": {
        "logical_availability_zone": "tiebreaker",
        "availability_zone": "us-central1-c",
        "server_name": "tiebreaker-0000000002.a2ff16d9aa2645dc87ab1714e6e16a84",
        "xpack.installed": "true",
        "data": "hot",
        "instance_configuration": "gcp.es.master.n2.68x32x45",
        "region": "unknown-region"
      },
      "thread_pool": {
        "force_merge": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "search_coordination": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "searchable_snapshots_cache_fetch_async": {
          "type": "scaling",
          "core": 0,
          "max": 6,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "ml_datafeed": {
          "type": "scaling",
          "core": 1,
          "max": 512,
          "keep_alive": "1m",
          "queue_size": -1
        },
        "snapshot_meta": {
          "type": "scaling",
          "core": 1,
          "max": 6,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "fetch_shard_started": {
          "type": "scaling",
          "core": 1,
          "max": 4,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "rollup_indexing": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "search": {
          "type": "fixed",
          "size": 4,
          "queue_size": 1000
        },
        "cluster_coordination": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "security-crypto": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "ccr": {
          "type": "fixed",
          "size": 32,
          "queue_size": 100
        },
        "flush": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "fetch_shard_store": {
          "type": "scaling",
          "core": 1,
          "max": 4,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "ml_utility": {
          "type": "scaling",
          "core": 1,
          "max": 2048,
          "keep_alive": "10m",
          "queue_size": -1
        },
        "get": {
          "type": "fixed",
          "size": 2,
          "queue_size": 1000
        },
        "system_read": {
          "type": "fixed",
          "size": 1,
          "queue_size": 2000
        },
        "system_critical_read": {
          "type": "fixed",
          "size": 1,
          "queue_size": 2000
        },
        "write": {
          "type": "fixed",
          "size": 2,
          "queue_size": 10000
        },
        "watcher": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "security-token-key": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "system_critical_write": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1500
        },
        "refresh": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "repository_azure": {
          "type": "scaling",
          "core": 0,
          "max": 5,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "vector_tile_generation": {
          "type": "fixed",
          "size": 1,
          "queue_size": -1
        },
        "system_write": {
          "type": "fixed",
          "size": 1,
          "queue_size": 1000
        },
        "generic": {
          "type": "scaling",
          "core": 4,
          "max": 128,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "warmer": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "auto_complete": {
          "type": "fixed",
          "size": 1,
          "queue_size": 100
        },
        "azure_event_loop": {
          "type": "scaling",
          "core": 0,
          "max": 1,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "management": {
          "type": "scaling",
          "core": 1,
          "max": 2,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "analyze": {
          "type": "fixed",
          "size": 1,
          "queue_size": 16
        },
        "searchable_snapshots_cache_prewarming": {
          "type": "scaling",
          "core": 0,
          "max": 16,
          "keep_alive": "30s",
          "queue_size": -1
        },
        "ml_job_comms": {
          "type": "scaling",
          "core": 4,
          "max": 2048,
          "keep_alive": "1m",
          "queue_size": -1
        },
        "snapshot": {
          "type": "scaling",
          "core": 1,
          "max": 1,
          "keep_alive": "5m",
          "queue_size": -1
        },
        "search_throttled": {
          "type": "fixed",
          "size": 1,
          "queue_size": 100
        }
      }
    }
  }
}

```

During the test we indexed into a varying number of shards (2, 4, 8, 16, and 32) using a varying number of concurrent clients (8, 16, 24, 32, 48, and 64) for each cluster. For every combination of shard and client count we indexed 6.4 million documents with a batch size of 100 documents and another 6.4 million documents with a batch size of 200 documents. This means that in total we attempted to index 384 million documents per cluster.

For this test we treat the clusters as a black box, and perform the analysis from the client’s perspective. To limit the scope we will also not look at the impact of various configurations on performance as that is a quite large topic on its own.

在测试中，

（1）我们并发写入设置不同的并发客户端，分别为 8/16/24/32/48/64 个客户端，

（2）索引分别设置不同的分片数，分别为 2/4/8/16/32 个分片数。

（3）批量写入设置不同的文档数，分别为 100/200个文档，

对于不同的设置，写入文档总数量都是相同的，每个集群最终将写入接近4亿文档。

在此次测试中，我们将集群视为一个黑盒子，并从客户的角度进行分析。为了限制范围，我们也不会考虑各种配置对于性能的影响，因为这是一个非常大的主题。

All the generated, detailed metrics were sent to a separate Elastic Cloud instance for analysis using Kibana. For each request Rally measures how many the documents in the bulk request were rejected and successful. Based on this data we can classify each request as successful, partially rejected, and fully rejected. A few requests also timed out, and these have also been included for completeness.

Unlike Beats and Logstash, Rally does not retry failed indexing requests, so each has the same number of requests executed but the final number of documents indexed varied from run to run depending on the volume of rejections.

所有测试指标，都会发送到一个独立的集群，并且使用kibana可视化。对于每个客户端 bulk请求，`rally`可以记录请求是否成功，如果失败，拒绝写入的文档数。根据`rally`记录数据，可以区分bulk请求成功/完全拒绝/部分拒绝。另外，还有一些请求超时，为了完整性起见，统计数据也会包含超时的请求。

与 beats 和 logstash不同，`rally`写入失败不会重试。因此，最终落到ES中文档数也会不同，因为有部分请求被拒绝了。

## How bulk rejection frequency depend on shard count, clients count, and data node count?

bulk写入拒绝频率和分片数量、客户端并发量、数据节点数之间有什么关联？

Bulk rejections occur when the bulk queues fill up. The number of queue slots that get used depends both on the number of concurrent requests, and the number of shards being indexed into. To measure this correlation we have added a calculated metric, client shard concurrency, to each run. This is defined as the number of shards being indexed into, multiplied by the number of concurrent indexing threads, and indicates how many queue slots would be needed to hold all bulk sub-requests.

当bulk队列占满后，后入的bulk请求将被拒绝。写入请求会使用的请求队列大小，取决于客户端的并发量和索引的分片数设置。为了准确描述相关性，引入一个新的指标，称为客户端分片并发量 = 客户端并发量 * 索引分片数。


![客户端分片并发量](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/101/bulk-rejection-blog-1.png?raw=true)

In the graph below, we show how the percentage of requests that result in partial or full rejections, depends on the client shard concurrency for the three different clusters.

For clusters with one or two nodes we can see that appearance of bulk rejections start when the client shard concurrency level is somewhere between 192 and 256. This makes sense as each node has a bulk queue size of 200. For the cluster with 3 nodes we can see that it is able to handle even higher level of client shard concurrency without any bulk rejections appearing. 

Once we get over this limit, we start seeing partial bulk rejections, where at least one sub-request has managed to get queued and processed. A relatively small portion of requests also result on full rejections as the concurrency level increases, especially for the single node cluster. 

When we compare the single and two node clusters, we can see that the percentage of fully successful requests increases slightly and that there are fewer full rejections. This is expected, as the total bulk queue across the cluster is twice as large and requests are sent to all data nodes. Even though the total bulk queue size is twice as large across the cluster, the 2 node cluster does not appear able to handle twice the client shard concurrency of the single node cluster. This is likely due to the fact that distribution is not perfect and that the introduction of replica shards have resulted in each indexing operation requiring more work and being slower as a result. An important thing to note is also that all partial rejections are treated as equals in this graph. The number of rejected documents is not shown and does indeed vary depending on the cluster size, but we will shortly look at that in greater detail. 


由上图可以看到，我们展示了三个集群里面，不同类型响应占比（写入成功/部分写入成功/写入全部失败/写入超时）。对于具有一个节点和两个节点的集群，可以看到，当客户端分片并发量达到 192～256，开始出现拒绝请求。这个是有意义的，因为根据`_nodes/thread_pool/`，每个节点的bulk队列大小为200。对于3个节点的集群，因为客户端分片并发量更大，可以看到 192~256 范围内不会出现拒绝请求。

还可以看到，对于3节点集群，当达到256限制以上，开始出现部分拒绝。当客户端分片并发量继续增加后，完全拒绝类型请求开始变多。

如果我们只是比较单节点和双节点的集群，可以看到双节点集群，完成成功的百分比更高，完全失败的百分比更低。这个结果在意料之中，因为双节点集群，可以接受bulk请求的bulk队列是单节点集群的两倍，并且双节点集群，有两个数据节点，都会接收bulk请求。但是，从图中，可以看到，双节点集群的处理能力并不是单节点集群的两倍。可能的原因如下：

（1）分片分发不一定是完美的，即不一定，每个节点都接收正好一半的bulk请求；

（2）副本分片的设置，导致双节点集群，bulk请求处理更耗时；

（3）需要注意到，上图中，部分被拒绝的请求，并没有详细的被拒绝文档数量，我们把所有部分被拒绝请求当作相同的类型，实际上部分被拒绝请求，可能包含非常不一样的拒绝文档数量。

当继续查看3节点集群，可以看到，只有当客户端分片并发量到达非常高时候，才会出现完全拒绝请求。

![详细bulk信息](https://github.com/Kua-Fu/blog-book-images/blob/main/elastic/101/bulk-rejection-blog-2.png?raw=true)

上图展示了部分bulk失败的详细信息，主要包含了bulk失败的文档数量。

可以看到，对于3种类型集群，随着客户端分片并发量的增加，被拒绝的比例变大；但是随着节点的变多，拒绝的级别（可以任务是拒绝的文档数量）变小。

对于单节点集群、双节点集群，部分拒绝几乎在相同的客户端分片并发量产生，但不相同的是，单节点集群被拒绝文档数量增加的更快。这意味着，即使不同类型集群都发生了部分拒绝请求，更多节点集群，实际上会写入更多的文档。

## Can’t I just get around this by increasing the bulk queue size?

可以通过增加bulk请求队列长度，解决bulk请求429问题吗？

One of the most common reactions when faced with bulk rejections is to increase the size of the bulk queue. Why not set it to a really large value so you do not have to worry about this again?

Increasing the size of the queue is not likely to improve the indexing performance or throughput of your cluster. Instead it would just make the cluster queue up more data in memory, which is likely to result in bulk requests taking longer to complete. The more bulk requests there are in the queue, the more precious heap space will be consumed. If the pressure on the heap gets too large, it can cause a lot of other performance problems and even cluster instability.

Adjusting the queue sizes is therefore strongly discouraged, as it is like putting a temporary band-aid on the problem rather than actually fixing the underlying issue. So what else can we do improve the situation?

当面临bulk请求被拒绝场景，最常见的反应是为什么不增加请求队列长度，设置一个非常大的值，这样就不用担心bulk请求被拒绝了？

需要明确的是，增加请求队列长度不等于增加索引的性能和吞吐量。相反，增加队列长度会使得集群内存中存在更多排队中的请求，这样可能导致bulk请求会经过更长时间才能完成。越多的bulk请求，越多宝贵的集群堆内存就会被消耗。如果堆内存占用过大，可能会导致其他的性能问题，甚至让集群变得不稳定。

因此，我们强烈建议不要调整队列大小，因为通常改变队列长度只会临时改变被拒绝请求数量，但是不会根本上改变bulk请求被拒绝问题。那么，我们应该如何做，根本上解决问题呢？

## Can coordinating only nodes help?

专有协调节点会有帮助吗？

By introducing coordinating only nodes, the data nodes will be able to focus on processing sub-requests, as the request itself will not take up a slot on their bulk queue. This is generally good, but the actual benefit of this arrangement is likely to vary from use-case to use-case. In many use cases it does relatively little difference, and we see lots of successful indexing heavy use cases that do not use dedicated coordinating nodes.

通过引入专有协调节点，数据节点可以更加专注于处理bulk子请求，bulk请求不会占用请求队列。通常情况下，引入专有协调节点可以改善bulk请求被拒绝问题，但实际上是否改善，还是因为不同实际使用场景会有不同的答案。在很多的案例中，引入专有协调节点，改善几乎没有发生，我们遇到过很多的索引数据量很大的集群，根本没有使用专有协调节点。

## What conclusions can we draw?

我们可以得出什么结论？

As always, there is not necessarily any good one-size-fits-all solution, and the way to address bulk rejections will vary from use-case to use-case. If you see bulk rejections, try to understand why they are taking place and whether it is a single node or the whole cluster that is affected.

一如既往的，我们无法得到一个万能方法，可以解决所有不同场景的bulk请求被拒绝问题。不同的用户使用场景，往往最终使用不同的解决方案。如果你发现了bulk请求失败，请（1）尝试排查发生拒绝的原因；（2）是单个节点、还是整个集群都发生了bulk请求被拒绝。

如果是整个集群处于过载状态，而不是单个节点承担了过多的写入压力，可能要考虑集群扩容（横向添加节点或者纵向升级配置）。扩容后，整个集群bulk请求队列的总长度将变大，可以承担更多的写入压力。如果只是简单添加现有集群节点bulk请求队列的长度，即使临时解决了问题，实际上后面可能会面临更多的其他性能问题。

最后，还需要记住的是，bulk被拒绝并不代表所有的文档都写入失败，程序代码中需要考虑到只有一部分文档写入失败，要考虑这一部分文档的重传机制。可以参考logstash,beats 中的重传逻辑。





