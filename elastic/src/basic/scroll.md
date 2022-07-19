# 查询游标

scroll 查询 可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价

游标查询允许我们 先做查询初始化，然后再批量地拉取结果。 这有点儿像传统数据库中的 cursor 。

游标查询会取某个时间点的快照数据。 查询初始化之后索引上的任何变化会被它忽略。 它通过保存旧的数据文件来实现这个特性，结果就像保留初始化时的索引 视图 一样。

深度分页的代价根源是结果集全局排序，如果去掉全局排序的特性的话查询结果的成本就会很低。 游标查询用字段 _doc 来排序。 这个指令让 Elasticsearch 仅仅从还有结果的分片返回下一批结果。

启用游标查询可以通过在查询的时候设置参数 scroll 的值为我们期望的游标查询的过期时间。 游标查询的过期时间会在每次做查询的时候刷新，所以这个时间只需要足够处理当前批的结果就可以了，而不是处理查询结果的所有文档的所需时间。 

这个过期时间的参数很重要，因为保持这个游标查询窗口需要消耗资源，所以我们期望如果不再需要维护这种资源就该早点儿释放掉。 设置这个超时能够让 Elasticsearch 在稍后空闲的时候自动释放这部分资源。

```json

GET /kibana_sample_data_logs/_search?scroll=1m 0️⃣
{
  "query": {
    "match_all": {}
  },
  "sort": [ 1️⃣
    "_doc"
  ],
  "size": 1000
}

{
  "_scroll_id": "FGluY2x1ZGVfY29udGV4dF91dWlkDXF1ZXJ5QW5kRmV0Y2gBFmxnRlZGdy14VDVDVWlsTENZLUEwUncAAAAAAEJDTBZzbkVCTi1hVlRjS2lKZm9UODdMZ0tB",
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 14074,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}

```

0️⃣ 保持游标查询窗口一分钟。

1️⃣ 关键字 _doc 是最有效的排序顺序。

这个查询的返回结果包括一个字段 _scroll_id， 它是一个base64编码的长字符串 。 现在我们能传递字段 _scroll_id 到 _search/scroll 查询接口获取下一批结果：

```json

GET /_search/scroll
{
    "scroll": "1m", 0️⃣
    "scroll_id" : "FGluY2x1ZGVfY29udGV4dF91dWlkDXF1ZXJ5QW5kRmV0Y2gBFmxnRlZGdy14VDVDVWlsTENZLUEwUncAAAAAAEJDTBZzbkVCTi1hVlRjS2lKZm9UODdMZ0tB"
}

{
  "_scroll_id": "FGluY2x1ZGVfY29udGV4dF91dWlkDXF1ZXJ5QW5kRmV0Y2gBFmxnRlZGdy14VDVDVWlsTENZLUEwUncAAAAAAEJDTBZzbkVCTi1hVlRjS2lKZm9UODdMZ0tB",
  "took": 8,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 14074,
      "relation": "eq"
    },
    "max_score": null,
    "hits": []
  }
}

```

0️⃣ 注意再次设置游标查询过期时间为一分钟

这个游标查询返回的下一批结果。 尽管我们指定字段 size 的值为1000，我们有可能取到超过这个值数量的文档。 当查询的时候， 字段 size 作用于单个分片，所以每个批次实际返回的文档数量最大为 size * number_of_primary_shards 。

> 🦉**Note**
>
>注意游标查询每次返回一个新字段 _scroll_id。
>
>每次我们做下一次游标查询， 我们必须把前一次查询返回的字段 _scroll_id 传递进去。 当没有更多的结果返回的时候，我们就处理完所有匹配的文档了。
>
>提示：某些官方的 Elasticsearch 客户端比如 Python 客户端 和 Perl 客户端 提供了这个功能易用的封装。
