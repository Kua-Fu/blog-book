# 创建新文档

当我们索引一个文档，怎么确认我们正在创建一个完全新的文档，而不是覆盖现有的呢？

请记住， _index 、 _type 和 _id 的组合可以唯一标识一个文档。所以，确保创建一个新文档的最简单办法是，使用索引请求的 POST 形式让 Elasticsearch 自动生成唯一 _id :

```json

POST /website/blog/
{ ... }

```

然而，如果已经有自己的 _id ，那么我们必须告诉 Elasticsearch ，只有在相同的 _index 、 _type 和 _id 不存在时才接受我们的索引请求。这里有两种方式，他们做的实际是相同的事情。使用哪种，取决于哪种使用起来更方便。

第一种方法使用 op_type 查询-字符串参数：

```json

PUT /website/blog/123?op_type=create
{ ... }

```

第二种方法是在 URL 末端使用 /_create :

```json

PUT /website/blog/123/_create
{ ... }

```

如果创建新文档的请求成功执行，Elasticsearch 会返回元数据和一个 201 Created 的 HTTP 响应码。

另一方面，如果具有相同的 _index 、 _type 和 _id 的文档已经存在，Elasticsearch 将会返回 409 Conflict 响应码，以及如下的错误信息：

```json

POST /website/_doc/123?op_type=create
{
  "title": "My first blog entry",
  "text": "I am starting to get the hang of this...",
  "date": "2014/01/02"
}

{
  "error": {
    "root_cause": [
      {
        "type": "version_conflict_engine_exception",
        "reason": "[123]: version conflict, document already exists (current version [2])",
        "index_uuid": "SjLz7UQ6Q0WbQWLXRl-KeQ",
        "shard": "0",
        "index": "website"
      }
    ],
    "type": "version_conflict_engine_exception",
    "reason": "[123]: version conflict, document already exists (current version [2])",
    "index_uuid": "SjLz7UQ6Q0WbQWLXRl-KeQ",
    "shard": "0",
    "index": "website"
  },
  "status": 409
}

```
