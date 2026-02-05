# 真题3

## 一、参考

> [真题 3 详解](https://www.yuque.com/deep_elasticsearch/tzcm9n/dq3bn4)

## 二、检索 + 聚合

### 2.1 题目

文档的字段有制造商，原材料，最终加工成的食品，满足如下条件

（这道题的题目比较艰涩，理解这道题花了一些时间，还好有google翻译)

* a.食品用了某种原材料
* b.返回生产这些食品的数量前10的制造商（这个生产食品数就是文档数量，某个制造商的文档数量越多，排名越靠前）
  
  
###  2.2 分析

简单的match 然后 terms 聚合

### 2.3 解答

```

PUT yztest-3-1

POST yztest-3-1/_bulk
{"index": {}}
{"f1": "v1", "f2": "v2"}
{"index": {}}
{"f1": "v2", "f2": "v1"}
{"index": {}}
{"f1": "v3", "f2": "v2"}

POST yztest-3-1/_search
{
  "query": {
    "match": {
      "f1": "v1"
    }
  },
  "aggs": {
    "top10": {
      "terms": {
        "field": "f2.keyword",
        "size": 10
      }
    }
  }
}

```
