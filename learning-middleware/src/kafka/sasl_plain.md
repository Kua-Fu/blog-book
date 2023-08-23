# SASL/PLAIN

## 一、参考

> [Authentication using SASL/PLAIN](https://kafka.apache.org/documentation/#security_sasl_plain)

> [给 Kafka 配置 SASL/PLAIN 认证](https://kyle.ai/blog/7631.html)

> [Kafka3.3.1版本配置SASL认证](https://www.jianshu.com/p/9cb16ec086e4)


## 二、修改 zookeeper 配置

下面的配置都是 kafka3.5.0 版本

### 2.1 添加认证

创建新文件 `./config/zookeeper_server_jaas.conf`

```

Server {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="zkadmin"
    password="zkadminpassword"
    user_zk="zkpassword";
};

```


### 2.2 修改启动脚本

在zookeeper 启动脚本 `./config/zookeeper-server-start.sh` 中添加如下配置

```

export KAFKA_OPTS="-Djava.security.auth.login.config=$base_dir/../config/zookeeper_server_jaas.conf"

```


### 2.3 修改zookeeper 配置

在 `./config/zookeeper.properties` 中添加下面的配置

```

authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
requireClientAuthScheme=sasl
jaasLoginRenew=3600000

```

### 2.4 启动zookeeper

```

./bin/zookeeper-server-start.sh config/zookeeper.properties

```


## 三、修改 kafka 配置



### 3.1 添加认证

创建新文件 `./config/kafka_server_jaas.conf`

```shell

KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="admin-kafka"
	user_admin="adminpassword"
    user_yztest="yztestpassword";
};

Client {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="zk"
    password="zkpassword";
};



```

⚠️ 注意: 

`user_admin="adminpassword" `这个记录必须存在，是由 `username` 和 `password` 组合成

如果 `username: yzadmin password: yzpassword`, 则该字段是 `user_yzadmin="yzpassword"`



### 3.2 修改启动脚本

修改 `./bin/kafka-server-start.sh`

添加一行

`export KAFKA_OPTS="-Djava.security.auth.login.config=$base_dir/../config/kafka_server_jaas.conf"`


### 3.3 修改server 配置

修改 `config/server.properties`, 添加配置


```

listeners=SASL_PLAINTEXT://10.100.65.93:9092
advertised.listeners=SASL_PLAINTEXT://10.100.65.93:9092
authorizer.class.name=kafka.security.authorizer.AclAuthorizer
allow.everyone.if.no.acl.found=true
security.inter.broker.protocol= SASL_PLAINTEXT
sasl.mechanism.inter.broker.protocol=PLAIN
sasl.enabled.mechanisms=PLAIN
super.users=User:admin

```

### 3.4 运行kafka

```

./bin/kafka-server-start.sh config/server.properties

```


## 四、配置kafka ui


![sasl-plain](https://github.com/Kua-Fu/blog-book-images/blob/main/kafka/sasl_plain.png?raw=true)