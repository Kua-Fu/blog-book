# 热温冷架构


## 一、参考

> [ILM索引生命周期管理实战](https://www.yuque.com/deep_elasticsearch/tzcm9n/zqxpwi)

> [干货 | Elasticsearch 8.X 节点角色划分深入详解](https://mp.weixin.qq.com/s/3486iH3VH7TV6lza-a7adQ)

## 二、热温冷存储架构


### 2.1 启动时，指定节点属性

```

./bin/elasticsearch -Ecluster.name=yz-cluster -Enode.name=node1 -Enode.attr.box_type=hot

./bin/elasticsearch -Ecluster.name=yz-cluster -Enode.name=node2 -Enode.attr.box_type=warm

./bin/elasticsearch -Ecluster.name=yz-cluster -Enode.name=node3 -Enode.attr.box_type=cold
```


注意⚠️ 此时 box_type 需要使用 data 表示

可以通过接口 `GET _cat/nodeattrs?v` 接口查看节点的属性


### 2.2 修改集群策略检查周期


```

PUT _cluster/settings
{
  "persistent": {
    "indices.lifecycle.poll_interval": "3s"
  }
}

#! [xpack.monitoring.collection.enabled] setting was deprecated in Elasticsearch and will be removed in a future release.
{
  "acknowledged" : true,
  "persistent" : {
    "indices" : {
      "lifecycle" : {
        "poll_interval" : "3s"
      }
    }
  },
  "transient" : { }
}

```

### 2.3 创建索引策略

| 序号 | 名称 | 描述| |
|---| ---|---|---|
|1| 热阶段 | 滚动条件为 最大文档数 5个/最大保留时间7d/ 最大的索引大小5gb| |
|2| 温阶段 | 合并段为1，设置存储层为 warm | |
|3| 冷阶段| 设置存储层为cold | |
|4| 删除阶段 | 删除数据| |

```
PUT _ilm/policy/yz_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "set_priority": {
            "priority": 100
          },
          "rollover": {
            "max_age": "7d",
            "max_docs": 5,
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
          "forcemerge": {
            "max_num_segments": 1
          },
          "allocate": {
            "require": {
              "data": "warm"
            }
          }
        }
      },
      "cold": {
        "min_age": "3m",
        "actions": {
          "set_priority": {
            "priority": 20
          },
          "allocate": {
            "number_of_replicas": 0,
              "require": {
              "data": "cold"
            }
          }
        }
      },
      "delete": {
        "min_age": "1d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```



### 2.4 创建模版

```

PUT _index_template/yz_template
{
  "index_patterns": ["yztest-*"], 
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "yz_policy", 
      "index.lifecycle.rollover_alias": "yztest",
      "index.routing.allocation.require.data": "hot"
    }
  }
}

```


### 2.5 创建索引

```
PUT yztest-000001
{
  "aliases": {
    "yztest": {
      "is_write_index": true
    }
  }
}
```

### 2.6 写入数据

```

POST _bulk
{ "index" : { "_index" : "yztest", "_id" : "1" } }
{ "field1" : "value1" }
{ "index" : { "_index" : "yztest", "_id" : "2" } }
{ "field1" : "value1" }
{ "index" : { "_index" : "yztest", "_id" : "3" } }
{ "field1" : "value1" }
{ "index" : { "_index" : "yztest", "_id" : "4" } }
{ "field1" : "value1" }
{ "index" : { "_index" : "yztest", "_id" : "5" } }
{ "field1" : "value1" }
```


### 2.7 查看索引生命周期

```
GET yztest/_ilm/explain
```
