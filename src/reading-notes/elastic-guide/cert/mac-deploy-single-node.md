# mac 单节点集群搭建

## 一、Elasticsearch集群

### 1.1 解压安装包


### 1.2 运行ES

```
./bin/elasticsearch
```

运行成功

```

✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  lD-pWP8GLoQjWpgMc4k-

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  06b59714596f3d612d80d884790e9ccabd2e602e36d2b8538302c65873b07176

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMCIsImFkciI6WyIxMC4xMDAuNjUuNDg6OTIwMCJdLCJmZ3IiOiIwNmI1OTcxNDU5NmYzZDYxMmQ4MGQ4ODQ3OTBlOWNjYWJkMmU2MDJlMzZkMmI4NTM4MzAyYzY1ODczYjA3MTc2Iiwia2V5IjoiTzN4WnRZWUIyN0tZWEVPMDMxSEQ6MGl2OU5VektRV1MxMnpneHNMc2dCdyJ9

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.
```

### 1.2 运行kibana

```
./bin/kibana
```

```
➜  kibana-8.1.0 ./bin/kibana
[2023-03-06T13:22:37.762+08:00][INFO ][plugins-service] Plugin "metricsEntities" is disabled.
[2023-03-06T13:22:37.841+08:00][INFO ][http.server.Preboot] http server running at http://localhost:5601
[2023-03-06T13:22:37.883+08:00][INFO ][plugins-system.preboot] Setting up [1] plugins: [interactiveSetup]
[2023-03-06T13:22:37.885+08:00][INFO ][preboot] "interactiveSetup" plugin is holding setup: Validating Elasticsearch connection configuration…
[2023-03-06T13:22:37.918+08:00][INFO ][root] Holding setup until preboot stage is completed.


i Kibana has not been configured.

Go to http://localhost:5601/?code=579260 to get started.

```


### 1.3 获取集群信息

```
GET /

{
  "name" : "node-1",
  "cluster_name" : "my-application",
  "cluster_uuid" : "65mxew5NS6GEWPmmeb1VMg",
  "version" : {
    "number" : "8.1.0",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "3700f7679f7d95e36da0b43762189bab189bc53a",
    "build_date" : "2022-03-03T14:20:00.690422633Z",
    "build_snapshot" : false,
    "lucene_version" : "9.0.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}


GET _cluster/health


{
  "cluster_name" : "my-application",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 10,
  "active_shards" : 10,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

```

![kibana](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/kibana.png?raw=true)
