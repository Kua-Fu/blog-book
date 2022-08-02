# 取回多个文档

Elasticsearch 的速度已经很快了，但甚至能更快。 将多个请求合并成一个，避免单独处理每个请求花费的网络延时和开销。 如果你需要从 Elasticsearch 检索很多文档，那么使用 multi-get 或者 mget API 来将这些检索请求放在一个请求中，将比逐个文档请求更快地检索到全部文档。

mget API 要求有一个 docs 数组作为参数，每个元素包含需要检索文档的元数据， 包括 _index 、 _type 和 _id 。如果你想检索一个或者多个特定的字段，那么你可以通过 _source 参数来指定这些字段的名字：

```json

GET /_mget
{
  "docs": [
    {
      "_index": "website",
      "_id": 2
    },
    {
      "_index": "website",
      "_id": 1,
      "_source": "views"
    }
  ]
}

{
  "docs": [
    {
      "_index": "website",
      "_id": "2",
      "_version": 10,
      "_seq_no": 6,
      "_primary_term": 1,
      "found": true,
      "_source": {
        "title": "My first external blog entry",
        "text": "This is a piece of cake..."
      }
    },
    {
      "_index": "website",
      "_id": "1",
      "_version": 1,
      "_seq_no": 13,
      "_primary_term": 1,
      "found": true,
      "_source": {
        "views": 0
      }
    }
  ]
}


```

该响应体也包含一个 docs 数组， 对于每一个在请求中指定的文档，这个数组中都包含有一个对应的响应，且顺序与请求中的顺序相同。 其中的每一个响应都和使用单个 get request 请求所得到的响应体相同：

如果想检索的数据都在相同的 _index 中（甚至相同的 _type 中），则可以在 URL 中指定默认的 /_index 或者默认的 /_index/_type 。

你仍然可以通过单独请求覆盖这些值

注意，我们请求的第二个文档是不存在的。我们指定类型为 blog ，但是文档 ID 1 的类型是 pageviews ，这个不存在的情况将在响应体中被报告：

```json

GET /_mget
{
  "docs": [
    {
      "_index": "website",
      "_id": 2
    },
    {
      "_index": "website",
      "_id": 12,
      "_source": "views"
    }
  ]
}


{
  "docs": [
    {
      "_index": "website",
      "_id": "2",
      "_version": 10,
      "_seq_no": 6,
      "_primary_term": 1,
      "found": true,
      "_source": {
        "title": "My first external blog entry",
        "text": "This is a piece of cake..."
      }
    },
    {
      "_index": "website",
      "_id": "12",
      "found": false 0️⃣
    }
  ]
}

```

0️⃣ 未找到该文档。

事实上第二个文档未能找到并不妨碍第一个文档被检索到。每个文档都是单独检索和报告的。

>🦉 **Note**
>
> 即使有某个文档没有找到，上述请求的 HTTP 状态码仍然是 200 。事实上，即使请求 没有 找到任何文档，它的状态码依然是 200 --因为 mget 请求本身已经成功执行。 为了确定某个文档查找是成功或者失败，你需要检查 found 标记。
