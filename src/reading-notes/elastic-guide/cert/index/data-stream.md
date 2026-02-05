# 数据流

## 一、参考

>[Elasticsearch data stream 实战](https://www.yuque.com/deep_elasticsearch/tzcm9n/gnvb9l)

> [data stream](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/set-up-a-data-stream.html)


## 二、创建数据流

### 2.1 创建一个policy

```

PUT _ilm/policy/yz-data-stream-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "set_priority": {
            "priority": 100
          },
          "rollover": {
            "max_docs": 1000,
            "max_size": "5gb"
          }
        }
      },
      "warm": {
        "min_age": "0ms",
        "actions": {
          "set_priority": {
            "priority": 50
          },
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "cold": {
        "min_age": "3d",
        "actions": {
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "7d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}


```


### 2.2 创建模版

```

PUT _component_template/yz-data-stream-mappings
{
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date",
          "format": "date_optional_time||epoch_millis"
        },
        "message": {
          "type": "text"
        }
      }
    }
  }
}


PUT _component_template/yz-data-stream-settings
{
  "template": {
    "settings": {
      "index.lifecycle.name": "yz-data-stream-policy"
    }
  }
}


PUT _index_template/yz-data-stream-template
{
  "index_patterns": ["yz-data-stream*"],
  "data_stream": {},
  "composed_of": ["yz-data-stream-mappings", "yz-data-stream-settings"],
  "priority": 100
}



```


### 2.3 创建索引

```

PUT yz-data-stream/_bulk
{"create": {}}
{"@timestamp": "2099-05-06T16:21:15.000Z", "message": "192.0.2.42 - - [06/May/2099:16:21:15 +0000] \"GET /images/bg.jpg HTTP/1.0\" 200 24736"}

{
  "took" : 5926,
  "errors" : false,
  "items" : [
    {
      "create" : {
        "_index" : ".ds-yz-data-stream-2023.03.08-000001",
        "_id" : "x7HCv4YBU9rsD44rrpfD",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 0,
        "_primary_term" : 1,
        "status" : 201
      }
    }
  ]
}

```
