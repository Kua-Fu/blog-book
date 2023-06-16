# ali openstore

当前dataflux产品中，日志类数据存储依赖于阿里云ES日志增强版产品，在我们一年多的使用过程中，总体服务是比较稳定的，但也有一些问题，下面是我们团队的一些使用总结。

## 一、数据写入

## 1.1 openStore 功能

[openStore功能](https://help.aliyun.com/document_detail/311665.html)基于 ES的多级存储架构，可以将冷数据保存到对象存储，降低存储成本。当前，我们的使用场景是把 滚动出的索引都保存到openstore，最大化减少存储成本。

## 1.2 Indexing Service功能

由于openstore 提供了[Indexing Service功能(写分离、云托管)](https://help.aliyun.com/document_detail/212971.html?spm=a2c4g.11186623.0.0.20e1134210kuCx)，当前我们付费客户都默认开启该功能。

在客户海量日志或是有峰值抖动场景，都可以比较好的实现写入低延迟。

当前杭州节点，只有偶发的日志量特别大时刻，可能会达到 600M 写入限流阈值，导致写入失败。

## 1.3 aliyun-codec插件

[codec插件](https://help.aliyun.com/document_detail/363036.html)基于 zstd压缩算法（兼顾LZ4 和 DEFLATE 算法优点），可以对ES 索引的行存、列存、倒排3个部分进行压缩。在我们的实际使用中，大概可以在原有基础上再减少 1/3 ～ 2/3 的存储用量。

## 1.4 analytic-search插件

[analytic-search插件](https://help.aliyun.com/document_detail/460080.html) 插件主要作用是，在写入时候，指定 sort fields，通过增加一定的写入消耗，增强某些查询性能。

目前，我们是把 date 字段启用为 sort fields, 但是查询性能提升不是很明显。

## 1.5 小结

| 序号 | 优点 |
| --- | --- | 
| 1 | openstore的多级存储，保证了滚动出索引存储在对象存储，最大化降低存储成本| 
| 2 | IndexService 的写分离架构，保证了写入的低延迟和对于查询的更小影响 |
| 3 | aliyun-codec插件，更好的压缩性能，降低了存储成本，但是 lucene实际上也支持 zstd压缩算法，ES 目前也可以通过实现接口使用 zstd压缩算法|



| 序号| 缺点|
|---|---|
| 1 | 当前我们多租户场景，集群索引会非常多，保存策略、索引模版、索引mapping管理不是很稳定，杭州节点经常出现模版创建超时 |
| 2 | IndexService的写入限流（集群维度 600M，分片维度 30M），会限制集群的写入性能, 或者限制不合理|
| 3 | IndexService的写入分离，有时候会出现数据实际写入，但是查询不到问题|
 

## 二、数据查询

### 2.1 概述

阿里云ES日志增强版本，查询主要还是基于原生ES的。使用过程中，当前我们遇到的问题主要还是，openstore下的冷节点，搜索性能不高（相比较热节点），当多个查询涉及同一个冷节点，经常会导致节点CPU 飙升，查询超时甚至节点崩溃重启。

### 2.2 小结

|序号| 优点|
|---|---|
| 1 | IndexService的读写分离架构，保证了大查询、慢查询对于写入性能的低影响|
| 2 | 阿里云ES内核中，存在一些查询优化，但是这一部分对于客户透明|

|序号| 缺点|
|---|---|
| 1 | 查询冷节点数据，速度慢|
| 2 | 冷数据节点可能由于多个慢查询，导致节点崩溃重启|



## 三、运维

### 3.1 资源管理

当前使用阿里云ES，版本升级基本可以做到平滑升级，集群添加新节点、升级原有配置等都比较方便，但是，升配后无法降配

### 3.2 集群监控

当前，阿里云ES自带的监控日志和监控指标，可以较好的弥补kibana monitor（在集群有问题、多索引场景下打不开的问题），但是当前的监控日志保存天数不可选（默认7天），遇到问题也无法查看服务器日志

### 3.3 技术支持

使用阿里云ES，技术支持有保证。

## 四、ES原生问题

针对我们的多租户场景，ES原生的架构有下面问题:

| 序号 | 问题| 描述| 解决思路|
| --- | --- | --- | --- |
| 1 | 不支持超多分片| 目前官方建议是单个节点1000个分片，我们的使用场景（单个工作空间，10多种数据类型，占用10多个分片），即使数据量不大，也可能导致集群性能瓶颈;| 多集群|
|2 | 单索引不支持超多字段| 当前ES 单个索引建议1000个字段以内，当采集数据出现非常多字段，会触发严重的写入延迟||
|3 | 单索引不支持字段类型变更 | 由于ES 每个索引都有固定的mapping(Scheme), 当前采集数据字段类型变更，会导致无法写入，只能滚动出新索引 | | 






 
