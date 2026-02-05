# 内部结构

## 一、参考

> [内部数据结构深度解读](https://www.yuque.com/deep_elasticsearch/tzcm9n/glo283)


## 二、doc values


doc values 是正排索引，默认除了text 类型，都开启

## 三、fielddata

text 类型用于聚合、排序，默认禁用

## 四、_source

实际的原始文档，不支持搜索，但是可以通过搜索返回； 默认开启

## 五、store

是否保存某些字段，默认不保存


```
PUT yztest-stored/
{
  "mappings": {
    "properties": {
      "message": {
        "type": "text",
        "store": true
      }
    }
  }
}


POST yztest-stored/_doc
{
  "message": "test message"
}

GET yztest-stored/_search
{
  "stored_fields": ["message"]
}


```
