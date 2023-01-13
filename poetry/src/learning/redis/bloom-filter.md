# Bloom Filter

## 一、参考

> [Redis Best Practices ——  Bloom Filter](https://redis.com/redis-best-practices/bloom-filter-pattern/)

> [bloom模块](https://github.com/RedisBloom/RedisBloom)

> [吃透Redis系列（四）：布隆（bloom）过滤器详细介绍](https://blog.csdn.net/u013277209/article/details/112376005)

> [go-redis bloom](https://github.com/go-redis/redis/blob/27f0addeb2e6997bdd2faa1b48f6a989f83c4751/example/redis-bloom/main.go)

## 二、添加 bloom 模块

### 2.1 源码安装

[redis bloom 源码](https://github.com/RedisBloom/RedisBloom/releases?page=1)

```

tar -zxvf RedisBloom-2.2.18.tar.gz

cd RedisBloom-2.2.18

make

```

将编译后的动态库，移动到 redis-server 安装目录

```

cp redisbloom.so ../redis-5.0.0/

```

### 2.2 运行 redis-server


```

redis-server --loadmodule ./redisbloom.so

```

## 三、redis-cli 基本使用

### 3.1 添加 key

```

127.0.0.1:6379> BF.ADD usernames user1
(integer) 1
127.0.0.1:6379> BF.ADD usernames user2
(integer) 1
127.0.0.1:6379> BF.ADD usernames user3
(integer) 1

```

### 3.2 添加多个keys

```

127.0.0.1:6379> BF.MADD usernames user11 user22 user33
1) (integer) 1
2) (integer) 1
3) (integer) 1


```

### 3.2 判断 key 是否存在

```

127.0.0.1:6379> BF.EXISTS usernames user1
(integer) 1
127.0.0.1:6379> BF.EXISTS usernames user4
(integer) 0

```

### 3.3 判断多个 keys 是否存在

```

127.0.0.1:6379> BF.MEXISTS usernames user1 user2 user3 user4
1) (integer) 1
2) (integer) 1
3) (integer) 1
4) (integer) 0

```


## 四、go-redis 基本使用


```golang

package main

import (
	"fmt"
	"testing"

	"github.com/go-redis/redis"
)

// 创建 redis 客户端
func createRedisClient() (*redis.Client, error) {
	client := redis.NewClient(&redis.Options{
		Addr:     "localhost:6379",
		Password: "abc123",
		DB:       0,
		PoolSize: 5,
	})

	pong, err := client.Ping().Result()
	fmt.Println(pong, err)
	if err != nil {
		return nil, err
	}
	return client, nil
}

// 测试 bloom filter
func TestBloomFilter(t *testing.T) {

	cli, err := createClient()
	if err != nil {
		return
	}

	bKey := "bf_key"

	// 1. 添加一个值

	v := "item0"

	inserted, err := cli.Do("BF.ADD", bKey, v).Result()

	if err != nil {
		panic(err)
	}

	res, _ := inserted.(int64)

	if res > 0 {
		fmt.Printf("%s was inserted \n", v)
	} else {
		fmt.Printf("%s already exists \n", v)
	}

	// // 2. 添加多个值

	bools, err := cli.Do("BF.MADD", bKey, "v1", "v2", "v3").Result()

	if err != nil {
		panic(err)
	}

	fmt.Println("--bools--", bools)

	// 3. 判断某个 key 是否存在
	for _, item := range []string{"item0", "item1"} {
		exists, err := cli.Do("BF.EXISTS", bKey, item).Result()
		if err != nil {
			panic(err)
		}
		res, _ := exists.(int64)
		if res > 0 {
			fmt.Printf("%s does exist\n", item)
		} else {
			fmt.Printf("%s does not exist\n", item)
		}
	}

	// 4. 判断多个 key 是否存在

	values := []string{"item0", "item1"}
	exists, err := cli.Do("BF.MEXISTS", bKey, "item0", "item1").Result()

	if err != nil {
		panic(err)
	}

	nExists, _ := exists.([]interface{})

	for i, res := range nExists {
		r, _ := res.(int64)
		if r > 0 {
			fmt.Printf("%s does exist\n", values[i])
		} else {
			fmt.Printf("%s does not exist\n", values[i])
		}
	}

}



```

## 五、benchmark 

### 5.1 测试 1亿个 value 占用多个redis内存

```golang

// 测试添加 1亿个不同的value，例如:
func TestBloomMemory(t *testing.T) {

	cli, err := createClient()
	if err != nil {
		return
	}

	f, err := os.Create("/tmp/bloom.log")
	if err != nil {
		return
	}

	defer f.Close()
	bKey := "bloom_big_key"

	for i := 0; i < 100000000; {

		v := xid.New().String()

		inserted, err := cli.Do("BF.ADD", bKey, v).Result()

		if err != nil {
			panic(err)
		}

		res, _ := inserted.(int64)

		if res > 0 {
			i++
			s1 := fmt.Sprintf("%s was inserted, %d \n", v, i)
			f.WriteString(s1)
		} else {
			s2 := fmt.Sprintf("%s already exists \n", v)
			f.WriteString(s2)
		}
	}
}

```

最终写入太慢了，测试结果如下: 

600 万数据，占用内存 25M， 原始数据大小为 122M, 压缩比例为 20%

```
cbr1scq87d5st57kkujg was inserted, 6424491 
cbr1scq87d5st57kkuk0 was inserted, 6424492 
cbr1scq87d5st57kkukg was inserted, 6424493 
cbr1scq87d5st57kkul0 was inserted, 6424494 
cbr1scq87d5st57kkulg was inserted, 6424495 
cbr1scq87d5st57kkum0 was inserted, 6424496

```

```

127.0.0.1:6379> info memory
# Memory
used_memory:26676736
used_memory_human:25.44M
used_memory_rss:27537408
used_memory_rss_human:26.26M
used_memory_peak:26731104
used_memory_peak_human:25.49M
used_memory_peak_perc:99.80%
used_memory_overhead:1054536
used_memory_startup:987744
used_memory_dataset:25622200
used_memory_dataset_perc:99.74%
allocator_allocated:26639808
allocator_active:27499520
allocator_resident:27499520
total_system_memory:17179869184
total_system_memory_human:16.00G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:1.03
allocator_frag_bytes:859712
allocator_rss_ratio:1.00
allocator_rss_bytes:0
rss_overhead_ratio:1.00
rss_overhead_bytes:37888
mem_fragmentation_ratio:1.03
mem_fragmentation_bytes:897600
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:66600
mem_aof_buffer:0
mem_allocator:libc
active_defrag_running:0
lazyfree_pending_objects:0

```

```

127.0.0.1:6379> memory usage bloom_big_key
(integer) 25580961

```

### 5.2 bloom filter vs hash

用时 32 min，写入同样的数据，6424496 条记录，占用内存为 364M, 原始数据大小为 122M, 膨胀了3倍，即是 bloom filter 保存的 14 倍

```

--create time-- 2022-08-12 17:54:33.806326 +0800 CST m=+0.007609377
--finish gen-- 2022-08-12 18:27:32.910779 +0800 CST m=+1979.087582115 32m59.079972738s


```


```

127.0.0.1:6379> info memory
# Memory
used_memory:607731696
used_memory_human:579.58M
used_memory_rss:557879296
used_memory_rss_human:532.04M
used_memory_peak:607739888
used_memory_peak_human:579.59M
used_memory_peak_perc:100.00%
used_memory_overhead:1037694
used_memory_startup:987744
used_memory_dataset:606694002
used_memory_dataset_perc:99.99%
allocator_allocated:607694784
allocator_active:557841408
allocator_resident:557841408
total_system_memory:17179869184
total_system_memory_human:16.00G
used_memory_lua:37888
used_memory_lua_human:37.00K
used_memory_scripts:0
used_memory_scripts_human:0B
number_of_cached_scripts:0
maxmemory:0
maxmemory_human:0B
maxmemory_policy:noeviction
allocator_frag_ratio:0.92
allocator_frag_bytes:18446744073659698240
allocator_rss_ratio:1.00
allocator_rss_bytes:0
rss_overhead_ratio:1.00
rss_overhead_bytes:37888
mem_fragmentation_ratio:0.92
mem_fragmentation_bytes:18446744073659736128
mem_not_counted_for_evict:0
mem_replication_backlog:0
mem_clients_slaves:0
mem_clients_normal:49686
mem_aof_buffer:0
mem_allocator:libc
active_defrag_running:0
lazyfree_pending_objects:0

```

```

127.0.0.1:6379> memory usage hash_big_key
(integer) 381909320

```

