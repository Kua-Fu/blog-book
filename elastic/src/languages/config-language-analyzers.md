# 配置语言分析器

语言分析器都不需要任何配置，开箱即用， 它们中的大多数都允许你控制它们的各方面行为，具体来说：

* 词干提取排除

	想象下某个场景，用户们想要搜索 World Health Organization 的结果， 但是却被替换为搜索 organ health 的结果。有这个困惑是因为 organ 和 organization 有相同的词根： organ 。 通常这不是什么问题，但是在一些特殊的文档中就会导致有歧义的结果，所以我们希望防止单词 organization 和 organizations 被缩减为词干。

* 自定义停用词

	英语中默认的停用词列表如下：
	
	a, an, and, are, as, at, be, but, by, for, if, in, into, is, it,
	
	no, not, of, on, or, such, that, the, their, then, there, these,
	
	they, this, to, was, will, with
	
	关于单词 no 和 not 有点特别，这俩词会反转跟在它们后面的词汇的含义。或许我们应该认为这两个词很重要，不应该把他们看成停用词。
	
为了自定义 english （英语）分词器的行为，我们需要基于 english （英语）分析器创建一个自定义分析器，然后添加一些配置：

```python

PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_english": {
          "type": "english",
          "stem_exclusion": [  0️⃣
            "organization",
            "organizations"
          ],
          "stopwords": [ 1️⃣
            "a",
            "an",
            "and",
            "are",
            "as",
            "at",
            "be",
            "but",
            "by",
            "for",
            "if",
            "in",
            "into",
            "is",
            "it",
            "of",
            "on",
            "or",
            "such",
            "that",
            "the",
            "their",
            "then",
            "there",
            "these",
            "they",
            "this",
            "to",
            "was",
            "will",
            "with"
          ]
        }
      }
    }
  }
}
``

 0️⃣ 防止 organization 和 organizations 被缩减为词干
 
 1️⃣ 指定一个自定义停用词列表
 
 

```python

GET /my_index/_analyze
{
  "analyzer": "my_english",
  "text": "The World Health Organization does not sell organs."
}

{
  "tokens": [ 0️⃣
    {
      "token": "world",
      "start_offset": 4,
      "end_offset": 9,
      "type": "<ALPHANUM>",
      "position": 1
    },
    {
      "token": "health",
      "start_offset": 10,
      "end_offset": 16,
      "type": "<ALPHANUM>",
      "position": 2
    },
    {
      "token": "organization",
      "start_offset": 17,
      "end_offset": 29,
      "type": "<ALPHANUM>",
      "position": 3
    },
    {
      "token": "doe",
      "start_offset": 30,
      "end_offset": 34,
      "type": "<ALPHANUM>",
      "position": 4
    },
    {
      "token": "not",
      "start_offset": 35,
      "end_offset": 38,
      "type": "<ALPHANUM>",
      "position": 5
    },
    {
      "token": "sell",
      "start_offset": 39,
      "end_offset": 43,
      "type": "<ALPHANUM>",
      "position": 6
    },
    {
      "token": "organ",
      "start_offset": 44,
      "end_offset": 50,
      "type": "<ALPHANUM>",
      "position": 7
    }
  ]
}

```

0️⃣  切词为 world 、 health 、 organization 、 does 、 not 、 sell 、 organ

我们在 将单词还原为词根 和 停用词: 性能与精度 中分别详细讨论了词干提取和停用词。
