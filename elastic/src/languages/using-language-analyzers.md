# 使用语言分析器

Elasticsearch 的内置分析器都是全局可用的，不需要提前配置，它们也可以在字段映射中直接指定在某字段上：

```python

PUT /my_index
{
  "mappings": {
    "blog": {
      "properties": {
        "title": {
          "type":     "string",
          "analyzer": "english" 
        }
      }
    }
  }
}

```
