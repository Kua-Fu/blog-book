# 只备份集群中的策略、模版等信息

## 参考

>[用户管理](https://www.elastic.co/guide/en/elasticsearch/reference/current/security-api-put-user.html)

>[集群配置](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/cluster-update-settings.html)

>[策略管理](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/data-management.html)

>[模版管理](https://www.elastic.co/guide/en/elasticsearch/reference/8.3/index-templates.html)

## 一、集群配置

### 1.1 从现有集群获取集群配置

```

GET _cluster/settings?flat_settings

curl -u elastic:xxx -XGET "http://127.0.0.1:9200/_cluster/settings?flat_settings"

```

返回值如下:

```json

{
  "persistent" : {
    "action.auto_create_index" : "+.*,-*",
    "action.destructive_requires_name" : "false",
    "cluster.max_shards_per_node" : "5000",
    "xpack.monitoring.collection.enabled" : "true"
  },
  "transient" : { }
}

```

### 1.2 将集群配置导入到新集群

```

PUT _cluster/settings
{
  "persistent" : {
    "action.auto_create_index" : "+.*,-*",
    "action.destructive_requires_name" : "false",
    "cluster.max_shards_per_node" : "5000",
    "xpack.monitoring.collection.enabled" : "true"
  },
  "transient" : { }
}

curl -u elastic:xxx -XPUT "http://127.0.0.l:9200/_cluster/settings" -H 'Content-Type: application/json' -d'{  "persistent" : {    "action.auto_create_index" : "+.*,-*",    "action.destructive_requires_name" : "false",    "cluster.max_shards_per_node" : "5000",    "xpack.monitoring.collection.enabled" : "true"  },  "transient" : { }}'

```

## 二、索引策略

### 2.1 从现有集群获取策略信息

```

GET _ilm/policy

curl -u elastic:xxx -XGET "http://127.0.0.1:9200/_ilm/policy"

```

返回值如下:

```json

{
  ".items-default" : {
    "version" : 1,
    "modified_date" : "2021-12-30T06:02:12.763Z",
    "policy" : {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "50gb"
            }
          }
        }
      }
    }
  }
  ...
}

```

### 2.2 将策略信息导入到新集群

⚠️ 一次只能导入一个策略, PUT body 不需要 version/modified_date 信息

```

PUT _ilm/policy/.items-default
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_size": "50gb"
          }
        }
      }
    }
  }
}

curl -u elastic:xxx -XPUT "http://127.0.0.1:9200/_ilm/policy/.items-default" -H 'Content-Type: application/json' -d'{  "policy": {    "phases": {      "hot": {        "min_age": "0ms",        "actions": {          "rollover": {            "max_size": "50gb"          }        }      }    }  }}'

```

## 三、索引模版

### 3.1 从现有集群获取索引模版

```

GET _template/*

curl -u elastic:xxx -XGET "http://127.0.0.1:9200/_template/*"

```

返回结果如下:

```json

{
  "wksp_xxx_template" : {
    "order" : 0,
    "index_patterns" : [
      "wksp_xxx-*"
    ],
    "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "es_rp4",
          "rollover_alias" : "wksp_xxx"
        },
        "number_of_shards" : "1",
        "number_of_replicas" : "1"
      }
    },
    "mappings" : {
      "dynamic_templates" : [
        {
          "strings_as_keywords" : {
            "mapping" : {
              "type" : "keyword"
            },
            "match_mapping_type" : "string"
          }
        }
      ],
      "date_detection" : false,
      "properties" : {
        "message" : {
          "search_analyzer" : "ik_max_word",
          "search_quote_analyzer" : "ik_max_word",
          "analyzer" : "ik_max_word",
          "type" : "text"
        },
        "title" : {
          "search_analyzer" : "ik_max_word",
          "search_quote_analyzer" : "ik_max_word",
          "analyzer" : "ik_max_word",
          "type" : "text"
        }
      }
    },
    "aliases" : { }
  }
  
  ...
}

```

### 3.2 将索引模版导入到新集群

⚠️ 一次只能导入一个模版

```

PUT _template/wksp_xxx_template
{
  "order": 0,
  "index_patterns": [
    "wksp_xxx-*"
  ],
  "settings": {
    "index": {
      "lifecycle": {
        "name": "es_rp4",
        "rollover_alias": "wksp_xxx"
      },
      "number_of_shards": "1",
      "number_of_replicas": "1"
    }
  },
  "mappings": {
    "dynamic_templates": [
      {
        "strings_as_keywords": {
          "mapping": {
            "type": "keyword"
          },
          "match_mapping_type": "string"
        }
      }
    ],
    "date_detection": false,
    "properties": {
      "message": {
        "search_analyzer": "ik_max_word",
        "search_quote_analyzer": "ik_max_word",
        "analyzer": "ik_max_word",
        "type": "text"
      },
      "title": {
        "search_analyzer": "ik_max_word",
        "search_quote_analyzer": "ik_max_word",
        "analyzer": "ik_max_word",
        "type": "text"
      }
    }
  },
  "aliases": {}
}

curl -u elastic:xxx -XPUT "http://127.0.0.1:9200/_template/wksp_xxx_template" -H 'Content-Type: application/json' -d'{  "order": 0,  "index_patterns": [    "wksp_xxx-*"  ],  "settings": {    "index": {      "lifecycle": {        "name": "es_rp4",        "rollover_alias": "wksp_xxx"      },      "number_of_shards": "1",      "number_of_replicas": "1"    }  },  "mappings": {    "dynamic_templates": [      {        "strings_as_keywords": {          "mapping": {            "type": "keyword"          },          "match_mapping_type": "string"        }      }    ],    "date_detection": false,    "properties": {      "message": {        "search_analyzer": "ik_max_word",        "search_quote_analyzer": "ik_max_word",        "analyzer": "ik_max_word",        "type": "text"      },      "title": {        "search_analyzer": "ik_max_word",        "search_quote_analyzer": "ik_max_word",        "analyzer": "ik_max_word",        "type": "text"      }    }  },  "aliases": {}}'

```

## 四、用户

elasticsearch 无法通过接口，获取用户密码，也就不存在用户密码备份

如果需要在新集群添加用户，或者重置（内置用户）密码，可以使用 `POST /_security/user/username` 

```
POST /_security/user/jacknich
{
  "password" : "l0ng-r4nd0m-p@ssw0rd",
  "roles" : [ "admin", "other_role1" ],
  "full_name" : "Jack Nicholson",
  "email" : "jacknich@example.com",
  "metadata" : {
    "intelligence" : 7
  }
}

```
