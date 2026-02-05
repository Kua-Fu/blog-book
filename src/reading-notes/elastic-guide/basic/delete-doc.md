# 删除文档

删除文档的语法和我们所知道的规则相同，只是使用 DELETE 方法：

```json

DELETE /website/_doc/123

{
  "_index": "website",
  "_id": "123",
  "_version": 3, 0️⃣
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 3,
  "_primary_term": 1
}

```

如果找到该文档，Elasticsearch 将要返回一个 200 ok 的 HTTP 响应码，和一个类似以下结构的响应体。注意，字段 _version 值已经增加:


如果文档没有找到，我们将得到 404 Not Found 的响应码和类似这样的响应体：

```json

{
  "found" :    false,
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 4
}

```

即使文档不存在（ Found 是 false ）， _version 值仍然会增加。这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。

> 🦉**Note**
>
> 正如已经在更新整个文档中提到的，删除文档不会立即将文档从磁盘中删除，只是将文档标记为已删除状态。随着你不断的索引更多的数据，Elasticsearch 将会在后台清理标记为已删除的文档。
