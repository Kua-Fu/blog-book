# runtime字段


## 一、参考

> [运行时类型 Runtime fields 深入详解](https://www.yuque.com/deep_elasticsearch/tzcm9n/ql434d)

> [runtime fields](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/runtime.html)

> [Ingest pipelines](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/ingest.html)

> [Update By Query](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/docs-update-by-query.html)

## 二、场景

现在，我们有一个索引，保存着对于某个电影的评分，其中字段 grade 表示评分，范围是 1-10, 现在，我们希望添加一个新字段 evaluation， 表示对于该电影的评价，其中可选值为 

good [8, 10]

bad [1, 5)

common [5, 8)


```
PUT test-runtime
{
  "mappings": {
    "properties": {
      "grade": {
        "type": "integer"
      }
    }
  }
}
 
 
POST test-runtime/_bulk
{"index":{"_id":1}}
{"grade":1}
{"index":{"_id":2}}
{"grade":10}
{"index":{"_id":3}}
{"grade":5}
{"index":{"_id":4}}
{"grade":7}
```

## 三、使用管道实现

### 3.1 创建管道

```

PUT _ingest/pipeline/test-runtime-pipeline
{
  "processors": [
    {
      "script": {
        "description": "add new field, evaluation",
        "lang": "painless",
        "source": """
        if (ctx['grade'] >= 1 && ctx['grade'] < 5)
          ctx['evaluation'] = 'bad';
        if (ctx['grade'] >= 5 && ctx['grade'] < 8)
          ctx['evaluation'] = 'common';
        if (ctx['grade'] >= 8 && ctx['grade'] <= 10)
          ctx['evaluation'] = 'good';
        """
      }
    }
  ]
}
```

### 3.2 更新文档

```

POST test-runtime/_update_by_query?pipeline=test-runtime-pipeline
{
  "query": {
    "match_all": {}
  }
}

```

### 3.3 查看文档

```

GET test-runtime/_search

{
  "took" : 12,
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
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test-runtime",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "evaluation" : "bad",
          "grade" : 1
        }
      },
      {
        "_index" : "test-runtime",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "evaluation" : "good",
          "grade" : 10
        }
      },
      {
        "_index" : "test-runtime",
        "_id" : "3",
        "_score" : 1.0,
        "_source" : {
          "evaluation" : "common",
          "grade" : 5
        }
      },
      {
        "_index" : "test-runtime",
        "_id" : "4",
        "_score" : 1.0,
        "_source" : {
          "evaluation" : "common",
          "grade" : 7
        }
      }
    ]
  }
}


```


## 四、使用script_fields 实现

### 4.1 查询语句 


```
GET test-runtime/_search
{
  "fields":["*"], 
  "query": {
    "match_all": {}
  },
  "script_fields": {
    "evaluation_new": {
      "script": {
        "lang": "painless",
        "source": """
        if (doc['grade'].value >=1 && doc['grade'].value < 5)
          return 'bad';
        if (doc['grade'].value >=5 && doc['grade'].value < 8)
          return 'common';
        if (doc['grade'].value >=8 && doc['grade'].value <= 10)
          return 'good';
        """
      }
    }
  }
}

```

### 4.2 查询结果

```
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
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test-runtime",
        "_id" : "1",
        "_score" : 1.0,
        "fields" : {
          "evaluation" : [
            "bad"
          ],
          "evaluation_new" : [
            "bad"
          ],
          "grade" : [
            1
          ],
          "evaluation.keyword" : [
            "bad"
          ]
        }
      },
      {
        "_index" : "test-runtime",
        "_id" : "2",
        "_score" : 1.0,
        "fields" : {
          "evaluation" : [
            "good"
          ],
          "evaluation_new" : [
            "good"
          ],
          "grade" : [
            10
          ],
          "evaluation.keyword" : [
            "good"
          ]
        }
      },
      {
        "_index" : "test-runtime",
        "_id" : "3",
        "_score" : 1.0,
        "fields" : {
          "evaluation" : [
            "common"
          ],
          "evaluation_new" : [
            "common"
          ],
          "grade" : [
            5
          ],
          "evaluation.keyword" : [
            "common"
          ]
        }
      },
      {
        "_index" : "test-runtime",
        "_id" : "4",
        "_score" : 1.0,
        "fields" : {
          "evaluation" : [
            "common"
          ],
          "evaluation_new" : [
            "common"
          ],
          "grade" : [
            7
          ],
          "evaluation.keyword" : [
            "common"
          ]
        }
      }
    ]
  }
}



```


## 五、使用runtime fields

### 5.1 查询时候指定mapping

```

GET test-runtime/_search
{
  "fields":["*"], 
  "query": {
    "match_all": {}
  },
  "runtime_mappings": {
    "evaluation_new": {
      "type": "keyword",
      "script": {
        "lang": "painless",
        "source": """
        if (doc['grade'].value >=1 && doc['grade'].value < 5){
          emit('bad')
        }
        if (doc['grade'].value >=5 && doc['grade'].value < 8){
          emit('common')
        }
        if (doc['grade'].value >=8 && doc['grade'].value <= 10){
          emit('good')
        }
        """
      }
    }
  }
}

```

### 5.2 定义mapping

```

PUT test-runtime/_mapping
{
  "runtime": {
    "evaluation_new": {
      "type": "keyword",
      "script": {
        "lang": "painless",
        "source": """
        if (doc['grade'].value >=1 && doc['grade'].value < 5)
          emit('bad');
        if (doc['grade'].value >=5 && doc['grade'].value < 8)
          emit('common');
        if (doc['grade'].value >=8 && doc['grade'].value <= 10)
          emit('good');
      """
      }
    }
  }
}

```
