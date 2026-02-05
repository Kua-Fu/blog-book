# 索引部分

索引(indices) 部分列出了这个节点上所有索引的聚合过的统计值：

```json

   "indices": {
        "docs": { 0️⃣
          "count": 6539564,
          "deleted": 2082
        },
        "shard_stats": {
          "total_count": 97
        },
        "store": {
          "size_in_bytes": 2646907845,
          "total_data_set_size_in_bytes": 2646907845,
          "reserved_in_bytes": 0
        },
        "indexing": {
          "index_total": 15286202,
          "index_time_in_millis": 2403080,
          "index_current": 0,
          "index_failed": 14118852,
          "delete_total": 20512,
          "delete_time_in_millis": 3402,
          "delete_current": 0,
          "noop_update_total": 1,
          "is_throttled": false,
          "throttle_time_in_millis": 0 1️⃣
        },
		...
```

返回的统计值被归入以下部分：

* 0️⃣ docs 展示节点内存有多少文档，包括还没有从段里清除的已删除文档数量。

* 1️⃣ store 部分显示节点耗用了多少物理存储。这个指标包括主分片和副本分片在内。如果限流时间很大，那可能表明你的磁盘限流设置得过低（在段和合并里讨论过）。

```json

 "indexing": {
   "index_total": 15286202,
   "index_time_in_millis": 2403080,
   "index_current": 0,
   "index_failed": 14118852,
   "delete_total": 20512,
   "delete_time_in_millis": 3402,
   "delete_current": 0,
   "noop_update_total": 1,
   "is_throttled": false,
   "throttle_time_in_millis": 0
 },
 "get": {
   "total": 1980564,
   "time_in_millis": 202478,
   "exists_total": 1973030,
   "exists_time_in_millis": 201941,
   "missing_total": 7534,
   "missing_time_in_millis": 537,
   "current": 0
 },
 "search": {
   "open_contexts": 0,
   "query_total": 3272088,
   "query_time_in_millis": 1806935,
   "query_current": 0,
   "fetch_total": 3129569,
   "fetch_time_in_millis": 97430,
   "fetch_current": 0,
   "scroll_total": 866806,
   "scroll_time_in_millis": 7417778,
   "scroll_current": 0,
   "suggest_total": 0,
   "suggest_time_in_millis": 0,
   "suggest_current": 0
 },
 "merges": {
   "current": 0,
   "current_docs": 0,
   "current_size_in_bytes": 0,
   "total": 97527,
   "total_time_in_millis": 5367815,
   "total_docs": 364477569,
   "total_size_in_bytes": 136348510561,
   "total_stopped_time_in_millis": 0,
   "total_throttled_time_in_millis": 762543,
   "total_auto_throttle_in_bytes": 2482171511
 }
		
```

1. indexing 显示已经索引了多少文档。

   * 这个值是一个累加计数器。在文档被删除的时候，数值不会下降。
   
   * 还要注意的是，在发生内部 索引 操作的时候，这个值也会增加，比如说文档更新。
   
   * 还列出了索引操作耗费的时间，正在索引的文档数量，以及删除操作的类似统计值。

1. get 显示通过 ID 获取文档的接口相关的统计值。

	* 包括对单个文档的 GET 和 HEAD 请求。

1. search 描述在活跃中的搜索（ open_contexts ）数量、查询的总数量、以及自节点启动以来在查询上消耗的总时间。
   
   * 用 query_time_in_millis / query_total 计算的比值，可以用来粗略的评价你的查询有多高效。比值越大，每个查询花费的时间越多，你应该要考虑调优了。
   
   * fetch 统计值展示了查询处理的后一半流程（query-then-fetch 里的 fetch ）。如果 fetch 耗时比 query 还多，说明磁盘较慢，或者获取了太多文档，或者可能搜索请求设置了太大的分页（比如， size: 10000 ）。
   
1. merges 包括了 Lucene 段合并相关的信息。

	* 它会告诉你目前在运行几个合并，合并涉及的文档数量，正在合并的段的总大小，以及在合并操作上消耗的总时间。
	* 在你的集群写入压力很大时，合并统计值非常重要。合并要消耗大量的磁盘 I/O 和 CPU 资源。如果你的索引有大量的写入，同时又发现大量的合并数，一定要去阅读索引性能技巧。
	
	* 注意：文档更新和删除也会导致大量的合并数，因为它们会产生最终需要被合并的段 碎片 。

```json

"query_cache": {
  "memory_size_in_bytes": 997384,
  "total_count": 1037643,
  "hit_count": 19450,
  "miss_count": 1018193,
  "cache_size": 188,
  "cache_count": 670,
  "evictions": 482
},
"fielddata": {
  "memory_size_in_bytes": 736,
  "evictions": 0
},
"completion": {
  "size_in_bytes": 0
},
"segments": {
  "count": 192,
  "memory_in_bytes": 0,
  "terms_memory_in_bytes": 0,
  "stored_fields_memory_in_bytes": 0,
  "term_vectors_memory_in_bytes": 0,
  "norms_memory_in_bytes": 0,
  "points_memory_in_bytes": 0,
  "doc_values_memory_in_bytes": 0,
  "index_writer_memory_in_bytes": 4173916,
  "version_map_memory_in_bytes": 51318,
  "fixed_bit_set_memory_in_bytes": 593296,
  "max_unsafe_auto_id_timestamp": 1656995860384,
  "file_sizes": {}
},

```

1. filter_cache 展示了已缓存的过滤器位集合所用的内存数量，以及过滤器被驱逐出内存的次数。

	* 过多的驱逐数 可能 说明你需要加大过滤器缓存的大小，或者你的过滤器不太适合缓存（比如它们因为高基数而在大量产生，就像是缓存一个 now 时间表达式）
	
	* 不过，驱逐数是一个很难评定的指标。过滤器是在每个段的基础上缓存的，而从一个小的段里驱逐过滤器，代价比从一个大的段里要廉价的多。有可能你有很大的驱逐数，但是它们都发生在小段上，也就意味着这些对查询性能只有很小的影响。
	
	* 把驱逐数指标作为一个粗略的参考。如果你看到数字很大，检查一下你的过滤器，确保他们都是正常缓存的。不断驱逐着的过滤器，哪怕都发生在很小的段上，效果也比正确缓存住了的过滤器差很多。
	
1. field_data 显示 fielddata 使用的内存，用以聚合、排序等等。

	* 这里也有一个驱逐计数。和 filter_cache 不同的是，这里的驱逐计数是很有用的：这个数应该或者至少是接近于 0。因为 fielddata 不是缓存，任何驱逐都消耗巨大，应该避免掉。
	
	* 如果你在这里看到驱逐数，你需要重新评估你的内存情况，fielddata 限制，请求语句，或者这三者。
	
1. segments 会展示这个节点目前正在服务中的 Lucene 段的数量。

	* 这是一个重要的数字。大多数索引会有大概 50–150 个段，哪怕它们存有 TB 级别的数十亿条文档。
	
	* 段数量过大表明合并出现了问题（比如，合并速度跟不上段的创建）。
	
	* 注意这个统计值是节点上所有索引的汇聚总数。记住这点。
	
	* memory 统计值展示了 Lucene 段自己用掉的内存大小。这里包括底层数据结构，比如倒排表，字典，和布隆过滤器等。太大的段数量会增加这些数据结构带来的开销，这个内存使用量就是一个方便用来衡量开销的度量值。
	
