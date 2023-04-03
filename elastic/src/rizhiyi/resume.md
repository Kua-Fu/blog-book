# 命令简述

|命令|描述|示例|
|---|---|---|
|`search` | 指定的搜索条件，或者作为过滤条件 | `host: localhost AND status: >=200`|
|`multisearch`| 同时执行多个搜索，子查询只允许分布式流式命令 | …​ \| multisearch [[ status: 200 \| eval tag=succ ]] [[ status: 404 \| eval tag=err ]] |
| `addinfo` | 向每个事件添加包含有关搜索的全局通用信息的字段| …​ \| addinfo|
| `append` | 将子管道的结果附加在主管道的结果之后 | …​ \| append [[ * \| stats max(status) by appname ]]|
| `union`| 同时执行多个搜索，子查询可以用任意命令 | …​ \| union [[ * \| stats max(status) by appname ]] [[ * \| stats max(status) by tag ]]| 
| `appendcols` | 添加一个子搜索，并将子搜索的结果按顺序合并到父搜索上 | …​ \| appendcols [[ * \| stats min(timestamp)]]|
| `autoregress`| 拷贝一个或者多个当前事件之前的事件中的字段值到当前事件| …​ \| autoregress clientip p=1-2|
| `chart`| 按照over字段进行分桶后的统计行为| …​ \| chart sep="," format="$VAL**$AGG" limit=5 cont=false rendertype="pie" count(apache.x_forward) over apache.status | 
|`collect`| 将查询的结果写到索引| …​ \|collect index=test marker="appname=\"test\", tag=\"tag1\"" | 
| `bucket`| 将连续的值放入离散集中 | …​ \| bucket timestamp span=1h as ts|
|`dedup`| 对搜索结果中指定字段值的重复情况进行去重和过滤| …​ \| dedup 3 apache.status, apache.geo.city | 
| `dbxlookup` | 类似sql的连接，将来自远程数据库表的结果和子管道的结果连接在一起 | \| dbxlookup lookup="test1" <br>  \| dbxlookup test1,test2 connection="179test" query="select * from test" on id=bid | 
| `dbxoutput`| 将当前搜索的数据按照已配置的dbxoutput名称写出到远程数据库。 | \| dbxoutput output="test1" | 
| `dbxquery` | 是一个可以使用sql来查远程数据库的数据并作为spl的查询语句的命令 | \| dbxquery connection="179test" query="select * from test" | 
| `esma` | 对某一个字段的未来值进行预测 | …​ \| esma latency timefield=ts period=7 futurecount=30|
|`eval` | 计算表达式，并将表达式的值放入字段中，请参阅 搜索命令函数 | …​ \| eval username = case(user_name, user) | 
| `eventstats` | 提供统计信息，可以选择字段进行分组，并且将按照当前行所属于的分组的统计结果作为新的字段值添加在本行 | …​ \| eventstats count() by logtype | 
| `fields` | 通过操作符保留或排除结果中的系列字段| …​ \| fields status, clientip | 
|`filldown` | 将某些字段的null值用之前最近的非null值进行填充，支持通配符| …​ \| filldown hostname app*|
|`fillnull` | 将空值替换为指定值 | …​ \| fillnull value="aaa" foo,bar |
| `gentimes` | 可以生成指定时间范围内的时间戳| \| gentimes start="2019-01-01:00:00:00" end="2019-01-04:00:00:00"|
| `geostats` | 可以基于地理位置信息，即经纬度进行分区域统计| …​ \| geostats count(appname) | 
| `inputlookup` | 可以读取csv文件 | \| inputlookup a.csv | 
| `join` | 类似sql的连接，将来自主管道的结果和子管道的结果连接在一起 | …​ \| join type=left clientip [[ * \| stats avg(resp_len) by clientip ]]|
| `jpath` | 类似xpath抽取json中的字段值 | …​ \| jpath output=prices path="store.book[*].price" |
|`kvextract` | 将指定字段按键值对抽取 | …​ \| kvextract json.kvex| 
| `limit`  | 返回前n个结果 | …​ \| limit 10 | 
| `lookup` | 显示调用字段值查找 | …​ \| lookup email http://data.cn/user.csv on id=userId|
| `lookup2` | 显示调用指定方式查找 | …​ \| lookup2 external_file outputfields appname,hostname| 
| `makecontinuous` | 在一定数值或时间范围内，根据给定的区间大小，对原始数据升序处理，并补充不连续的区间，区间的划分采用向前圆整的方式 | …​ \| makecontinuous time span=3 start=216 end=226 |
| `makeresults` | 构造指定的结果 | \| makeresults count=1 | 
| `map` | 将前一个查询的结果用于下一个查询| …​ \| map "apache.status:$apache.status$ \| stats count()" | 
| `movingavg` | 计算移动平均值 | …​ \| movingavg sum_len,10 as smooth_sum_len | 
| `mvexpand` | 拆分多值字段 | …​ \| mvexpand iplist limit=100 | 
|` mvcombine` | 合并指定字段  | 	…​ \| mvcombine sep="," ip | 
| `nomv` | 将多值字段转换为单值字段 | …​ \| nomv delim="," a|
|`outputlookup`| 导出csv文件 | …​ \| outputlookup createempty=false overrideifempty=false maxresult=100 filename.csv |
| `parse` | 搜索时抽取字段 | …​ \| parse "(?<ip_addr>\d+\.\d+\.\d+\.\d+)"|
| `rename` | 重新命名指定字段 | …​ \| rename apache.status as http_code | 
|`rollingstd` | 计算移动的标准差 | …​ \| rollingstd sum_resp_len, 10 as resp_len_rolling_std | 
| `save` | 将搜索结果输出为外部文件 | …​ \| save /data/spldata/apahce_clientip.csv  | 
| `sort` | 按照指定的字段对结果进行排序 | …​ \| sort by apache.status | 
| `stats` | 提供统计信息，可以选择按照字段分组  | …​ \| stats count() by apache.method| 
| `streamstats` | 连续统计 | …​ \| streamstats count() as cnt | 
| `table` | 将查询结果以表格形式展示，并对字段进行筛选| …​ \| table apache.status, apache.method | 
| `timechart` | 对时间分桶进行统计查询 | …​ \| timechart limit=5 bins=10 minspan=1m span=10m max(x) as ma count() as cnt by apache.geo.city | 
| `timewrap` | 对timechart命令的结果 进行展示或者折叠| …​ \| top 3 apache.clientip by apache.method | 
| `top` | 返回指定字段top的值集合| …​ \| top 3 apache.clientip by apache.method | 
| `rare` | 返回指定字段最少出现次数的值集合 | …​ \| rare apache.clientip by apache.method | 
| `transaction` | 将结果分组成交易 | …​ \| transaction apache.clientip startswith="Android 4.3" endswith="AndroidPhone" maxopenevents = 10 | 
| `transpose` | 将查询的表格结果进行行列转换 | …​ \| transpose row=apache.method column=apache.status valuefield=cnt | 
| `where` | 使用表达式对结果进行过滤 | …​ \| where apache.status < 200 && apache.status>400 | 
| `xpath` | 提供对xml数据的处理和抽取 | …​ \| xpath input=json.xp output=lyly path="/purchases/book/title" | 
| `unpivot` | 行转列转换 | …​ \| unpivot 10 header_field=count column_name=title | 
| `foreach` | 对字段列表执行流式命令 | …​ \| foreach count* [[ eval <<FIELD>> = <<FIELD>> + 1 ]] | 
| `iplocation` | 从ip地址抽取地理信息 | …​ \| iplocation clientip | 
| `replace` | 使用指定字符串替换字段值，可以指定一个或多个字段，仅替换指定字段的值，如果没有指定字段，则替换所有字段 | …​ \| replace "192.168.1.1" with "localhost" | 
| `makemv` | 使用分隔符或者带捕获组的正则表达式，将单值字段转换为多值字段 | …​ \| makemv delim="," testmv | 
| `localop` | localop命令强制随后的命令都是spl 单机执行| …​ \| localop | 
| `strcat` | 连接来自2个或更多字段的值。将字段值和指定字符串组合到一个新字段中 | …​ \| eval field1=\"10.192.1.1\",field2=\"192.168.1.1\" \|strcat field1 \"abcd\" field2 | 
| `loadjob` | 加载先前完成的定时任务或告警的执行结果。由ID 和type唯一确定一个任务。如果最近一次时间点的结果不存在，则临时运行原始查询。|…​ \| loadjob id=1,type="savedschedule" | 
| `accum` | 对每个事件中为数字的指定字段进行逐次累加，得到的累加结果会放入该字段或者新字段中。 | …​ \| accum apache.resp_len as sum_resp_len | 
| `untable` | table指令的逆操作，使用该指令可以将表格的查看方式转换到事件列表的查看方式。 | …​ \| untable | 
| `rest` | 调用日志易API，返回对应结果 | …​ \| rest /agentgroup/ apikey="user apikey" count=2| 
| `typeahead` | 返回指定前缀的字段信息。返回的最大结果数取决于为size参数指定的值。typeahead命令可以以指定索引为目标，并受时间限制| …​ \| typeahead prefix="app" size=5 index=yotta| 
| `history` | 查看搜索历史记录 | …​ \| history|
| `correlation` | 计算的与搜索结果相关性高的字段与字段值 | …​ \| bucket timestamp ranges=0, 1000),(1000, 10000),(1000, 1753587702986 as ts\| correlation bucket_field=ts\|sort by correlation\| sort by -ts | 
| `fromes` | 搜索指定索引的数据 | \|fromes host=10.200.0.140 index=logs-my_app-default querydsl='{"query": {"match_all": { }}}' | 
| `fromkafkapy`| 消费指定主题的数据 | \|fromkafkapy topic=test | 

