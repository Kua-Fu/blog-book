# 文档的部分更新

在 更新整个文档 , 我们已经介绍过 更新一个文档的方法是检索并修改它，然后重新索引整个文档，这的确如此。然而，使用 update API 我们还可以部分更新文档，例如在某个请求时对计数器进行累加。

我们也介绍过文档是不可变的：他们不能被修改，只能被替换。 update API 必须遵循同样的规则。 从外部来看，我们在一个文档的某个位置进行部分更新。然而在内部， update API 简单使用与之前描述相同的 检索-修改-重建索引 的处理过程。 区别在于这个过程发生在分片内部，这样就避免了多次请求的网络开销。通过减少检索和重建索引步骤之间的时间，我们也减少了其他进程的变更带来冲突的可能性。

update 请求最简单的一种形式是接收文档的一部分作为 doc 的参数， 它只是与现有的文档进行合并。对象被合并到一起，覆盖现有的字段，增加新的字段。 例如，我们增加字段 tags 和 views 到我们的博客文章，如下所示：

```json

POST /website/_update/1
{
  "doc": {
    "tags": [
      "testing"
    ],
    "views": 0
  }
}

{
  "_index": "website",
  "_id": "1",
  "_version": 2,
  "result": "noop",
  "_shards": {
    "total": 0,
    "successful": 0,
    "failed": 0
  },
  "_seq_no": 7,
  "_primary_term": 1
}

```

如果请求成功，我们看到类似于 index 请求的响应。

检索文档显示了更新后的 _source 字段：

```json

GET website/_doc/1

{
  "_index": "website",
  "_id": "1",
  "_version": 2,
  "_seq_no": 7,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "type": "blog",
    "title": "My first blog entry",
    "text": "Just trying this out...",
    "views": 0, 0️⃣
    "tags": [ 1️⃣
      "testing"
    ]
  }
}

```

0️⃣ 1️⃣ 新的字段已被添加到 _source 中。

## 使用脚本部分更新文档

脚本可以在 update API中用来改变 _source 的字段内容， 它在更新脚本中称为 ctx._source 。 例如，我们可以使用脚本来增加博客文章中 views 的数量：

```json

POST /website/_update/1
{
  "script": "ctx._source.views+=1"
}

{
  "_index": "website",
  "_id": "1",
  "_version": 3,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 8,
  "_primary_term": 1
}

```

> **用 Groovy 脚本编程**
>
> 对于那些 API 不能满足需求的情况，Elasticsearch 允许你使用脚本编写自定义的逻辑。 许多API都支持脚本的使用，包括搜索、排序、聚合和文档更新。 脚本可以作为请求的一部分被传递，从特殊的 .scripts 索引中检索，或者从磁盘加载脚本。
>
> 默认的脚本语言 是 Groovy，一种快速表达的脚本语言，在语法上与 JavaScript 类似。 它在 Elasticsearch V1.3.0 版本首次引入并运行在 沙盒 中，然而 Groovy 脚本引擎存在漏洞， 允许攻击者通过构建 Groovy 脚本，在 Elasticsearch Java VM 运行时脱离沙盒并执行 shell 命令。
>
> 因此，在版本 v1.3.8 、 1.4.3 和 V1.5.0 及更高的版本中，它已经被默认禁用。 此外，您可以通过设置集群中的所有节点的 config/elasticsearch.yml 文件来禁用动态 Groovy 脚本：
> `script.groovy.sandbox.enabled: false`
>
> 这将关闭 Groovy 沙盒，从而防止动态 Groovy 脚本作为请求的一部分被接受， 或者从特殊的 .scripts 索引中被检索。当然，你仍然可以使用存储在每个节点的 config/scripts/ 目录下的 Groovy 脚本。
>
> 如果你的架构和安全性不需要担心漏洞攻击，例如你的 Elasticsearch 终端仅暴露和提供给可信赖的应用， 当它是你的应用需要的特性时，你可以选择重新启用动态脚本。
> 
> 你可以在 [scripting reference documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting.html) 获取更多关于脚本的资料。

我们也可以通过使用脚本给 tags 数组添加一个新的标签。在这个例子中，我们指定新的标签作为参数，而不是硬编码到脚本内部。 这使得 Elasticsearch 可以重用这个脚本，而不是每次我们想添加标签时都要对新脚本重新编译：

```json

POST /website/_update/1
{
  "script": {
    "source": "ctx._source.tags.add(params.new_tag)",
    "lang": "painless",
    "params": {
      "new_tag": "search"
    }
  }
}

{
  "_index": "website",
  "_id": "1",
  "_version": 5,
  "result": "updated",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 10,
  "_primary_term": 1
}

```

获取文档并显示最后两次请求的效果：

```json

GET website/_doc/1

{
  "_index": "website",
  "_id": "1",
  "_version": 5,
  "_seq_no": 10,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "type": "blog",
    "title": "My first blog entry",
    "text": "Just trying this out...",
    "views": 1, 0️⃣
    "tags": [
      "testing",
      null,
      "search" 1️⃣
    ]
  }
}

```

0️⃣ views 字段已递增

1️⃣ search 标签已追加到 tags 数组中

我们甚至可以选择通过设置 ctx.op 为 delete 来删除基于其内容的文档：

```json

POST /website/_update/1/
{
  "script": {
    "source": "ctx.op = ctx._source.views == params.count ? 'delete' : 'none'",
    "params": {
      "count": 1
    }
  }
}

{
  "_index": "website",
  "_id": "1",
  "_version": 6,
  "result": "deleted",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 11,
  "_primary_term": 1
}

```

## 更新的文档可能尚不存在

假设我们需要在 Elasticsearch 中存储一个页面访问量计数器。 每当有用户浏览网页，我们对该页面的计数器进行累加。但是，如果它是一个新网页，我们不能确定计数器已经存在。 如果我们尝试更新一个不存在的文档，那么更新操作将会失败。

在这样的情况下，我们可以使用 upsert 参数，指定如果文档不存在就应该先创建它：

我们第一次运行这个请求时， upsert 值作为新文档被索引，初始化 views 字段为 1 。 在后续的运行中，由于文档已经存在， script 更新操作将替代 upsert 进行应用，对 views 计数器进行累加。

```json

POST /website/_update/11/
{
  "script": {
    "source": "ctx._source.views+=1"
  },
  "upsert": {
    "views": 1
  }
}


{
  "_index": "website",
  "_id": "11",
  "_version": 1,
  "result": "created",
  "_shards": {
    "total": 2,
    "successful": 2,
    "failed": 0
  },
  "_seq_no": 12,
  "_primary_term": 1
}

```

```json

GET website/_doc/11

{
  "_index": "website",
  "_id": "11",
  "_version": 1,
  "_seq_no": 12,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "views": 1
  }
}

```

## 更新和冲突

在本节的介绍中，我们说明 检索 和 重建索引 步骤的间隔越小，变更冲突的机会越小。 但是它并不能完全消除冲突的可能性。 还是有可能在 update 设法重新索引之前，来自另一进程的请求修改了文档。

为了避免数据丢失， update API 在 检索 步骤时检索得到文档当前的 _version 号，并传递版本号到 重建索引 步骤的 index 请求。 如果另一个进程修改了处于检索和重新索引步骤之间的文档，那么 _version 号将不匹配，更新请求将会失败。

对于部分更新的很多使用场景，文档已经被改变也没有关系。 例如，如果两个进程都对页面访问量计数器进行递增操作，它们发生的先后顺序其实不太重要； 如果冲突发生了，我们唯一需要做的就是尝试再次更新。

这可以通过设置参数 retry_on_conflict 来自动完成， 这个参数规定了失败之前 update 应该重试的次数，它的默认值为 0 。

```json

POST /website/_update/1/?retry_on_conflict=5  0️⃣
{
  "script": "ctx._source.views+=1",
  "upsert": {
    "views": 0
  }
}

```

0️⃣ 失败之前重试该更新5次

在增量操作无关顺序的场景，例如递增计数器等这个方法十分有效，但是在其他情况下变更的顺序 是 非常重要的。 类似 index API ， update API 默认采用 最终写入生效 的方案，但它也接受一个 version 参数来允许你使用 optimistic concurrency control 指定想要更新文档的版本。
