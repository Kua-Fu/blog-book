# 单节点集群搭建

## 一、环境

腾讯云服务器 

Centos7.6 

4 core/ 4G

![tx](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/tx.png?raw=true)

安装包下载

```

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.1.0-linux-x86_64.tar.gz

wget https://artifacts.elastic.co/downloads/kibana/kibana-8.1.0-linux-x86_64.tar.gz

```

## 二、单节点集群部署

### 2.1 解压

```
tar -zxvf elasticsearch-8.1.0-linux-x86_64.tar.gz
tar -zxvf kibana-8.1.0-linux-x86_64.tar.gz
```

### 2.2 运行

```
cd elasticsearch-8.1.0
./bin/elasticsearch

```

(1) 报错如下

```
uncaught exception in thread [main]
java.lang.RuntimeException: can not run elasticsearch as root
	at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:103)
	at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:183)
	at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:358)
	at org.elasticsearch.bootstrap.Elasticsearch.init(Elasticsearch.java:166)
	at org.elasticsearch.bootstrap.Elasticsearch.execute(Elasticsearch.java:157)
	at org.elasticsearch.common.cli.EnvironmentAwareCommand.execute(EnvironmentAwareCommand.java:81)
	at org.elasticsearch.cli.Command.mainWithoutErrorHandling(Command.java:112)
	at org.elasticsearch.cli.Command.main(Command.java:77)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:122)
	at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:80)
For complete error details, refer to the log at /root/work/es/elasticsearch-8.1.0/logs/elasticsearch.log
2023-03-04 06:30:35,379684 UTC [9101] INFO  Main.cc@112 Parent process died - ML controller exiting
```


需要创建一个elasticsearch 用户

```
adduser elasticsearch
passwd elasticsearch

```

然后，修改解压目录权限

```
chown -R elasticsearch:elasticsearch elasticsearch-8.1.0
```

(2) 报错如下

```
[elasticsearch@VM-12-11-centos elasticsearch-8.1.0]$ ./bin/elasticsearch
could not find java in bundled JDK at /root/work/es/elasticsearch-8.1.0/jdk/bin/java
[elasticsearch@VM-12-11-centos elasticsearch-8.1.0]$ pwd
/root/work/es/elasticsearch-8.1.0
```

权限问题，需要把解压目录移动到 elasticsearch home目录

```
mv es /home/elasticsearch/
```

(3) 正常运行

注意⚠️ : 需要保留账号信息

![security](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/security.png?raw=true)

```
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  hpM8wQzc+b*2q-06AF6j

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  d04be114373aff451c0af1665aa7affabc6f07b49edd8734ae36ff504a42c938

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjEuMCIsImFkciI6WyIxMC4wLjEyLjExOjkyMDAiXSwiZmdyIjoiZDA0YmUxMTQzNzNhZmY0NTFjMGFmMTY2NWFhN2FmZmFiYzZmMDdiNDllZGQ4NzM0YWUzNmZmNTA0YTQyYzkzOCIsImtleSI6IlBXTnJxNFlCNTJpcjlEcGIwTVpNOmtoeU03LXpiVEhPcng1WVYtZ0l5YmcifQ==

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.

```

(4) 运行报错

```
ERROR: [1] bootstrap checks failed. You must address the points described in the following [1] lines before starting Elasticsearch.
bootstrap check failure [1] of [1]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
ERROR: Elasticsearch did not exit normally - check the logs at /home/elasticsearch/es/elasticsearch-8.1.0/logs/elasticsearch.log

```


需要修改系统配置（root权限）

```
sysctl -w vm.max_map_count=262144
```


(5) 查看集群状态，无返回值

```
curl -u "elastic:hpM8wQzc+b*2q-06AF6j" 127.0.0.1:9200

➜  ~ curl -u "elastic:hpM8wQzc+b*2q-06AF6j" 127.0.0.1:9200
curl: (52) Empty reply from server

[2023-03-04T15:13:22,198][WARN ][o.e.x.s.t.n.SecurityNetty4HttpServerTransport] [VM-12-11-centos] received plaintext http traffic on an https channel, closing connection Netty4HttpChannel{localAddress=/127.0.0.1:9200, remoteAddress=/127.0.0.1:50162}
```

|序号 | 方法| | 
|---|---|---|
|1 | 因为elasticsearch8.1 默认是开启tls 安全配置，所以，需要使用https 并且证书认证方式访问 |`curl --cacert /home/elasticsearch/es/elasticsearch-8.1.0/config/certs/http_ca.crt  -u"elastic:hpM8wQzc+b*2q-06AF6j" https://127.0.0.1:9200/` |
|2 | 可以使用curl -k 参数，忽略证书认证 | `curl -k -u"elastic:hpM8wQzc+b*2q-06AF6j" https://127.0.0.1:9200/`|
|3 | 修改配置，关闭https认证，使用账号、密码认证，即`xpack.security.http.ssl: enabled: false`, 详见下文| `curl -u"elastic:hpM8wQzc+b*2q-06AF6j" http://127.0.0.1:9200/` | 
|4 | 关闭所有的安全认证, 即`xpack.security.enabled: false`|`curl http://127.0.0.1:9200/`| 


安全配置说明

```
# Enable security features
xpack.security.enabled: true # 表示是否开启xpack安全模块，如果设置为false，不需要https访问，也不需要账号密码基本认证

xpack.security.enrollment.enabled: true 

# Enable encryption for HTTP API client connections, such as Kibana, Logstash, and Agents
xpack.security.http.ssl:
  enabled: true # 表示是否开启https访问，如果设置为false，即只需要基本的账号密码认证
  keystore.path: certs/http.p12

# Enable encryption and mutual authentication between cluster nodes
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
# Create a new cluster with the current node only
# Additional nodes can still join the cluster later
cluster.initial_master_nodes: ["VM-12-11-centos"]

# Allow HTTP API connections from localhost and local networks
# Connections are encrypted and require user authentication
http.host: [_local_, _site_]

# Allow other nodes to join the cluster from localhost and local networks
# Connections are encrypted and mutually authenticated
#transport.host: [_local_, _site_]
```

正常运行

```

➜  ~ curl -u"elastic:hpM8wQzc+b*2q-06AF6j" http://127.0.0.1:9200/
{
  "name" : "VM-12-11-centos",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "suMBivwmSJCHFPNduQCvrQ",
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

```

(6) 本地请求

注意⚠️ ： 需要配置腾讯云防火墙规则，允许9200 端口访问



![login](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/login.png?raw=true)

![login-2](https://github.com/Kua-Fu/blog-book-images/blob/main/es-cert/login-2.png?raw=true)


### 2.3 kibana 访问

然后，修改解压目录权限

```
chown -R elasticsearch:elasticsearch kibana-8.1.0
```

