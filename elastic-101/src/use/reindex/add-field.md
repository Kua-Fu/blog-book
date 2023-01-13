# reindex添加新字段

> [Accessing document fields and special variables](https://www.elastic.co/guide/en/elasticsearch/reference/current/modules-scripting-fields.html#modules-scripting-doc-vals)

假设我们想要知道 之前索引中的日志大小，

可以在reindex阶段，使用script，计算每个doc 的_source长度

```
POST _reindex
{
  "source": {
    "index": "yztest-log"
  },
  "dest": {
    "index": "yztest-backup"
  },
  "script": {
    "lang": "painless",
    "source": """ctx._source.len = ctx._source.toString().length()"""
  }
}
```
