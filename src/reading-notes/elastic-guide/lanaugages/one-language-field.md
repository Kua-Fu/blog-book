# 每个域一种语言

对于一些实体类，例如:产品、电影、法律声明， 通常这样的一份文本会被翻译成不同语言的文档。虽然这些不同语言的文档可以单独保存在各自的索引中。但另一种更合理的方式是同一份文本的所有翻译统一保存在一个索引中。。

```python

{
   "title":     "Fight club",
   "title_br":  "Clube de Luta",
   "title_cz":  "Klub rváčů",
   "title_en":  "Fight club",
   "title_es":  "El club de la lucha",
   ...
}

```

每份翻译存储在不同的域中，根据域的语言决定使用相应的分析器：

```python

PUT /movies
{
  "mappings": {
    "properties": {
      "title": { 0️⃣
        "type": "text"
      },
      "title_br": {
        "type": "text",
        "analyzer": "brazilian" 1️⃣
      },
      "title_cz": {
        "type": "text",
        "analyzer": "czech"
      },
      "title_en": {
        "type": "text",
        "analyzer": "english"
      },
      "title_es": {
        "type": "text",
        "analyzer": "spanish"
      }
    }
  }
}

```

0️⃣ title 域含有title的原文，并使用 standard （标准）分析器。

1️⃣ 其他字段使用适合自己语言的分析器。

在维持干净的词频方面，虽然 index-per-language （一种语言一份索引的方法），不像 field-per-language （一种语言一个域的方法）分开索引那么灵活。但是使用 update-mapping API 添加一个新域也很简单，那些新域需要新的自定义分析器，这些新分析器只能在索引创建时被装配。有一个变通的方案，你可以先关闭这个索引 close ，然后使用 update-settings API ，重新打开这个索引，但是关掉这个索引意味着得停止服务一段时间。

文档的一种语言可以单独查询，也可以通过查询多个域来查询多种语言。我们甚至可以通过对特定语言设置偏好来提高字段优先级：

```python

GET /movies/_search
{
  "query": {
    "multi_match": {
      "query": "club de la lucha", 0️⃣
      "fields": [
        "title*",
        "title_es^2"
      ],
      "type": "most_fields"
    }
  }
}


```

	
这个搜索查询所有以 title 为前缀的域，但是对 title_es 域加权重 2 。其他的所有域是中性权重 1 。


