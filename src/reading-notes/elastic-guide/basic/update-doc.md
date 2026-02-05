# 更新整个文档

在 Elasticsearch 中文档是 不可改变 的，不能修改它们。相反，如果想要更新现有的文档，需要 重建索引 或者进行替换， 我们可以使用相同的 index API 进行实现，在 索引文档 中已经进行了讨论。


```json

POST /website/_doc/123
{
  "title": "My first blog entry",
  "text": "I am starting to get the hang of this...",
  "date": "2014/01/02"
}

{
  "_index": "website",
  "_id": "123",
  "_version": 2, 0️⃣
  "result": "updated", 1️⃣
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 2,
  "_primary_term": 1
}

```

在响应体中，我们能看到 Elasticsearch 已经增加了 _version 字段值：

在内部，Elasticsearch 已将旧文档标记为已删除，并增加一个全新的文档。 尽管你不能再对旧版本的文档进行访问，但它并不会立即消失。当继续索引更多的数据，Elasticsearch 会在后台清理这些已删除文档。

在本章的后面部分，我们会介绍 update API, 这个 API 可以用于 partial updates to a document 。 虽然它似乎对文档直接进行了修改，但实际上 Elasticsearch 按前述完全相同方式执行以下过程：

1. 从旧文档构建 JSON

1. 更改该 JSON

1. 删除旧文档

1. 索引一个新文档

唯一的区别在于, update API 仅仅通过一个客户端请求来实现这些步骤，而不需要单独的 get 和 index 请求。
