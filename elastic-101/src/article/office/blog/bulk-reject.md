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




