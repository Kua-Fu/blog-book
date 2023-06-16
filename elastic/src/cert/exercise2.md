# 真题2

## 一、参考

> [真题 2 详解](https://www.yuque.com/deep_elasticsearch/tzcm9n/hdo7ov)

## 二、自定义分词

## 2.1 题目

有一个文档，内容类似dog & cat， 要求索引这条文档，

并且使用match_phrase query，查询dog & cat或者dog and cat都能match

## 2.2 分析

考察点：自定义分词、同义词。

即需要将 & 和 and 匹配为同义词，可以在 char_filter阶段

## 2.3 解答

```

PUT yztest-2-1
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "char_filter": [
            "my_mappings_char_filter"
          ]
        }
      },
      "char_filter": {
        "my_mappings_char_filter": {
          "type": "mapping",
          "mappings": [
            "& => and"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "message": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}

POST yztest-2-1/_bulk
{"index": {}}
{"message": "dog & cat"}
{"index": {}}
{"message": "dog and cat"}

GET yztest-2-1/_search
{
  "query": {
    "match_phrase": {
      "message": "dog & cat"
    }
  }
}

```


## 三、reindex 

### 3.1 题目

有index_a包含一些文档， 

要求创建索引index_b，通过reindex api将index_a的文档索引到index_b。 

要求增加一个整形字段，value是index_a的field_x的字符长度； 

再增加一个数组类型的字段，value是field_y的词集合。(field_y是空格分割的一组词，比方"foo bar"，索引到index_b后，要求变成["foo", "bar"]

### 3.2 分析

需要通过 pipeline + reindex 实现，pipeline 中需要使用 split 和 script

### 3.3 先使用 simulate 模拟

```
POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "processors": [
      {
       "split": {
         "field": "f1",
         "separator": " "
       }
      },
      {
        "script": {
          "lang": "painless", 
          "source": "ctx['f3'] = ctx['f2'].length()"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_source": {
        "f1": "v1 v2",
        "f2": "111"
      }
    }
  ]
}

{
  "docs" : [
    {
      "doc" : {
        "_index" : "index",
        "_id" : "_id",
        "_source" : {
          "f1" : [
            "v1",
            "v2"
          ],
          "f2" : "111",
          "f3" : 3
        },
        "_ingest" : {
          "timestamp" : "2023-03-15T05:43:30.46806Z"
        }
      }
    }
  ]
}

```


### 3.4 reindex

```
PUT _ingest/pipeline/yztest-2-2-pipeline
{
  "description": "test",
  "processors": [
    {
      "set": {
        "field": "f3",
        "value": "{{f1}}"
      }
    },
    {
      "split": {
        "field": "f3",
        "separator": " "
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": "ctx['f4'] = ctx['f2'].length()"
      }
    }
  ]
}

PUT yztest-2-2-source

PUT yztest-2-2-dest


POST yztest-2-2-source/_doc
{
  "f1": "v1 v2 v3",
  "f2": "test"
}


POST _reindex
{
  "source": {
    "index": "yztest-2-2-source"
  },
  "dest": {
    "index": "yztest-2-2-dest",
    "pipeline": "yztest-2-2-pipeline"
  }
}


GET yztest-2-2-dest/_search

{
  "took" : 9,
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
        "_index" : "yztest-2-2-dest",
        "_id" : "RM7N44YBHHgM2AP0Qysd",
        "_score" : 1.0,
        "_source" : {
          "f1" : "v1 v2 v3",
          "f2" : "test",
          "f3" : [
            "v1",
            "v2",
            "v3"
          ],
          "f4" : 4
        }
      }
    ]
  }
}


```

## 四、布尔组合查询

### 4.1 题目

按要求写一个查询， 其中一个条件是某个关键词必须包含在4个字段中至少2个

### 4.2 分析

考察点: bool 组合查询，should查询的 minimum_should_match参数

### 4.3 解决

```

PUT yztest-2-3

POST yztest-2-3/_bulk
{"index": {}}
{"f1": "hello", "f2": "hello", "f3": "he", "f4": "h"}
{"index": {}}
{"f1": "hello", "f2": "hell", "f3": "he", "f4": "h"}
{"index": {}}
{"f1": "hello", "f2": "hell", "f3": "hello", "f4": "h"}

GET yztest-2-3/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "f1": "hello"
          }
        }
      ],
      "should": [
        {
          "match": {
            "f2": "hello"
          }
        },
        {
          "match": {
            "f3": "hello"
          }
        },
        {
          "match": {
            "f4": "hello"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}


{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.1143606,
    "hits" : [
      {
        "_index" : "yztest-2-3",
        "_id" : "Rs7f44YBHHgM2AP0Uivz",
        "_score" : 1.1143606,
        "_source" : {
          "f1" : "hello",
          "f2" : "hello",
          "f3" : "he",
          "f4" : "h"
        }
      },
      {
        "_index" : "yztest-2-3",
        "_id" : "SM7f44YBHHgM2AP0Uivz",
        "_score" : 1.1143606,
        "_source" : {
          "f1" : "hello",
          "f2" : "hell",
          "f3" : "hello",
          "f4" : "h"
        }
      }
    ]
  }
}
```

## 五、search template

### 5.1 题目

按照要求写一个search template

### 5.2 按照官方文档

```

PUT yztest-2-4

POST yztest-2-4/_bulk
{"index": {}}
{"f1": "hello world"}
{"index": {}}
{"f1": "hello"}
{"index": {}}
{"f1": "world"}



PUT _scripts/yztest-2-4-search-template
{
  "script": {
    "lang": "mustache",
    "source": {
      "query": {
        "match": {
          "f1": "{{query_string}}"
        }
      },
      "from": "{{from}}",
      "size": "{{size}}"
    },
    "params": {
      "query_string": "hello world"
    }
  }
}

GET yztest-2-4/_search/template
{
  "id": "yztest-2-4-search-template",
  "params": {
    "query_string": "hello world",
    "from": 0,
    "size": 2
  }
}

```