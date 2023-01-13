# Memory Analyze

## 一、参考

> [MEMORY USAGE](https://redis.io/commands/memory-usage/)

> [info memory](https://redis.io/commands/info/)

## 二、内置命令

### 2.1 info memory


* used_memory 由 Redis 分配器分配的内存总量，包含了redis进程内部的开销和数据占用的内存，以字节（byte）为单位，即当前redis使用内存大小。

* used_memory_human 已更直观的单位展示分配的内存总量。

* used_memory_rss 向操作系统申请的内存大小，与 top 、 ps等命令的输出一致，即redis使用的物理内存大小。

* used_memory_rss_human 已更直观的单位展示向操作系统申请的内存大小。

* used_memory_peak redis的内存消耗峰值(以字节为单位)，即历史使用记录中redis使用内存峰值。

* used_memory_peak_human 以更直观的格式返回redis的内存消耗峰值

* used_memory_peak_perc 使用内存达到峰值内存的百分比，used_memory/ used_memory_peak) *100%，即当前redis使用内存/历史使用记录中redis使用内存峰值*100%

* used_memory_overhead Redis为了维护数据集的内部机制所需的内存开销，包括所有客户端输出缓冲区、查询缓冲区、AOF重写缓冲区和主从复制的backlog。

* used_memory_startup Redis服务器启动时消耗的内存

* used_memory_dataset 数据实际占用的内存大小，即 used_memory-used_memory_overhead

* used_memory_dataset_perc 数据占用的内存大小的百分比，100%*(used_memory_dataset/(used_memory-used_memory_startup))

* total_system_memory 整个系统内存

* total_system_memory_human 以更直观的格式显示整个系统内存

* used_memory_lua Lua脚本存储占用的内存

* used_memory_lua_human 以更直观的格式显示Lua脚本存储占用的内存

* maxmemory Redis实例的最大内存配置

* maxmemory_human 以更直观的格式显示Redis实例的最大内存配置

* maxmemory_policy 当达到maxmemory时的淘汰策略

* mem_fragmentation_ratio 碎片率，used_memory_rss/ used_memory。ratio指数>1表明有内存碎片，越大表明越多，<1表明正在使用虚拟内存，虚拟内存其实就是硬盘，性能比内存低得多，这是应该增强机器的内存以提高性能。一般来说，mem_fragmentation_ratio的数值在1 ~ 1.5之间是比较健康的。详解

* mem_allocator 内存分配器

* active_defrag_running 表示没有活动的defrag任务正在运行，1表示有活动的defrag任务正在运行（defrag:表示内存碎片整理）详解

* lazyfree_pending_objects 0表示不存在延迟释放的挂起对象


### 2.2 memory usage

memory usage 命令可以返回具体的 key 占用的内存大小，单位是字节

```

127.0.0.1:6379> memory usage hash_big_key
(integer) 81776897

```

