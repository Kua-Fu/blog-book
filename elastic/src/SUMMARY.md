
[简介](./index.md)

# 概览

- [前言](./perface.md)

  - [谁应该读这本书](./perface/who-should-read.md)
  - [为什么我们要写这本书](./perface/why-we-write-this-book.md)
  - [如何读这本书](./perface/how-to-read-this-book.md)
  - [鸣谢](./perface/acknowledgment.md)

# 基础

- [基础入门](./basic/basic.md)
  - [你知道的, 为了搜索…​](./basic/intro.md)
	  - [安装使用](./basic/running-elastic.md)
	  - [面向文档](./basic/document-oriented.md)
	  - [适应新环境](./basic/find-your-feet.md)
	  - [索引员工文档](./basic/index-employee-document.md)
	  - [检索文档](./basic/retrieving-a-document.md)
	  - [轻量搜索](./basic/search-lite.md)
	  - [使用查询表达式搜索](./basic/search-with-query-dsl.md)
	  - [更复杂的搜索](./basic/more-complicated-search.md)
	  - [全文搜索](./basic/full-text-search.md)
	  - [短语搜索](./basic/phrase-search.md)
	  - [高亮搜索](./basic/highlighting-intro.md)
	  - [分析](./basic/analytics.md)
	  - [小结](./basic/conclusion.md)
	  - [分布式特性](./basic/distributed_nature.md)
	  - [后续步骤](./basic/next-step.md)
	  
  - [集群内的原理](./basic/distributed-cluster.md)
	  - [空集群](./basic/empty-cluster.md)
	  - [集群健康](./basic/cluster-health.md)
	  - [添加索引](./basic/add-an-index.md)
	  - [添加故障转移](./basic/add-failover.md)
	  - [水平扩容](./basic/scale-horizontally.md)
	  - [应对故障](./basic/coping-with-failure.md)
	  
  - [数据输入和输出](./basic/data-in-data-out.md)
	  - [什么是文档](./basic/document.md)
	  - [文档元数据](./basic/document-metadata.md)
	  - [索引文档](./basic/index-doc.md)
	  - [取回一个文档](./basic/get-doc.md)
	  - [检查文档是否存在](./basic/doc-exists.md)
	  - [更新整个文档](./basic/update-doc.md)
	  - [创建新文档](./basic/create-doc.md)
	  - [删除文档](./basic/delete-doc.md)
	  - [处理冲突](./basic/version-control.md)
	  - [乐观并发控制](./basic/optimistic-concurrency-control.md)
	  - [文档的部分更新](./basic/partial-update.md)
	  - [取回多个文档](./basic/retrieving-multiple-docs.md)
	  - [代价较小的批量操作](./basic/bulk.md)
	  
  - [分布式文档存储](./basic/distributed-docs.md)
	  - [路由一个文档到一个分片中](./basic/routing-value.md)
	  - [主分片和副本分片如何交互](./basic/how-primary-and-replica-shard-interact.md)
	  - [新建、索引和删除文档](./basic/distrib-write.md)
	  - [取回一个文档](./basic/distrib-read.md)
	  - [局部更新文档](./basic/partial-update-to-a-doc.md)
	  - [多文档模式](./basic/distrib-multi-doc.md)
	  
  - [搜索-最基本的工具](./basic/search.md)
  
  - [映射和分析](./basic/mapping-analysis.md)
  
  - [请求体查询](./basic/full-search-body.md)
  
  - [排序与相关性](./basic/sorting.md)
  
  - [执行分布式检索](./basic/distributed-search.md)
	  - [查询阶段](./basic/query-phase.md)
	  - [取回阶段](./basic/fetch-phase.md)
	  - [搜索选项](./basic/search-options.md)
	  - [查询游标](./basic/scroll.md)
  
  - [索引管理](./basic/index-management.md)
  
  - [分片内部原理](./basic/inside-a-shard.md)
	  - [使文本可被搜索](./basic/making-text-searchable.md)
	  - [动态更新索引](./basic/dynamic-indices.md)
	  - [近实时搜索](./basic/near-real-time.md)
	  - [持久化变更](./basic/translog.md)
	  - [段合并](./basic/merge-process.md)

# 深入搜索

# 处理人类语言

- [概览](./languages/readme.md)
- [开始处理各种语言](./languages/language-intro.md)
   - [使用语言分析器](./languages/using-language-analyzers.md)
   - [配置语言分析器](./languages/config-language-analyzers.md)
   - [混合语言的陷阱](./languages/language-pitfalls.md)
   - [每份文档一种语言](./languages/one-language-docs.md)
   - [每个域一种语言](./lanaugages/one-language-field.md)
   - [混合语言域](./languages/mixed-language-fields.md)

# 聚合

# 地理位置

# 数据建模

# 管理、监控和部署

- [管理、监控入门](./manage/administration.md)
  - [监控](./manage/cluster-admin.md)
	- [Marval监控](./manage/marval.md)
	- [集群健康](./manage/cluster-health.md)
	- [监控单个节点](./manage/monitor-individual-nodes.md)
		- [索引部分](./manage/monitor-individual-nodes-index.md)
		- [操作系统和进程](./manage/monitor-individual-nodes-os.md)
		- [jvm部分](./manage/monitor-individual-nodes-jvm.md)
		- [线程池部分](./manage/monitor-individual-nodes-thread-pool.md)
		- [文件系统和网络部分](./manage/monitor-individual-nodes-fs.md)
		- [断路器部分](./manage/monitor-individual-nodes-breaker.md)
	- [集群统计](./manage/cluster-stats.md)
	- [索引统计](./manage/index-stats.md)
	- [等待中的任务](./manage/pending-task.md)




