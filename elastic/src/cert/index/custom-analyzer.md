# 自定义分词器


## 一、参考

> [自定义分词，从一个问题说开去](https://www.yuque.com/deep_elasticsearch/tzcm9n/efkd3i)


## 二、分词器组成

### 2.1 char filter

主要是对字符进行处理

一个分词器可以有多个 char filter

### 2.2 tokenizer

分词器，将字符流变为 token 列表

一个分词器只能有一个 tokenizer

### 2.3 token filter

主要对token，进行过滤

一个分词器可以有多个 token filter

## 三、场景

来自星友的一个真实业务场景问题：

我现在的业务需求是这样的。有一个作者字段，比如是这样的Li,LeiLei;Han,MeiMei;

还有一些是LeiLei Li...。

现在要精确匹配。 

我的想法是：用自定义分词通过分号分词。

但是这样我检索Li,LeiLei那么LeiLei Li就不能搜索到，

我希望的结果是LeiLei Li也被搜索到

而且这种分词，Li,LeiLei不加逗号，也不能匹配到。

但是不知道为什么我在mapping里面添加停用词也不管用？

### 3.1 思考

因为是精确匹配，

(1) 需要在字符过滤阶段，删除逗号, 但是不能删除其他特殊符号

(2) 需要按照分号拆分字符流为单词

(3) 需要添加一些同义词，即 LeiLei Li 等同于 Li LeiLei

(4) 不需要区分大小写

### 3.2 创建自定义分词器

```

PUT test-custom-analyzer
{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "custom_char_filter"
          ],
          "tokenizer": "custom_tokenizer",
          "filter": [
            "lowercase",
            "custom_filter"
          ]
        }
      },
      "char_filter": {
        "custom_char_filter": {
          "type": "mapping",
          "mappings": [
            ", => "
          ]
        }
      },
      "tokenizer": {
        "custom_tokenizer": {
          "type": "char_group",
          "tokenize_on_chars": [
            ";"
          ]
        }
      },
      "filter": {
        "custom_filter": {
          "type": "synonym",
          "lenient": true,
          "synonyms": [
            "li leilei => leileili",
            "han meimei => meimeihan",
            "lileilei => leileili",
            "hanmeimei => meimeihan"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "message": {
        "type": "text",
        "analyzer": "custom_analyzer"
      }
    }
  }
}

```


### 3.3 验证搜索

```

GET test-custom-analyzer/_analyze
{
  "field": "message",
  "text": "Li leilei"
}

{
  "tokens" : [
    {
      "token" : "leileili",
      "start_offset" : 0,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 0
    }
  ]
}

GET test-custom-analyzer/_analyze
{
  "field": "message",
  "text": "Li,leilei"
}

{
  "tokens" : [
    {
      "token" : "leileili",
      "start_offset" : 0,
      "end_offset" : 9,
      "type" : "SYNONYM",
      "position" : 0
    }
  ]
}

```
