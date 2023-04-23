# stats相关函数

## 一、函数介绍

eval <expression>介绍：

1. 可以在通常使用<field>的统计函数中使用eval <expression>，此时必须使用 as newName。

1. | stats func(eval(<expression>)) as newName 等同于 | eval temp_field = <expression> | stats func(temp_field) as newName.

1. 唯一的特殊情况，当统计函数为count(), 且expression结果为bool类型时，统计当结果仅为当expression 为true 的事件数

	例如：…​ | stats count(eval(status="404")) AS count_status BY sourcetype, 统计的结果为status 的404的事件总数
	
以下是可与stats等命令使用的统计函数，后续将扩展到其他的命令。约定：

"X" 表示指定字段名， +

"INTERVAL"表示指定时间间隔，描述方式如1m, 1d… 后缀有以下几种：y|M|w|d|h|m|s ， +

"LIMIT"表示返回值的限制数量。

## 二、分类

### 2.1 single函数

avg、count、distinct_count / estdc / dc、distinct、earliest、first、last、latest、max、min、rate、stddev、sum、sumsq、var

|  函数|  描述| 示例|
| --- | --- | --- |
| avg(X) | 返回字段X的平均值 | 返回平均响应时间：avg(response_time) |
| count([X]) | 返回字段X的出现次数 | 返回status的个数： count(status) |
| distinct(X)| 返回字段X的值去重后的个数准确值| 返回字段clientip的唯一值值的个数的精确值： distinct(clientip) |
| distinct_count(X) estdc(X) dc(X)| 返回字段X的值去重后的个数的估计值|返回clientip的唯一值值的个数的估计值： dc(clientip) |
| earliest(X)| 返回字段X按照时间增序排序后的第一个值 | 返回appname字段按照时间增序排序后的第一个值： earliest(appname) |
|first(X)| 返回数据中字段X的第一个出现的值| 返回第一个appname的值： first(appname) |
| last(X) | 返回字段X的最后一个出现的值| 返回数据中最后一个appname的值： last(appname) |
| latest(X) | 返回字段X按照时间增序排序后的最后一个值 | 返回数据中的appname字段按照时间增序排序后的最后一个值： latest(appname) |
| mad(X) | 此函数将统计指定字段的绝对中位差(MAD)值 | 以下示例返回响应时间的绝对中位差(MAD)值: stats mad(response_time) |
| max(X)| 返回字段X的最大值 字段X必须为数值类型 | 返回响应时间的最大值： max(response_time) |
|min(X)| 返回字段X的最小值 字段X必须为数值类型 | 返回响应时间的最小值： min(response_time) |
|rate(X)| 此函数将统计在指定时间跨度内指定字段值的变化速率。 具体计算方法：(latest - earliest) / (latestT - earliestT) latest为字段X按照时间增序排序后的最后一个值 earliest为字段X按照时间增序排序后的第一个值 latestT为latest对应的timestamp earliestT为earliest对应的timestamp | 返回数据中apache.resp_len值的变化速率： *\|stats rate(apache.resp_len) |
|stddev(X)| 统计字段X的标准差 字段X必须为数值类型 | 返回响应时间的标准差： stats stddev(response_time) |
| sum(X) | 返回字段X的值的和 字段X必须为数值类型 | 返回响应长度的和： sum(response_len) | 
| sumsq(X)| 统计字段X的平方和 字段X必须为数值类型 | 返回响应时间的平方和： stats sumsq(response_time) |
| var(X)| 统计字段X的方差 字段X必须为数值类型 | 返回响应时间的方差： stats var(response_time) |



### 2.2 multi函数

extend_stat / es、percentiles / pct、percentile_ranks / pct_ranks

| 函数| 描述| 示例|
| --- | --- |--- |
| extend_stat(X) es(X) | 返回字段X的es扩展统计。 es将返回多个值，将返回如下字段：_es.X.count _es.X.min _es.X.max _es.X.avg _es.X.sum _es.X.sum_of_squares _es.X.variance _es.X.std_deviation  | 返回resp_len字段的es统计值： es(resp_len)|
| percentiles(X, Y1, Y2…​) pct(X, Y1, Y2…​) | 返回字段X的值排序后，百分位Y1, Y2所对应的字段值。 pct将返回多个值 字段命名方式如下: Y1对应的字段为_pct.X.Y1, Y2对应的字段为_pct.X.Y2 | 返回response_time在50%，75%, 95%分位的值：pct(response_time, 50, 75, 95) 将返回三个字段： _pct.response_time.50, _pct.response_time.75, _pct.response_time.95 |
| percentile_ranks(X, Y1, Y2…​) pct_ranks(X, Y1, Y2…​) | 返回Y1，Y2所对应的百分位, X： 数值类型字段, Y1，Y2： 为字段X对应的值，pct_ranks将返回多个值，字段命名方式如下， _pct_ranks.X.Y1 _pct_ranks.X.Y2 | 以下示例返回100， 200， 500在response_time字段中对应的百分位： pct_ranks(response_time, 100, 200, 500) 返回字段集合 _pct_ranks.response_time.100 _pct_ranks.response_time.200 _pct_ranks.response_time.500 |
|top(X, LIMIT)| 此函数统计字段X内最多出现的若干个值| 返回apache.status使用最多的三个值及其对应使用的次数：top(apache.status, 3) |
| values(X,[LIMIT]) | 返回字段X去重后的值。LIMIT默认值为100  | 以下示例返回数据中appname出现的前90个不重复的值：*|stats values(appname,90) | 




### 2.3 画图函数 

date_histogram / dhg、histogram / hg、range_bucket / rb、sparkline

| 函数| 描述| 示例|
| --- | --- |--- |
|date_histogram(X, INTERVAL)| 时间直方图统计| 把timestamp字段以1h分桶统计： dhg(timestamp, 1h) |
|histogram(X, INTERVAL) hg(X, INTERVAL)|  直方图统计。 字段X必须为整数类型 | 把apache.status以200分桶统计： hg(apache.status, 200)|
| range_bucket(RANGE_BUCKET, RANGE_BUCKET…​) rb(RANGE_BUCKET, RANGE_BUCKET…​) | X：为数值类型 RANGE_BUCKET：统计区间，表示为(start, end)。可以设置多个。 | 以下示例把apache.status以指定区间分桶统计： rb(apache.status,(100,200) , (200,300), (300,400)) |
|  sparkline(agg(X), INTERVAL) | 按照指定区间分桶，通过面积图展示每个分桶内统计数据。 agg：部分与stats有关的函数，支持所有的single函数 | 返回按1h分桶，按tag分类后，apache.resp_len的平均值对应的面积图： stats sparkline(avg(apache.resp_len), 1h) by tag  | 



### 2.4 其他

| 函数| 描述| 示例|
| --- | --- |--- |
| list(X,[LIMIT]) | 将字段X的值组合成列表返回。 LIMIT为值列表中值的个数上限，默认值为100 | 以下示例返回数据中appname出现的前90个值： *|stats list(appname,90) |
| signify(X,[LIMIT])| 返回指定字段中有趣或不寻常的字段值的集合，并按照重要性排序。 字段值的重要性取决于score的大小。 示例： 用户在文本中搜索“禽流感”时提示“H5N1”； 发现欺诈性医生诊断出的鞭伤伤害超过了他们的公平份额； 发现爆胎次数不成比例的轮胎制造商。在所有这些情况下，所选择的术语不仅仅是一组中最流行的术语。 它们是在前景和背景集之间测量的流行度发生显着变化的术语。 如果术语“H5N1”仅存在于 1000 万个文档索引中的 5 个文档中，但在构成用户搜索结果的 100 个文档中的 4 个中找到，这些文档很重要并且可能与他们的搜索非常相关。 5/10,000,000 对 4/100 是一个很大的频率摆动。打分的原理是根据foregroundPercent(目标术语在前景集所占的百分比)与backgroundPercen(目标术语在背景集所占的百分比)计算得分，前景集：与查询直接匹配的搜索结果；背景集：从中检索它们的索引；目标术语：用户感兴趣的、重要的术语 重要术语聚合的任务是比较这些集合并找到最常与用户查询关联的术语。LIMIT默认值为10。|  返回目标字段appname对应的值中最重要的10个。此例中appname为前景集，query语句查询的内容为背景集，返回的结果为目标术语。*\|stats signify(appname,10)|



