# 文档元数据

一个文档不仅仅包含它的数据 ，也包含 元数据 —— 有关 文档的信息。 三个必须的元数据元素如下：

* _index 文档在哪存放

* _type 文档表示的对象类别, ⚠️ 高版本，没有_type

* _id 文档唯一标识

## 一、_index

一个 索引 应该是因共同的特性被分组到一起的文档集合。 例如，你可能存储所有的产品在索引 products 中，而存储所有销售的交易到索引 sales 中。 虽然也允许存储不相关的数据到一个索引中，但这通常看作是一个反模式的做法。

> 🐙 **Tip**
>
> 实际上，在 Elasticsearch 中，我们的数据是被存储和索引在 分片 中，而一个索引仅仅是逻辑上的命名空间， 这个命名空间由一个或者多个分片组合在一起。 然而，这是一个内部细节，我们的应用程序根本不应该关心分片，对于应用程序而言，只需知道文档位于一个 索引 内。 Elasticsearch 会处理所有的细节。

我们将在 索引管理 介绍如何自行创建和管理索引，但现在我们将让 Elasticsearch 帮我们创建索引。 所有需要我们做的就是选择一个索引名，

这个名字必须

* 小写，

* 不能以下划线开头，

* 不能包含逗号。

我们用 website 作为索引名举例。

```json
PUT YZtest

{
  "error": {
    "root_cause": [
      {
        "type": "invalid_index_name_exception",
        "reason": "Invalid index name [YZtest], must be lowercase",
        "index_uuid": "_na_",
        "index": "YZtest"
      }
    ],
    "type": "invalid_index_name_exception",
    "reason": "Invalid index name [YZtest], must be lowercase",
    "index_uuid": "_na_",
    "index": "YZtest"
  },
  "status": 400
}

```

```json

PUT _yztest


{
  "error": {
    "root_cause": [
      {
        "type": "invalid_index_name_exception",
        "reason": "Invalid index name [_yztest], must not start with '_', '-', or '+'",
        "index_uuid": "_na_",
        "index": "_yztest"
      }
    ],
    "type": "invalid_index_name_exception",
    "reason": "Invalid index name [_yztest], must not start with '_', '-', or '+'",
    "index_uuid": "_na_",
    "index": "_yztest"
  },
  "status": 400
}

```

```json

PUT yz,test

{
  "error": {
    "root_cause": [
      {
        "type": "invalid_index_name_exception",
        "reason": """Invalid index name [yz,test], must not contain the following characters ['\','/','*','?','"','<','>','|',' ',',']""",
        "index_uuid": "_na_",
        "index": "yz,test"
      }
    ],
    "type": "invalid_index_name_exception",
    "reason": """Invalid index name [yz,test], must not contain the following characters ['\','/','*','?','"','<','>','|',' ',',']""",
    "index_uuid": "_na_",
    "index": "yz,test"
  },
  "status": 400
}

```

## 二、_type 

⚠️ 高版本，没有_type概念

数据可能在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用的。 例如，所有的产品都放在一个索引中，但是你有许多不同的产品类别，比如 "electronics" 、 "kitchen" 和 "lawn-care"。

这些文档共享一种相同的（或非常相似）的模式：他们有一个标题、描述、产品代码和价格。他们只是正好属于“产品”下的一些子类。

Elasticsearch 公开了一个称为 types （类型）的特性，它允许您在索引中对数据进行逻辑分区。不同 types 的文档可能有不同的字段，但最好能够非常相似。 我们将在 类型和映射 中更多的讨论关于 types 的一些应用和限制。

一个 _type 命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符. 我们使用 blog 作为类型名举例。

## 三、_id

ID 是一个字符串，当它和 _index 以及 _type 组合就可以唯一确定 Elasticsearch 中的一个文档。 当你创建一个新的文档，要么提供自己的 _id ，要么让 Elasticsearch 帮你生成。

## 四、其他元数据

还有一些其他的元数据元素，他们在 类型和映射 进行了介绍。通过前面已经列出的元数据元素， 我们已经能存储文档到 Elasticsearch 中并通过 ID 检索它—​换句话说，使用 Elasticsearch 作为文档的存储介质。
