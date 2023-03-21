# 真题4

## 一、参考

> [真题 4 详解](https://www.yuque.com/deep_elasticsearch/tzcm9n/uosfh8)

## 二、自定义分词

### 2.1 题目

正确的排除 ' 对查询的影响, 确保对 waynes/wayne's, kings/king's 的查询有一样的结果和分值 (custom analyzer)

### 2.2 分析

自定义分词器，char filters 把 ' 过滤

```

PUT yztest-4-1
{
  "settings": {
     "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom", 
          "tokenizer": "standard",
          "char_filter": [
            "my_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_char_filter": {
          "type": "mapping",
          "mappings": [
            "' => "
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "f1": {
        "type": "text",
        "analyzer": "my_custom_analyzer"
      }
    }
  }
}

GET yztest-4-1/_analyze
{
  "field": "f1", 
  "text": "tom's toms"
}

POST yztest-4-1/_bulk
{"index":{}}
{"f1": "tom's"}
{"index":{}}
{"f1": "toms"}

GET yztest-4-1/_search
{
  "query": {
    "match": {
      "f1": "tom's"
    }
  }
}


```


## 三、动态模版

### 3.1 题目

要求将 string 类型映射为 keyword, 将 x_ 开头字段映射为 integer (dynamic template)

### 3.2 分析

动态模板

### 3.3 解答

```

PUT yztest-4-2
{
  "mappings": {
    "dynamic_templates": [
      {
        "long_to_int": {
          "match": "long*",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "string_to_keyword": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}

POST yztest-4-2/_bulk
{"index":{}}
{"long1": 100, "long2": 1.1, "f1": "v1"}

GET yztest-4-2/_mapping

```


## 四、update_by_query + pipeline

### 4.1 题目

为索引添加一个新字段 e 是已有四个字段 a b c d 的拼接 (pipeline + update_by_query)

### 4.2 分析

可以使用 pipeline script 或者 update with script

### 4.3 解答

```

PUT yztest-4-3

POST yztest-4-3/_bulk
{"index": {}}
{"f1": "v1", "f2": "v2", "f3": "v3"}

POST yztest-4-3/_update_by_query
{
  "script": {
    "source": "ctx._source.f4 = ctx._source.f1 + '-' + ctx._source.f2 + '-' + ctx._source.f3",
    "lang": "painless"
  }, 
  "query": {
    "match_all": {}
  }
}

{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "yztest-4-3",
        "_id" : "U84D5IYBHHgM2AP0bStj",
        "_score" : 1.0,
        "_source" : {
          "f1" : "v1",
          "f2" : "v2",
          "f3" : "v3",
          "f4" : "v1-v2-v3"
        }
      }
    ]
  }
}

```


```

PUT yztest-4-3

POST yztest-4-3/_bulk
{"index": {}}
{"f1": "v1", "f2": "v2", "f3": "v3"}


POST yztest-4-3/_update_by_query?pipeline=yztest-4-3-pipeline
{
  "query": {
    "match_all": {}
  }
}

GET yztest-4-3/_search


```


## 五、multi_match

### 5.1 题目

对三个字段 a/b/c 查询 xxx, 要求 c 字段 boost 2, 各字段查询算分加和 (bool query should 或者 multi_match)

### 5.2 分析

multi_match 的 most_fields 和 boost

### 5.3 解答

```
PUT yztest-4-4

POST yztest-4-4/_bulk
{"index": {}}
{"f1": "hello", "f2": "hello hello", "f3": "hello world"}

GET yztest-4-4/_search
{
  "query": {
    "multi_match": {
      "query": "hello",
      "fields": ["f1^2", "f2", "f3"],
      "type": "most_fields"
    }
  }
}


```


## 六、聚合和子聚合

### 6.1 题目

earthquakes 索引按照月份进行聚合, 并统计出每月 magnitude 和 depth 最大值 (date_histogram aggregation + max aggregation)

### 6.2 分析

### 6.3 解答

```
PUT yztest-4-5
{
  "mappings": {
    "properties": {
      "date": {
        "type": "date" 
      },
      "f1": {
        "type": "long"
      }
    }
  }
}

POST yztest-4-5/_bulk
{"index": {}}
{"date": "2023-01-01", "f1": 10}
{"index": {}}
{"date": "2023-01-02", "f1": 20}
{"index": {}}
{"date": "2023-02-01", "f1": 12}
{"index": {}}
{"date": "2023-03-02", "f1": 15}


GET yztest-4-5/_search
{
  "size": 0,
  "aggs": {
    "a1": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "a2": {
          "avg": {
            "field": "f1"
          }
        }
      }
    },
    "a3": {
      "max_bucket": {
        "buckets_path": "a1>a2"
      }
    }
  }
}

{
  "took" : 436,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  },
  "aggregations" : {
    "a1" : {
      "buckets" : [
        {
          "key_as_string" : "2023-01-01T00:00:00.000Z",
          "key" : 1672531200000,
          "doc_count" : 2,
          "a2" : {
            "value" : 15.0
          }
        },
        {
          "key_as_string" : "2023-02-01T00:00:00.000Z",
          "key" : 1675209600000,
          "doc_count" : 1,
          "a2" : {
            "value" : 12.0
          }
        },
        {
          "key_as_string" : "2023-03-01T00:00:00.000Z",
          "key" : 1677628800000,
          "doc_count" : 1,
          "a2" : {
            "value" : 15.0
          }
        }
      ]
    },
    "a3" : {
      "value" : 15.0,
      "keys" : [
        "2023-01-01T00:00:00.000Z",
        "2023-03-01T00:00:00.000Z"
      ]
    }
  }
}
```


## 七、检索 + 高亮

### 7.1 题目

对 movie 的 title 使用 match phrase 查询 star trek, 按照 revenue 倒序排序, title 字段用 <strong></strong> 高亮 (order + highlight)

### 7.2 分析

短语查询，高亮和排序

### 7.3 解答

```

PUT yztest-4-6

POST yztest-4-6/_bulk
{"index": {}}
{"f1": "test doc"}

GET yztest-4-6/_search
{
  "query": {
    "match_phrase": {
      "f1": "test"
    }
  },
  "sort": [
    {
      "f1.keyword": {
        "order": "desc"
      }
    }
  ],
  "highlight": {
    "fields": {
      "f1": { "pre_tags" : ["<strong>"], "post_tags" : ["</strong>"] }
    }
  }
}


{
  "took" : 1689,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [
      {
        "_index" : "yztest-4-6",
        "_id" : "Ws4u5IYBHHgM2AP0Gisz",
        "_score" : null,
        "_source" : {
          "f1" : "test doc"
        },
        "highlight" : {
          "f1" : [
            "<strong>test</strong> doc"
          ]
        },
        "sort" : [
          "test doc"
        ]
      }
    ]
  }
}


```
