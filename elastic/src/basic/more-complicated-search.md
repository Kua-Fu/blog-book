# 更复杂的搜索

现在尝试下更复杂的搜索。 同样搜索姓氏为 Smith 的员工，但这次我们只需要年龄大于 30 的。查询需要稍作调整，使用过滤器 filter ，它支持高效地执行一个结构化查询。


```json

GET /megacorp/_search?pretty
{
  "query": {
    "bool": {
      "must": {
        "match": {
          "last_name": "smith" 0️⃣
        } 
      },
      "filter": {
        "range": {
          "age": {
            "gt": 30 1️⃣
          }
        }
      }
    }
  }
}

// 返回结果
{
  "took": 2,
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
    "max_score": 0.4700036,
    "hits": [
      {
        "_index": "megacorp",
        "_id": "2",
        "_score": 0.4700036,
        "_source": {
          "type": "employee",
          "first_name": "Jane",
          "last_name": "Smith",
          "age": 32,
          "about": "I like to collect rock albums",
          "interests": [
            "music"
          ]
        }
      }
    ]
  }
}

```

* 0️⃣ 这部分与我们之前使用的 match 查询 一样

* 1️⃣ 这部分是一个 range 过滤器 ， 它能找到年龄大于 30 的文档，其中 gt 表示_大于_(great than)。


目前无需太多担心语法问题，后续会更详细地介绍。只需明确我们添加了一个 过滤器 用于执行一个范围查询，并复用之前的 match 查询。现在结果只返回了一名员工，叫 Jane Smith，32 岁。
