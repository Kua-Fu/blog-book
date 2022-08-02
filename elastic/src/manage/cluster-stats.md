# 集群统计

集群统计 API 提供了和 节点统计 相似的输出。 但有一个重要的区别：节点统计显示的是每个节点上的统计值，而 集群统计 展示的是对于单个指标，所有节点的总和值。

这里面提供一些很值得一看的统计值。比如说你可以看到，整个集群用了 50% 的堆内存，或者说过滤器缓存的驱逐情况不严重。这个接口主要用途是提供一个比 集群健康 更详细、但又没有 节点统计 那么详细的快速概览。对于非常大的集群来说也很有用，因为那时候 节点统计 的输出已经非常难于阅读了。

这个 API 可以像下面这样调用：

```json
GET _cluster/stats

{
  "_nodes": {
    "total": 3,
    "successful": 3,
    "failed": 0
  },
  "cluster_name": "a2ff16d9aa2645dc87ab1714e6e16a84",
  "cluster_uuid": "ZjfLI0y8QkyT28Q0cVcI2w",
  "timestamp": 1657862967234,
  "status": "yellow",
  "indices": {
    "count": 95,
    "shards": {
      "total": 194,
      "primaries": 97,
      "replication": 1,
      "index": {
        "shards": {
          "min": 2,
          "max": 6,
          "avg": 2.042105263157895
        },
        "primaries": {
          "min": 1,
          "max": 3,
          "avg": 1.0210526315789474
        },
        "replication": {
          "min": 1,
          "max": 1,
          "avg": 1
        }
      }
    },
    "docs": {
      "count": 8179616,
      "deleted": 3604
    },
    "store": {
      "size_in_bytes": 7312447557,
      "total_data_set_size_in_bytes": 7312447557,
      "reserved_in_bytes": 0
    },
    "fielddata": {
      "memory_size_in_bytes": 928,
      "evictions": 0
    },
    "query_cache": {
      "memory_size_in_bytes": 3102124,
      "total_count": 26571348,
      "hit_count": 2356601,
      "miss_count": 24214747,
      "cache_size": 418,
      "cache_count": 27062,
      "evictions": 26644
    },
    "completion": {
      "size_in_bytes": 0
    },
    "segments": {
      "count": 434,
      "memory_in_bytes": 0,
      "terms_memory_in_bytes": 0,
      "stored_fields_memory_in_bytes": 0,
      "term_vectors_memory_in_bytes": 0,
      "norms_memory_in_bytes": 0,
      "points_memory_in_bytes": 0,
      "doc_values_memory_in_bytes": 0,
      "index_writer_memory_in_bytes": 10516644,
      "version_map_memory_in_bytes": 1212,
      "fixed_bit_set_memory_in_bytes": 1592256,
      "max_unsafe_auto_id_timestamp": 1657774689022,
      "file_sizes": {}
    },
    "mappings": {
      "field_types": [
        {
          "name": "alias",
          "count": 1027,
          "index_count": 8,
          "script_count": 0
        },
        {
          "name": "boolean",
          "count": 207,
          "index_count": 28,
          "script_count": 0
        },
        {
          "name": "byte",
          "count": 1,
          "index_count": 1,
          "script_count": 0
        },
        {
          "name": "constant_keyword",
          "count": 15,
          "index_count": 5,
          "script_count": 0
        },
        {
          "name": "date",
          "count": 373,
          "index_count": 64,
          "script_count": 0
        },
        {
          "name": "date_range",
          "count": 1,
          "index_count": 1,
          "script_count": 0
        },
        {
          "name": "double",
          "count": 343,
          "index_count": 6,
          "script_count": 0
        },
        {
          "name": "flattened",
          "count": 12,
          "index_count": 1,
          "script_count": 0
        },
        {
          "name": "float",
          "count": 314,
          "index_count": 15,
          "script_count": 0
        },
        {
          "name": "geo_point",
          "count": 38,
          "index_count": 8,
          "script_count": 0
        },
        {
          "name": "half_float",
          "count": 24,
          "index_count": 6,
          "script_count": 0
        },
        {
          "name": "integer",
          "count": 13,
          "index_count": 9,
          "script_count": 0
        },
        {
          "name": "ip",
          "count": 78,
          "index_count": 7,
          "script_count": 0
        },
        {
          "name": "ip_range",
          "count": 1,
          "index_count": 1,
          "script_count": 0
        },
        {
          "name": "keyword",
          "count": 4783,
          "index_count": 77,
          "script_count": 0
        },
        {
          "name": "long",
          "count": 4229,
          "index_count": 34,
          "script_count": 0
        },
        {
          "name": "match_only_text",
          "count": 63,
          "index_count": 1,
          "script_count": 0
        },
        {
          "name": "nested",
          "count": 20,
          "index_count": 4,
          "script_count": 0
        },
        {
          "name": "object",
          "count": 5448,
          "index_count": 43,
          "script_count": 0
        },
        {
          "name": "scaled_float",
          "count": 154,
          "index_count": 3,
          "script_count": 0
        },
        {
          "name": "short",
          "count": 14,
          "index_count": 7,
          "script_count": 0
        },
        {
          "name": "text",
          "count": 239,
          "index_count": 19,
          "script_count": 0
        },
        {
          "name": "version",
          "count": 1,
          "index_count": 1,
          "script_count": 0
        },
        {
          "name": "wildcard",
          "count": 17,
          "index_count": 1,
          "script_count": 0
        }
      ],
      "runtime_field_types": []
    },
    "analysis": {
      "char_filter_types": [],
      "tokenizer_types": [],
      "filter_types": [],
      "analyzer_types": [],
      "built_in_char_filters": [],
      "built_in_tokenizers": [],
      "built_in_filters": [],
      "built_in_analyzers": [
        {
          "name": "english",
          "count": 1,
          "index_count": 1
        }
      ]
    },
    "versions": [
      {
        "version": "8.3.1",
        "index_count": 95,
        "primary_shard_count": 97,
        "total_primary_bytes": 3385710034
      }
    ]
  },
  "nodes": {
    "count": {
      "total": 3,
      "coordinating_only": 0,
      "data": 0,
      "data_cold": 0,
      "data_content": 2,
      "data_frozen": 0,
      "data_hot": 2,
      "data_warm": 0,
      "ingest": 2,
      "master": 3,
      "ml": 0,
      "remote_cluster_client": 2,
      "transform": 2,
      "voting_only": 1
    },
    "versions": [
      "8.3.1"
    ],
    "os": {
      "available_processors": 6,
      "allocated_processors": 6,
      "names": [
        {
          "name": "Linux",
          "count": 3
        }
      ],
      "pretty_names": [
        {
          "pretty_name": "Ubuntu 20.04.4 LTS",
          "count": 3
        }
      ],
      "architectures": [
        {
          "arch": "amd64",
          "count": 3
        }
      ],
      "mem": {
        "total_in_bytes": 9663676416,
        "adjusted_total_in_bytes": 8531214336,
        "free_in_bytes": 924491776,
        "used_in_bytes": 8739184640,
        "free_percent": 10,
        "used_percent": 90
      }
    },
    "process": {
      "cpu": {
        "percent": 6
      },
      "open_file_descriptors": {
        "min": 852,
        "max": 1372,
        "avg": 1154
      }
    },
    "jvm": {
      "max_uptime_in_millis": 867151943,
      "versions": [
        {
          "version": "18.0.1.1",
          "vm_name": "OpenJDK 64-Bit Server VM",
          "vm_version": "18.0.1.1+2-6",
          "vm_vendor": "Oracle Corporation",
          "bundled_jdk": true,
          "using_bundled_jdk": true,
          "count": 3
        }
      ],
      "mem": {
        "heap_used_in_bytes": 2435488768,
        "heap_max_in_bytes": 4198498304
      },
      "threads": 176
    },
    "fs": {
      "total_in_bytes": 434865438720,
      "free_in_bytes": 425859817472,
      "available_in_bytes": 425859817472
    },
    "plugins": [],
    "network_types": {
      "transport_types": {
        "security4": 3
      },
      "http_types": {
        "security4": 3
      }
    },
    "discovery_types": {
      "multi-node": 3
    },
    "packaging_types": [
      {
        "flavor": "default",
        "type": "docker",
        "count": 3
      }
    ],
    "ingest": {
      "number_of_pipelines": 35,
      "processor_stats": {
        "append": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        },
        "conditional": {
          "count": 13757,
          "failed": 2,
          "current": 0,
          "time_in_millis": 2039
        },
        "convert": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        },
        "date": {
          "count": 1361166,
          "failed": 2,
          "current": 0,
          "time_in_millis": 13095
        },
        "dot_expander": {
          "count": 44712,
          "failed": 0,
          "current": 0,
          "time_in_millis": 94
        },
        "geoip": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        },
        "grok": {
          "count": 1366785,
          "failed": 11,
          "current": 0,
          "time_in_millis": 45211
        },
        "gsub": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        },
        "json": {
          "count": 5600,
          "failed": 11,
          "current": 0,
          "time_in_millis": 134
        },
        "pipeline": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        },
        "remove": {
          "count": 1391685,
          "failed": 0,
          "current": 0,
          "time_in_millis": 522
        },
        "rename": {
          "count": 61004,
          "failed": 0,
          "current": 0,
          "time_in_millis": 101
        },
        "script": {
          "count": 13745,
          "failed": 11,
          "current": 0,
          "time_in_millis": 136
        },
        "set": {
          "count": 6822207,
          "failed": 0,
          "current": 0,
          "time_in_millis": 21445
        },
        "set_security_user": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        },
        "split": {
          "count": 1355575,
          "failed": 0,
          "current": 0,
          "time_in_millis": 1466
        },
        "trim": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        },
        "user_agent": {
          "count": 0,
          "failed": 0,
          "current": 0,
          "time_in_millis": 0
        }
      }
    },
    "indexing_pressure": {
      "memory": {
        "current": {
          "combined_coordinating_and_primary_in_bytes": 0,
          "coordinating_in_bytes": 0,
          "primary_in_bytes": 0,
          "replica_in_bytes": 0,
          "all_in_bytes": 0
        },
        "total": {
          "combined_coordinating_and_primary_in_bytes": 0,
          "coordinating_in_bytes": 0,
          "primary_in_bytes": 0,
          "replica_in_bytes": 0,
          "all_in_bytes": 0,
          "coordinating_rejections": 0,
          "primary_rejections": 0,
          "replica_rejections": 0
        },
        "limit_in_bytes": 0
      }
    }
  }
}
```
