# 基本使用


## 一、参考

> [Install Redis from Source](https://redis.io/docs/getting-started/installation/install-redis-from-source/)


## 二、基本使用

### 2.1 源码下载安装

[redis release](https://download.redis.io/releases/)

```

tar -xzvf redis-5.0.0.tar.gz

cd redis-5.0.0

make

make install


```


```

➜  redis-5.0.0 redis-server --version
Redis server v=5.0.0 sha=00000000:0 malloc=libc bits=64 build=4e0d65cbfe517bf4

```

### 2.2 设置密码

redis配置文件 redis.conf 中，可以设置参数 requirepass, 表示启动密码服务

```
requirepass 自定义密码

```

### 2.3 运行 redis-server

```

redis-server ./redis.conf

```

### 2.4 客户端连接

```

redis-cli -a 自定义密码


```

