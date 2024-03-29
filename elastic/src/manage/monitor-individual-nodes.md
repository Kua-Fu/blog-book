# 监控单个节点

集群健康 就像是光谱的一端——对集群的所有信息进行高度概述。而 节点统计值 API 则是在另一端。它提供一个让人眼花缭乱的统计数据的数组，包含集群的每一个节点统计值。

节点统计值 提供的统计值如此之多，在完全熟悉它之前，你可能都搞不清楚哪些指标是最值得关注的。我们将会高亮那些最重要的监控指标（但是我们鼓励你记录接口提供的所有指标——或者用 Marvel ——因为你永远不知道何时需要某个或者另一个值）。

节点统计值 API 可以通过如下命令执行：

```json

GET _nodes/stats

{
  "_nodes": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "cluster_name": "a2ff16d9aa2645dc87ab1714e6e16a84",
  "nodes": {
    "snEBN-aVTcKiJfoT87LgKA": {
      "timestamp": 1657777592809,
      "name": "instance-0000000001",
      "transport_address": "10.42.0.133:19375",
      "host": "10.42.0.133",
      "ip": "10.42.0.133:19375",
      "roles": [
        "data_content",
        "data_hot",
        "ingest",
        "master",
        "remote_cluster_client",
        "transform"
      ],
	  ...

```

在输出内容的开头，我们可以看到集群名称和我们的第一个节点。

节点是排列在一个哈希里，以节点的 UUID 作为键名。还显示了节点网络属性的一些信息（比如传输层地址和主机名）。这些值对调试诸如节点未加入集群这类自动发现问题很有用。通常你会发现是端口用错了，或者节点绑定在错误的 IP 地址/网络接口上了。


