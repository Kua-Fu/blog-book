# 多节点集群搭建


## 一、参考

> [集群安全配置功能大升级，单机模拟运行 Elasticsearch 8.1.2 三节点集群](https://www.bilibili.com/video/BV1xq4y1e7za/?vd_source=52d4e65861154c1f362e67e27482d7bf)

> [使用 rpm/deb 软件包安装 Elasticsearch 8.1.0](https://elastic.martinliu.cn/chapter3/chapter3-1/4/)




## 二、部署

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  1SM2d6YyVwyEz3QonYTz

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  1faab1a2fbe2141bb6fe4bf637cf01b254e5fa60cdba957950242dba0b909e43

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMCIsImFkciI6WyIxMC4xMDAuNjUuNDg6OTIwMCJdLCJmZ3IiOiIxZmFhYjFhMmZiZTIxNDFiYjZmZTRiZjYzN2NmMDFiMjU0ZTVmYTYwY2RiYTk1Nzk1MDI0MmRiYTBiOTA5ZTQzIiwia2V5IjoiamUtcXRZWUJCRXJkeFVhQ1g2WDQ6NU1GdTFVaVBUTE9VTkhBeEczcklRZyJ9

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.
```


![cluster](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/cluster.png?raw=true)
