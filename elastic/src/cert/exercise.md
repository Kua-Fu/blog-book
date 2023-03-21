# 真题解析

## 1. reindex

(1) 把source index 的某个字段(该字段是数组）中的子项，去除前后空格

(2) 增加一个字段，这个新的字段的值是 source index 的某两个字段的拼接

解析: 

主要考察 reindex with pipeline

其中，数组遍历可以使用  foreach, 去除前后空格可以使用 trim, 字段拼接可以使用 script

```

POST _ingest/pipeline/_simulate
{
  "pipeline": {
    "processors": [
      {
        "foreach": {
          "field": "f3",
          "processor": {
            "trim": {
              "field": "_ingest._value"
            }
          }
        }
      },
      {
        "script": {
          "lang": "painless",
          "source": "ctx['f4'] = ctx['f1'] + ctx['f2']"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "yztest-1",
      "_source": {
        "f1": "11",
        "f2": "22",
        "f3": [
          " 1 ",
          " 2 "
        ]
      }
    }
  ]
}

```


(1) create pipeline

```
PUT _ingest/pipeline/yztest-pipeline-1
{
  "description": "pipeline 1",
  "processors": [
    {
      "foreach": {
        "field": "f3",
        "processor": {
          "trim": {
            "field": "_ingest._value"
          }
        }
      }
    },
    {
      "script": {
        "lang": "painless",
        "source": "ctx['f4'] = ctx['f1'] + ctx['f2']"
      }
    }
  ]
}
```

(2) create index

```

PUT yztest-1

POST yztest-1/_doc
{
  "f1": "11",
  "f2": "22",
  "f3": [
    " 1 ",
    " 2 "
  ]
}

PUT yztest-1-dest

```

(3) reindex

```

POST _reindex
{
  "source": {
    "index": "yztest-1"
  },
  "dest": {
    "index": "yztest-1-dest",
    "pipeline": "yztest-pipeline-1"
  }
}
```
