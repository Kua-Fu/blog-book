# 索引员工文档

第一个业务需求是存储员工数据。 这将会以 员工文档 的形式存储：一个文档代表一个员工。存储数据到 Elasticsearch 的行为叫做 索引 ，但在索引一个文档之前，需要确定将文档存储在哪里。

一个 Elasticsearch 集群可以 包含多个 索引 ，相应的每个索引可以包含多个 类型 。 这些不同的类型存储着多个 文档 ，每个文档又有 多个 属性 。

⚠️ 高版本(`v8.0`) 已经没有 type 概念

>  **Index Versus Index Versus Index** 
>
> 你也许已经注意到 索引 这个词在 Elasticsearch 语境中有多种含义， 这里有必要做一些说明：
>
> (1) 索引（名词）
>
> 如前所述，一个 索引 类似于传统关系数据库中的一个 数据库 ，是一个存储关系型文档的地方。 索引 (index) 的复数词为 indices 或 indexes 。
>
> (2) 索引（动词）
>
> 索引一个文档 就是存储一个文档到一个 索引 （名词）中以便被检索和查询。这非常类似于 SQL 语句中的 INSERT 关键词，除了文档已存在时，新文档会替换旧文档情况之外。
>
> (3) 倒排索引
>
> 关系型数据库通过增加一个 索引 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 倒排索引 的结构来达到相同的目的。
> 
> 默认的，一个文档中的每一个属性都是 被索引 的（有一个倒排索引）和可搜索的。一个没有倒排索引的属性是不能被搜索到的。我们将在 [倒排索引](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html) 讨论倒排索引的更多细节。


对于员工目录，我们将做如下操作：

* 每个员工索引一个文档，文档包含该员工的所有信息。

* 每个文档都将是 employee 类型 。

* 该类型位于 索引 megacorp 内。

* 该索引保存在我们的 Elasticsearch 集群中。

实践中这非常简单（尽管看起来有很多步骤），我们可以通过一条命令完成所有这些动作：

```json

POST /megacorp/_doc/1?pretty
{
  "type": "employee",
  "first_name": "John",
  "last_name": "Smith",
  "age": 25,
  "about": "I love to go rock climbing",
  "interests": [
    "sports",
    "music"
  ]
}

```

注意，路径 /megacorp/_doc/1 包含了三部分的信息：

* megacorp 索引名称

* _doc 类型名称

* 1 特定雇员的ID

请求体 —— JSON 文档 —— 包含了这位员工的所有详细信息，他的名字叫 John Smith ，今年 25 岁，喜欢攀岩。

很简单！无需进行执行管理任务，如创建一个索引或指定每个属性的数据类型之类的，可以直接只索引一个文档。Elasticsearch 默认地完成其他一切，因此所有必需的管理任务都在后台使用默认设置完成。

进行下一步前，让我们增加更多的员工信息到目录中：

```json

POST /megacorp/_doc/2?pretty
{
  "type": "employee",
  "first_name": "Jane",
  "last_name": "Smith",
  "age": 32,
  "about": "I like to collect rock albums",
  "interests": [
    "music"
  ]
}


POST /megacorp/_doc/3?pretty
{
  "type": "employee",
  "first_name": "Douglas",
  "last_name": "Fir",
  "age": 35,
  "about": "I like to build cabinets",
  "interests": [
    "forestry"
  ]
}

```





