# 短语搜索

找出一个属性中的独立单词是没有问题的，但有时候想要精确匹配一系列单词或者_短语_ 。 比如， 我们想执行这样一个查询，仅匹配同时包含 “rock” 和 “climbing” ，并且 二者以短语 “rock climbing” 的形式紧挨着的雇员记录。

为此对 match 查询稍作调整，使用一个叫做 match_phrase 的查询：

```json

GET /megacorp/_search?pretty
{
  "query": {
    "match_phrase": {
      "about": "rock climbing"
    }
  }
}

// 返回结果
{
  "took": 14,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 1.4167401,
    "hits": [
      {
        "_index": "megacorp",
        "_id": "1",
        "_score": 1.4167401,
        "_source": {
          "type": "employee",
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}
```

毫无悬念，返回结果仅有 John Smith 的文档。
