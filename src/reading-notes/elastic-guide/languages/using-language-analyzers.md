# 使用语言分析器

Elasticsearch 的内置分析器都是全局可用的，不需要提前配置，它们也可以在字段映射中直接指定在某字段上：

```python

PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": {
          "type":     "string",
          "analyzer": "english" 0️⃣
        }
      }
    }
  }
}

```


```python

# ES 8.4.1 

PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english" 0️⃣
      }
    }
  }
}

```

0️⃣ title 字段将会用 english（英语）分析器替换默认的 standard（标准）分析器

当然，文本经过 english 分析处理，我们会丢失源数据：

```python

GET /my_index/_analyze
{
  "field": "title", 
  "text": "I'm not happy about the foxes"
}

// 返回值 
{
  "tokens": [ 0️⃣
    {
      "token": "i'm",
      "start_offset": 0,
      "end_offset": 3,
      "type": "<ALPHANUM>",
      "position": 0
    },
    {
      "token": "happi",
      "start_offset": 8,
      "end_offset": 13,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "about",
      "start_offset": 14,
      "end_offset": 19,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "fox",
      "start_offset": 24,
      "end_offset": 29,
      "type": "<ALPHANUM>",
      "position": 5
    }
  ]
}

```

0️⃣  切词为: i'm，happi，about，fox

我们无法分辨源文档中是

1. 包含单数 fox 还是复数 foxes ；

1. 单词 not 因为是停用词所以被移除了， 所以我们无法分辨源文档中是happy about foxes还是not happy about foxes，

虽然通过使用 english （英语）分析器，使得匹配规则更加宽松，我们也因此提高了召回率，但却降低了精准匹配文档的能力。

为了获得两方面的优势，我们可以使用multifields（多字段）对 title 字段建立两次索引： 一次使用 english（英语）分析器，另一次使用 standard（标准）分析器:

```python

PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text", 0️⃣
        "fields": {
          "english": {
            "type": "text",
            "analyzer": "english" 1️⃣
          }
        }
      }
    }
  }
}


```

0️⃣ 主 title 字段使用 standard（标准）分析器。
	
1️⃣ title.english 子字段使用 english（英语）分析器。


替换为该字段映射后，我们可以索引一些测试文档来展示怎么在搜索时使用两个字段：

```python

PUT /my_index/_doc/1
{ "title": "I'm happy for this fox" }

PUT /my_index/_doc/2
{ "title": "I'm not happy about my fox problem" }

```

```python

GET my_index/_search
{
  "query": {
    "multi_match": {
      "type": "most_fields", 0️⃣
      "query": "not happy foxes",
      "fields": [
        "title",
        "title.english"
      ]
    }
  }
}
```

0️⃣ 使用most_fields query type（多字段搜索语法来）让我们可以用多个字段来匹配同一段文本。

感谢 title.english 字段的切词，无论我们的文档中是否含有单词 foxes 都会被搜索到，第二份文档的相关性排行要比第一份高， 因为在 title 字段中匹配到了单词 not 。
