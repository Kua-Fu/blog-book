# 可搜索快照


## 一、参考

> [04.Elasticsearch 可搜索快照深入详解](https://www.yuque.com/deep_elasticsearch/tzcm9n/grbfm2)


## 二、可搜索快照

### 2.1 创建共享存储

每个节点都修改 elasticsearch.yml ，添加

```
path.repo: "/Users/yz/work/env/es/share"
```

### 2.2 依次重启节点

### 2.3 创建快照存储库

```
PUT /_snapshot/yz_repository
{
  "type": "fs",
  "settings": {
    "location": "/Users/yz/work/env/es/share"
  }
}
```

可以通过接口查看仓库信息

```

POST _snapshot/yz_repository/_verify

{
  "nodes" : {
    "gaWrkXVFSXen6BbmqxhFow" : {
      "name" : "node2"
    },
    "PM08omCVR465UByrpGeVKw" : {
      "name" : "node1"
    },
    "0QbfdkZET8-s5OZ0bZ72eA" : {
      "name" : "node3"
    }
  }
}

```

### 2.4 创建快照

```
PUT /_snapshot/yz_repository/yz_search_snapshot?wait_for_completion=true
{
  "indices": "yztest",
  "ignore_unavailable": true,
  "include_global_state": false
}

{
  "snapshot" : {
    "snapshot" : "yz_search_snapshot",
    "uuid" : "9Ul_b_aRRnqgRiakj0CXUg",
    "repository" : "yz_repository",
    "version_id" : 8010099,
    "version" : "8.1.0",
    "indices" : [
      "yztest-000002",
      "yztest-000001"
    ],
    "data_streams" : [ ],
    "include_global_state" : false,
    "state" : "SUCCESS",
    "start_time" : "2023-03-06T10:58:37.776Z",
    "start_time_in_millis" : 1678100317776,
    "end_time" : "2023-03-06T10:58:39.812Z",
    "end_time_in_millis" : 1678100319812,
    "duration_in_millis" : 2036,
    "failures" : [ ],
    "shards" : {
      "total" : 2,
      "failed" : 0,
      "successful" : 2
    },
    "feature_states" : [ ]
  }
}

```

### 2.5 将快照挂载为可搜索快照

```
POST /_snapshot/yz_repository/yz_search_snapshot/_mount?wait_for_completion=true
{
  "index": "yztest-000001", 
  "renamed_index": "yz-searchable", 
  "index_settings": { 
    "index.number_of_replicas": 0
  },
  "ignore_index_settings": [ "index.refresh_interval" ] 
}

{
  "snapshot" : {
    "snapshot" : "yz_search_snapshot",
    "indices" : [
      "yz-searchable"
    ],
    "shards" : {
      "total" : 1,
      "failed" : 0,
      "successful" : 1
    }
  }
}

```

注意⚠️ 可能会出现licenese权限不够问题，需要开启30天试用

### 2.6 搜索快照

```

GET yz-searchable/_search
```

