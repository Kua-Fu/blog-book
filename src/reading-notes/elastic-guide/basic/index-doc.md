# 索引文档

通过使用 index API ，文档可以被 索引 —— 存储和使文档可被搜索。 但是首先，我们要确定文档的位置。正如我们刚刚讨论的，一个文档的 _index 、 _type 和 _id 唯一标识一个文档。 我们可以提供自定义的 _id 值，或者让 index API 自动生成。

## 一、使用自定义的ID

如果你的文档有一个自然的标识符 （例如，一个 user_account 字段或其他标识文档的值），你应该使用如下方式的 index API 并提供你自己 _id ：

```json

PUT /{index}/{type}/{id}
{
  "field": "value",
  ...
}

```

举个例子，如果我们的索引称为 website ，类型称为 blog ，并且选择 123 作为 ID ，那么索引请求应该是下面这样：

```json

PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}

```

```json

POST /website/_doc/123
{
  "type": "blog",
  "title": "My first blog entry",
  "text": "Just trying this out...",
  "date": "2014/01/01"
}

{
  "_index": "website",
  "_id": "123",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 0,
  "_primary_term": 1
}

```

该响应表明文档已经成功创建，该索引包括 _index 、 _type 和 _id 元数据， 以及一个新元素： _version 。

在 Elasticsearch 中每个文档都有一个版本号。当每次对文档进行修改时（包括删除）， _version 的值会递增。 在 处理冲突 中，我们讨论了怎样使用 _version 号码确保你的应用程序中的一部分修改不会覆盖另一部分所做的修改。

## 二、自生成ID

如果你的数据没有自然的 ID， Elasticsearch 可以帮我们自动生成 ID 。 请求的结构调整为： 不再使用 PUT 谓词(“使用这个 URL 存储这个文档”)， 而是使用 POST 谓词(“存储文档在这个 URL 命名空间下”)。

现在该 URL 只需包含 _index 和 _type :

```json

POST /website/blog/
{
  "title": "My second blog entry",
  "text":  "Still trying this out...",
  "date":  "2014/01/01"
}

```

除了 _id 是 Elasticsearch 自动生成的，响应的其他部分和前面的类似：

```json

POST /website/_doc
{
  "type": "blog",
  "title": "My second blog entry",
  "text": "Still trying this out...",
  "date": "2014/01/01"
}

{
  "_index": "website",
  "_id": "jrkq4oEBZEeCE-yEdE3s",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 1,
  "_primary_term": 1
}

```

自动生成的 ID 是 URL-safe、 基于 Base64 编码且长度为20个字符的 GUID 字符串。 这些 GUID 字符串由可修改的 FlakeID 模式生成，这种模式允许多个节点并行生成唯一 ID ，且互相之间的冲突概率几乎为零。
