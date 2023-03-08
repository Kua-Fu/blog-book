# 跨集群查询


## 一、参考

> [跨集群复制](https://www.yuque.com/deep_elasticsearch/tzcm9n/izvdf1)


## 二、环境搭建

```

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  =vdZ17xI9tw-A9QT6_wy

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  e848ccb198de4206fa611432246f76351c230bb3d132c3db8ecd42502e11876c

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
                                                                                          [9/1998]
ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  =vdZ17xI9tw-A9QT6_wy

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  e848ccb198de4206fa611432246f76351c230bb3d132c3db8ecd42502e11876c

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMCIsImFkciI6WyIxMC4xMDAuNjUuNDg6OTIwMyJdLCJmZ3IiOiJlODQ4Y2NiMTk4ZGU0MjA2ZmE2MTE0MzIyNDZmNzYzNTFjMjMwYmIzZDEzMmMzZGI4ZWNkNDI1MDJlMTE4NzZjIiwia2V5IjoiUTBPbXRvWUIzUmtVdEgxVlRZMXU6XzFSWXdoSjBTbWkyRHBKYk1zZGs3QSJ9

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment
token that you generated.
```


```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  HV3PwqxdvCEgrNO++=FI

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  1e23bcb35f37bcdf6db4da4ed34894e6c9e523e1bc6a73022ef747b71340bb43

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMCIsImFkciI6WyIxMC4xMDAuNjUuNDg6OTIwMCJdLCJmZ3IiOiIxZTIzYmNiMzVmMzdiY2RmNmRiNGRhNGVkMzQ4OTRlNmM5ZTUyM2UxYmM2YTczMDIyZWY3NDdiNzEzNDBiYjQzIiwia2V5IjoicC1hdHVvWUJmdEdwTHZUclZzZDg6bTAyc2kwRWFUQ2l4Y3hSMlZYZE1nUSJ9

ℹ️  HTTP CA certificate SHA-256 fingerprint:                                              [11/1948]
  1e23bcb35f37bcdf6db4da4ed34894e6c9e523e1bc6a73022ef747b71340bb43

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next
 30 minutes):
  eyJ2ZXIiOiI4LjEuMCIsImFkciI6WyIxMC4xMDAuNjUuNDg6OTIwMCJdLCJmZ3IiOiIxZTIzYmNiMzVmMzdiY2RmNmRiNGRh
NGVkMzQ4OTRlNmM5ZTUyM2UxYmM2YTczMDIyZWY3NDdiNzEzNDBiYjQzIiwia2V5IjoicC1hdHVvWUJmdEdwTHZUclZzZDg6bT
Ayc2kwRWFUQ2l4Y3hSMlZYZE1nUSJ9

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment
token that you generated.
━━━━━━━━━━━━━━━━━━━━━━━━━━
```


折腾了一天，安全认证模块还是有问题，暂时测试环境，不使用xpack.security 了


### 2.1 创建了两个集群

```

# 集群1
单节点 127.0.0.1:9210/ 127.0.0.1:9310

# 集群2
单节点 127.0.0.1:9220 / 127.0.0.1:9320
```


## 三、跨集群查询

### 3.1 添加远程集群


![r1](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/remote-query.png?raw=true)

![r2](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/remote_query2.png?raw=true)


### 3.2 查看远端集群

```
GET _remote/info

{
  "remote" : {
    "connected" : true,
    "mode" : "sniff",
    "seeds" : [
      "127.0.0.1:9320"
    ],
    "num_nodes_connected" : 1,
    "max_connections_per_cluster" : 3,
    "initial_connect_timeout" : "30s",
    "skip_unavailable" : false
  }
}

```


### 3.3 远端集群搜索

```
GET remote:yztest/_search

{
  "took" : 57,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "_clusters" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "remote:yztest",
        "_id" : "nZB0v4YBbK1ckvCSmvQo",
        "_score" : 1.0,
        "_source" : {
          "f1" : "11"
        }
      }
    ]
  }
}


```


## 四、跨集群复制

![ccr1](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/ccr1.png?raw=true)

![ccr2](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/ccr2.png?raw=true)

```

GET remote:test-leader/_search

{
  "took" : 624,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "_clusters" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "remote:test-leader",
        "_id" : "npCHv4YBbK1ckvCSgvQj",
        "_score" : 1.0,
        "_source" : {
          "f1" : "leader message"
        }
      }
    ]
  }
}

```

```
GET test-follower/_search

{
  "took" : 2,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 1,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "test-follower",
        "_id" : "npCHv4YBbK1ckvCSgvQj",
        "_score" : 1.0,
        "_source" : {
          "f1" : "leader message"
        }
      }
    ]
  }
}

```
