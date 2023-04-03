# 搜索


## 一、搜索命令

### 1.1 按照执行阶段分类

|序号 | 类型 | 说明|
|---|---|---|
|1 | 分布式命令| 可以在引擎分布式执行，也可以在SPL单机执行|
|2 | 集中式命令| 只能在SPL 单机执行|


分布式命令，是否分布式执行规则如下

|序号 | 规则| 说明|
|---|---|---|
|1 | 如果一个命令是分布式命令，执行语句之前的命令也都是分布式命令，那么会分布式运行| `* ｜eval …​ ｜ parse …` 其中eval 和 parse 都是分布式命令，所以会是分布式运行|
|2 | 如果一个命令是单机执行的，那么之后的命令都会是单机执行| `* ｜ transaction …​｜ eval …​ ｜ parse …​` 中的transaction是集中式命令，所以eval，parse都是单机执行|

* 集中式命令有

	`sort`, `append`, `appendcols`, `autoregress`, `collect`, `custom`, `command`, `dbxlookup`, `ldaptestconnection`, `dbxoutput`, `lookup`, `dbxquery`, `lookup2`, `dedup`, `limit`, `delete`, `makecontinuous`, `download`, `makeresults`, `esma`, `map`, `eventstats`, `movingavg`, `filldown`, `mvcombine`, `gentimes`, `mvexpand`, `inputlookup`, `outputlookup`, `join`, `rollingstd`, `ldapfetch`, `save`, `ldapfilter`, `streamstats`, `ldapgroup`, `table`, `ldapsearch`, `timewrap`, `transaction`, `transpose`, `top`, `unpivot`, `loadjob`, `history`, `localop`, `accum`, `untable`, `rest`, `fromes`, `fromkafkapy`, `correlation`



* 分布式命令有


	`addinfo`, `bucket`, `eval`, `chart`, `fields`, `fillnull`, `foreach`, `geostats`, `iplocation`, `jpath`, `kvextract`, `parse`, `partition`, `stats`, `timechart`, `where`, `xpath`, `rare`, `rename`, `replace`, `makemv`, `strcat`, `typeahead`, `history`


### 1.2 按照功能分类

|序号 | 名称 | 说明| 
|---|---|---|
|1 | 生成命令| 生成数据的命令，用于产生数据，通常是命令或者子命令的第一个命令 |
|2 | 流式命令|流式命令对数据一行一行处理，处理一行产生一行结果 |
|3 | 转换命令| 把输入命令作为一个整体来处理， 需要所有数据才能产生结果，比如transpose，dedup | 

⚠️ 

1. 流式命令分为分布式流式命令和集中式流式命令

	* 分布式流式命令:

	  是可以分布式执行的流式命令，对数据一行一行处理，处理一行产生一行结果，不依赖全局顺序和上下行的命令，比如`eval，parse`

	* 集中式流式命令:

	  也对数据一行一行处理，和分布式流式命令区别在，依赖输入数据的顺序，比如`autoregress，filldown`

	  还有一些集中式流式命令是暂时只在SPL实现的，不在引擎执行的命令，具体参考命令类型列

1. 转换命令在SPL执行，不可分布式执行

* 生成命令有

	`search`, `multisearch`, `union`, `gentimes`, `inputlookup`, `makeresults`, `dbxquery`, `history`, `loadjob`, `rest`, `typeahead`, `fromes`, `fromkafkapy`
	
* 流式命令有

	`addinfo`, `append`, `appendcols`, `bucket`, `collect`, `dbxlookup`, `eval`, `fields`, `filldown`, `fillnull`, `foreach`, `head`, `iplocation`, `join`, `jpath`, `kvextract`, `limit`, `lookup`, `lookup2`, `movingavg`, `mvexpand`, `nomv`, `parse`, `rename`, `replace`, `rollingstd`, `streamstats`, `table`, `transaction`, `where`, `xpath`, `makemv`, `localop`, `strcat`, `accum`, `untable`

* 转换命令有

	`chart`, `dbxoutput`, `dedup`, `esma`, `eventstats`, `geostats`, `makecontinuous`, `map`, `mvcombine`, `outputlookup`, `rare`, `save`, `sort`, `stats`, `timechart`, `timewrap`, `top`, `transpose`, `unpivot`, `correlation`



















