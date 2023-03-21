# 7.13真题


## 一、参考

> [7.13新真题全梳理](https://www.yuque.com/deep_elasticsearch/tzcm9n/gykbi7)

## 二、runtime field

### 2.1 题目

在task6索引里，创建一个runtime字段，其值是A-B，A,B为字段；创建一个range聚合，分为三级：小于0，0-100，100以上；返回文档数为0

### 2.2 分析

runtime fields 和 range aggs

### 2.3 解答

```

PUT yztest-5-1

POST yztest-5-1/_bulk
{"index": {}}
{"f1": 1, "f2": 10}
{"index": {}}
{"f1": 10, "f2": 100}
{"index": {}}
{"f1": 100, "f2": 1000}


GET yztest-5-1/_search
{
  "size": 0, 
  "runtime_mappings": {
    "f3": {
      "type": "long",
      "script": {
        "source": """
          emit(doc['f2'].value - doc['f1'].value);
          """
      }
    }
  },
  "aggs": {
    "a1": {
      "range": {
        "field": "f3",
        "ranges": [
          {
            "to": 0
          },
          {
            "from": 0,
            "to": 100
          },
          {
            "from": 100
          }
        ]
      }
    }
  }
}


```
