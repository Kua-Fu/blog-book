# 华为云CSS生命周期管理

## 参考

>[如何更新CSS生命周期策略？](https://support.huaweicloud.com/css_faq/css_02_0119.html)
>
>[opendistro Index State Management](https://opendistro.github.io/for-elasticsearch-docs/docs/im/ism/)

## 一、简介

华为云CSS服务，对应的生命周期管理，使用`aws opendistro`对应的生命周期管理。

## 二、索引状态管理

If you analyze time-series data, you likely prioritize new data over old data. You might periodically perform certain operations on older indices, such as reducing replica count or deleting them.

Index State Management (ISM) is a plugin that lets you automate these periodic, administrative operations by triggering them based on changes in the index age, index size, or number of documents. Using the ISM plugin, you can define policies that automatically handle index rollovers or deletions to fit your use case.

如果分析时间序列数据，我们往往更乐于分析最新的数据，相比较于之前的老数据。你可能会对旧数据进行某些操作，例如: 缩少副本、删除过期数据。

ISM是一个插件，它允许你根据索引时间、索引大小、索引文档数量，去触发一些预先定义的管理操作，从而实现这些操作的自动化。通过ISM, 可以根据自己实际使用场景，定义索引滚动、删除策略。

For example, you can define a policy that moves your index into a read_only state after 30 days and then deletes it after a set period of 90 days. You can also set up the policy to send you a notification message when the index is deleted.

You might want to perform an index rollover after a certain amount of time or run a force_merge operation on an index during off-peak hours to improve search performance during peak hours.

例如：你可以定义一个策略，在写入数据30天后，将索引设置为只读状态，然后，在经过90天后，删除该索引。你还可以设置策略，在删除索引的时候，发送一个通知信息。

你可能希望在一定的时间后，滚动索引，或者在非写入高峰期，对索引执行`force_merge`操作，从而提高索引搜索性能。

## 三、基本概念

### 3.1 状态 states

A state is the description of the status that the managed index is currently in. A managed index can be in only one state at a time. Each state has associated actions that are executed sequentially on entering a state and transitions that are checked after all the actions have been completed.

This table lists the parameters that you can define for a state.

| Field|Description|Type |Required|
|---|---|---|---|
|name| the name of state|string |yes|
|actions|The actions to execute after entering a state. For more information, see Actions.|nested list of objects|yes|
|transitions|The next states and the conditions required to transition to those states. If no transitions exist, the policy assumes that it’s complete and can now stop managing the index. For more information, see Transitions.|nested list of objects|yes|

状态是当前被托管索引所处状态的描述。托管的索引在某个时刻只能有一种状态，每个状态，都会在进入该状态时候，执行一系列关联操作，在执行关联操作后，还会有一系列的检查操作, 检查是否进入下一个状态。

下面的表格，包含了一个状态的定义:

|  字段名称   | 描述  | 字段类型 | 是否必须|
|  ----  | ----  | --- | ---|
| name  | 状态名称 | string|是 |
| actions | 进入状态后，需要执行的操作 | nested list of objects| 是|
|transitions|接下来的状态，已经到达下一个状态的条件，如果该字段为空值，则表示该状态是索引最终状态，索引到达该状态不会进行索引管理| nested list of objects | 是|

### 3.2 操作 actions 

Actions are the steps that the policy sequentially executes on entering a specific state.

They are executed in the order in which they are defined.

This table lists the parameters that you can define for an action.

|Parameter|	Description|	Type	|Required|	Default|
| ---| ---| ---| ---| ---|
|timeout|	The timeout period for the action. Accepts time units for minutes, hours, and days.|	time unit|	No|	-|
|retry|	The retry configuration for the action.|	object|	No|	Specific to action|

The retry operation has the following parameters:

|Parameter	|Description|	Type|	Required|	Default|
|---|---|---|---|---|
|count|	The number of retry counts.|	number|	Yes|	-|
|backoff|	The backoff policy type to use when retrying.|	string|	No|	Exponential|
|delay|	The time to wait between retries. Accepts time units for minutes, hours, and days.|	time unit|	No|	1 minute|

操作是索引进入某个状态后，需要运行的一系列操作。

操作需要按照顺序依次执行。下面的表格，包含了一个操作的定义：


The following example action has a timeout period of one hour. The policy retries this action three times with an exponential backoff policy, with a delay of 10 minutes between each retry:

下面示例操作的超时时间为1个小时，该操作如果失败，会使用指数策略重试机制，每次重试的延迟为10分钟

```json

"actions": {
  "timeout": "1h",
  "retry": {
    "count": 3,
    "backoff": "exponential",
    "delay": "10m"
  }
}

```

### 3.3 转换 transitions

Transitions define the conditions that need to be met for a state to change. After all actions in the current state are completed, the policy starts checking the conditions for transitions.

Transitions are evaluated in the order in which they are defined. For example, if the conditions for the first transition are met, then this transition takes place and the rest of the transitions are dismissed.

转换定义了状态修改所需要满足的条件，当前状态下所有的 `actions`都执行完成后，进入到转换条件的检查。

转换按照定义的顺序进行检查，例如：满足了定义中的第一个条件，则转换会进行；会忽略其他的条件。

If you don’t specify any conditions in a transition and leave it empty, then it’s assumed to be the equivalent of always true. This means that the policy transitions the index to this state the moment it checks.

This table lists the parameters you can define for transitions.

如果定义中没有指定任何条件，则等同于`always true`，在索引检查时候，会进行状态转换。

下面的表格，定义了转换:

|Parameter|	Description|	Type|	Required|
|---|---|---|---|
|state_name	|The name of the state to transition to if the conditions are met.	|string	|Yes|
|conditions|	List the conditions for the transition.	|list	|Yes|

The conditions object has the following parameters:

|Parameter	Description	Type	Required
min_index_age	The minimum age of the index required to transition.	string	No
min_doc_count	The minimum document count of the index required to transition.	number	No
min_size	The minimum size of the total primary shard storage (not counting replicas). For example, if you set min_size to 100 GiB and your index has 5 primary shards and 5 replica shards of 20 GiB each, the total size of all primary shards is 100 GiB, so your index is transitioned to the next state.	string	No
cron	The cron job that triggers the transition if no other transition happens first.	object	No
cron.cron.expression	The cron expression that triggers the transition.	string	Yes
cron.cron.timezone	The timezone that triggers the transition.	string	Yes


### 3.4 策略 policies

Policies are JSON documents that define the following:

* The states that an index can be in, including the default state for new indices. For example, you might name your states “hot,” “warm,” “delete,” and so on. For more information, see States.

* Any actions that you want the plugin to take when an index enters a state, such as performing a rollover. For more information, see Actions.

* The conditions that must be met for an index to move into a new state, known as transitions. For example, if an index is more than eight weeks old, you might want to move it to the “delete” state. For more information, see Transitions.

In other words, a policy defines the states that an index can be in, the actions to perform when in a state, and the conditions that must be met to transition between states.

策略是一个JSON文档，里面定义了:

* 状态，策略中需要指定状态，例如：索引创建后的默认状态，状态的可选值有 `hot/warm/cold`等

* 操作，当索引进入到新状态后，执行的一系列操作，例如：滚动索引等

* 转换，索引进入新状态需要满足的条件。例如：一个索引，超过8周，可能需要变更为 delete 状态。


