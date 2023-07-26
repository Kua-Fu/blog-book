# SASL/SCRAM-SHA-256


## 一、参考

> [kafka SASL/SCRAM Failed authentication](https://stackoverflow.com/questions/61594103/kafka-sasl-scram-failed-authentication)

> [kafka的SCRAM配置](https://zixujing.github.io/2019/12/26/bigdata/kafka%E7%9A%84SCRAM%E9%85%8D%E7%BD%AE/)


## 二、创建认证

下面的配置都是 kafka3.5.0 版本

```

sh bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=zkpassword]' --entity-type users --entity-name zk

sh bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=admin-kafka]' --entity-type users --entity-name admin

sh bin/kafka-configs.sh --zookeeper localhost:2181 --alter --add-config 'SCRAM-SHA-256=[password=yztestpassword]' --entity-type users --entity-name yztest

```

⚠️ 注意: 

需要把各个用户都配置证书


查看认证信息

```

bin/kafka-configs.sh --zookeeper localhost:2181 --describe --entity-type users --entity-name zk

```


```

Warning: --zookeeper is deprecated and will be removed in a future version of Kafka.
Use --bootstrap-server instead to specify a broker to connect to.
Configs for user-principal 'zk' are SCRAM-SHA-256=salt=aDd1d2I1dGgxM2M3b2VnYW5iaTYxbnZ4eg==,stored_key=4hnkE0CdDzmo2K5YJ//3hKRbqwqTQ7LRNFQFPI+yNdo=,server_key=tI14lMMGmm1gBMohC/uoQgw5q+KyuwkSikHv7Od4zNk=,iterations=4096

```

## 三、修改zookeeper 配置

### 3.1 添加认证

创建新文件 `./config/zookeeper_server_jaas.conf`

```

Server {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="zkadmin"
    password="zkadminpassword"
    user_zk="zkpassword";
};

```

### 3.2 修改启动脚本

在zookeeper 启动脚本 `./config/zookeeper-server-start.sh` 中添加如下配置

```

export KAFKA_OPTS="-Djava.security.auth.login.config=$base_dir/../config/zookeeper_server_jaas.conf"


```

### 3.3 修改 zookeeper 配置


在 `./config/zookeeper.properties` 中添加下面的配置

```

authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider
requireClientAuthScheme=sasl
jaasLoginRenew=3600000

```

### 3.4 启动zookeeper

```

./bin/zookeeper-server-start.sh config/zookeeper.properties

```


## 四、修改kafka 配置

### 4.1 创建新文件 `./config/kafka_server_jaas.conf`


```

KafkaServer {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="admin"
    password="admin-kafka"
    user_admin="adminpassword"
    user_yztest="yztestpassword";
};

Client {
    org.apache.kafka.common.security.scram.ScramLoginModule required
    username="zk"
    password="zkpassword";
};


```

### 4.2 修改启动脚本

修改 `./bin/kafka-server-start.sh`

添加一行

`export KAFKA_OPTS="-Djava.security.auth.login.config=$base_dir/../config/kafka_server_jaas.conf"`



### 4.3 修改server 配置


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


### 4.4 运行kafka

```

./bin/kafka-server-start.sh config/server.properties

```


## 五、配置kafka ui

![scram-sha-256](https://github.com/Kua-Fu/blog-book-images/blob/main/kafka/scram_sha_256.png?raw=true)
