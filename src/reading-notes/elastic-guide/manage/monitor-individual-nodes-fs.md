# 文件系统和网络部分

继续向下阅读 node-stats API，你会看到一串和你的文件系统相关的统计值：

```json

"fs": {
        "timestamp": 1657854496146,
        "total": {
          "total_in_bytes": 193273528320,
          "free_in_bytes": 189751767040,
          "available_in_bytes": 189751767040
        },
        "data": [
          {
            "path": "/app/data",
            "mount": "/app (/dev/mapper/lxc-data)",
            "type": "xfs",
            "total_in_bytes": 193273528320,
            "free_in_bytes": 189751767040,
            "available_in_bytes": 189751767040
          }
        ],
        "io_stats": {
          "devices": [
            {
              "device_name": "dm-1",
              "operations": 637486387,
              "read_operations": 139559703,
              "write_operations": 497926684,
              "read_kilobytes": 4781145648,
              "write_kilobytes": 4130851568,
              "io_time_in_millis": 184204436
            }
          ],
          "total": {
            "operations": 637486387,
            "read_operations": 139559703,
            "write_operations": 497926684,
            "read_kilobytes": 4781145648,
            "write_kilobytes": 4130851568,
            "io_time_in_millis": 184204436
          }
        }
      }

```

* 可用空间，
	
* 数据目录路径，
	
* 磁盘 I/O 统计值，等等。

如果你没有监控磁盘可用空间的话，可以从这里获取这些统计值。磁盘 I/O 统计值也很方便，不过通常那些更专门的命令行工具（比如 iostat ）会更有用些。

显然，Elasticsearch 在磁盘空间满的时候很难运行——所以请确保不会这样。

还有两个跟网络统计值相关的部分：

```json

      "transport": {
        "server_open": 24,
        "total_outbound_connections": 8,
        "rx_count": 20373580,
        "rx_size_in_bytes": 67490236995,
        "tx_count": 20373578,
        "tx_size_in_bytes": 42900769252,
		
	  "http": {
        "current_open": 377,
        "total_opened": 13176,
```

1. transport 显示和 传输地址 相关的一些基础统计值。包括节点间的通信（通常是 9300 端口）以及任意传输客户端或者节点客户端的连接。如果看到这里有很多连接数不要担心；Elasticsearch 在节点之间维护了大量的连接。

1. http 显示 HTTP 端口（通常是 9200）的统计值。如果你看到 total_opened 数很大而且还在一直上涨，这是一个明确信号，说明你的 HTTP 客户端里有没启用 keep-alive 长连接的。持续的 keep-alive 长连接对性能很重要，因为连接、断开套接字是很昂贵的（而且浪费文件描述符）。请确认你的客户端都配置正确。


