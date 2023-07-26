# basic

## 一、参考

> [kafka quick start](https://kafka.apache.org/documentation/#quickstart)


## 二、安装运行

### 2.1 下载

下载最新的kafka [download](https://www.apache.org/dyn/closer.cgi?path=/kafka/3.5.0/kafka_2.13-3.5.0.tgz)

### 2.2 运行

```

# 1. 解压

tar -xzf kafka_2.13-3.5.0.tgz
mv kafka_2.13-3.5.0 kafka

# 2. 运行zookeeper
./bin/zookeeper-server-start.sh config/zookeeper.properties

# 3. 运行kafka
./bin/kafka-server-start.sh config/server.properties
```
