# 面向文档

在应用程序中对象很少只是一个简单的键和值的列表。通常，它们拥有更复杂的数据结构，可能包括日期、地理信息、其他对象或者数组等。

也许有一天你想把这些对象存储在数据库中。使用关系型数据库的行和列存储，这相当于是把一个表现力丰富的对象塞到一个非常大的电子表格中：为了适应表结构，你必须设法将这个对象扁平化—​通常一个字段对应一列—​而且每次查询时又需要将其重新构造为对象。

Elasticsearch 是 面向文档 的，意味着它存储整个对象或 文档。Elasticsearch 不仅存储文档，而且 索引 每个文档的内容，使之可以被检索。在 Elasticsearch 中，我们对文档进行索引、检索、排序和过滤—​而不是对行列数据。这是一种完全不同的思考数据的方式，也是 Elasticsearch 能支持复杂全文检索的原因。

## json

Elasticsearch 使用 JavaScript Object Notation（或者 [JSON](https://en.wikipedia.org/wiki/JSON)）作为文档的序列化格式。JSON 序列化为大多数编程语言所支持，并且已经成为 NoSQL 领域的标准格式。 它简单、简洁、易于阅读。

下面这个 JSON 文档代表了一个 user 对象：

```json

{ 
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "info": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01"
}

```

虽然原始的 user 对象很复杂，但这个对象的结构和含义在 JSON 版本中都得到了体现和保留。在 Elasticsearch 中将对象转化为 JSON 后构建索引要比在一个扁平的表结构中要简单的多。

> 🐳 **Note:** 几乎所有的语言都有可以将任意的数据结构或对象转化成 JSON 格式的模块，只是细节各不相同。具体请查看 serialization 或者 marshalling 这两个处理 JSON 的模块。官方 [Elasticsearch 客户端](https://www.elastic.co/guide/en/elasticsearch/client/index.html) 自动为您提供 JSON 转化。
