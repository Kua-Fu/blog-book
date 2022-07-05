# 轻量搜索

一个 GET 是相当简单的，可以直接得到指定的文档。 现在尝试点儿稍微高级的功能，比如一个简单的搜索！

第一个尝试的几乎是最简单的搜索了。我们使用下列请求来搜索所有雇员：

```json

GET /megacorp/_search

// 返回列表
{
  "took": 0,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 3,
      "relation": "eq"
    },
    "max_score": 1,
    "hits": [
      {
        "_index": "megacorp",
        "_id": "1",
        "_score": 1,
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
      },
      {
        "_index": "megacorp",
        "_id": "2",
        "_score": 1,
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
      },
      {
        "_index": "megacorp",
        "_id": "3",
        "_score": 1,
        "_source": {
          "type": "employee",
          "first_name": "Douglas",
          "last_name": "Fir",
          "age": 35,
          "about": "I like to build cabinets",
          "interests": [
            "forestry"
          ]
        }
      }
    ]
  }
}

```

可以看到，我们仍然使用索引库 megacorp ，但与指定一个文档 ID 不同，这次使用 _search 。返回结果包括了所有三个文档，放在数组 hits 中。一个搜索默认返回十条结果。


⚠️ 返回结果不仅告知匹配了哪些文档，还包含了整个文档本身：显示搜索结果给最终用户所需的全部信息。


接下来，尝试下搜索姓氏为 ``Smith`` 的雇员。为此，我们将使用一个 高亮 搜索，很容易通过命令行完成。这个方法一般涉及到一个 查询字符串 （query-string） 搜索，因为我们通过一个URL参数来传递查询信息给搜索接口：

```json

GET /megacorp/_search?q=last_name:Smith&pretty

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
      "value": 2,
      "relation": "eq"
    },
    "max_score": 0.4700036,
    "hits": [
      {
        "_index": "megacorp",
        "_id": "1",
        "_score": 0.4700036,
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
      },
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
