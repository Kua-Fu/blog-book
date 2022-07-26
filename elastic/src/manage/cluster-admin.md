# 监控

Elasticsearch 经常以多节点集群的方式部署。有多种 API 让你可以管理和监控集群本身，而不用和集群里存储的数据打交道。

和 Elasticsearch 里绝大多数功能一样，我们有一个总体的设计目标，即任务应该通过 API 执行，而不是通过修改静态的配置文件。这一点在你的集群扩容时尤为重要。即便通过配置管理系统（比如 Puppet，Chef 或者 Ansible），一个简单的 HTTP API 调用，也比往上百台物理设备上推送新配置文件简单多了。

因此，本章将介绍各种可以让你动态调整、调优和调配集群的 API。同时，还会介绍一系列提供集群自身统计数据的 API，你可以用这些接口来监控集群健康状态和性能。