# 分析

终于到了最后一个业务需求：支持管理者对员工目录做分析。 Elasticsearch 有一个功能叫聚合（aggregations），允许我们基于数据生成一些精细的分析结果。聚合与 SQL 中的 GROUP BY 类似但更强大。

举个例子，挖掘出员工中最受欢迎的兴趣爱好：

```json

GET /megacorp/_search?pretty
{
  "size": 0,
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests.keyword"
      }
    }
  }
}

```

暂时忽略掉语法，直接看看结果：

```json

{
  "took": 1,
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
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_interests": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "music",
          "doc_count": 2
        },
        {
          "key": "forestry",
          "doc_count": 1
        },
        {
          "key": "sports",
          "doc_count": 1
        }
      ]
    }
  }
}

```

可以看到，两位员工对音乐感兴趣，一位对林业感兴趣，一位对运动感兴趣。这些聚合的结果数据并非预先统计，而是根据匹配当前查询的文档即时生成的。如果想知道叫 Smith 的员工中最受欢迎的兴趣爱好，可以直接构造一个组合查询：

```json

GET /megacorp/_search
{
  "size": 0,
  "query": {
    "match": {
      "last_name": "smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests.keyword"
      }
    }
  }
}


```

all_interests 聚合已经变为只包含匹配查询的文档：

```json

{
  "took": 1,
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
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_interests": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "music",
          "doc_count": 2
        },
        {
          "key": "sports",
          "doc_count": 1
        }
      ]
    }
  }
}


```

聚合还支持分级汇总 。比如，查询特定兴趣爱好员工的平均年龄：

```json

GET /megacorp/_search
{
  "size": 0, 
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests.keyword"
      },
      "aggs": {
        "avg_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}

```

得到的聚合结果有点儿复杂，但理解起来还是很简单的：

```json

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
    "max_score": null,
    "hits": []
  },
  "aggregations": {
    "all_interests": {
      "doc_count_error_upper_bound": 0,
      "sum_other_doc_count": 0,
      "buckets": [
        {
          "key": "music",
          "doc_count": 2,
          "avg_age": {
            "value": 28.5
          }
        },
        {
          "key": "forestry",
          "doc_count": 1,
          "avg_age": {
            "value": 35
          }
        },
        {
          "key": "sports",
          "doc_count": 1,
          "avg_age": {
            "value": 25
          }
        }
      ]
    }
  }
}

```

输出基本是第一次聚合的加强版。依然有一个兴趣及数量的列表，只不过每个兴趣都有了一个附加的 `avg_age` 属性，代表有这个兴趣爱好的所有员工的平均年龄。

即使现在不太理解这些语法也没有关系，依然很容易了解到复杂聚合及分组通过 `Elasticsearch` 特性实现得很完美，能够提取的数据类型也没有任何限制。
