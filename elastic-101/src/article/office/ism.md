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

操作是



### 3.3 策略 policies

Policies are JSON documents that define the following:

* The states that an index can be in, including the default state for new indices. For example, you might name your states “hot,” “warm,” “delete,” and so on. For more information, see States.

* Any actions that you want the plugin to take when an index enters a state, such as performing a rollover. For more information, see Actions.

* The conditions that must be met for an index to move into a new state, known as transitions. For example, if an index is more than eight weeks old, you might want to move it to the “delete” state. For more information, see Transitions.

In other words, a policy defines the states that an index can be in, the actions to perform when in a state, and the conditions that must be met to transition between states.

策略是一个JSON文档，里面定义了:

* 状态，策略中需要指定状态，例如：索引创建后的默认状态，状态的可选值有 `hot/warm/cold`等

* 操作，当索引进入到新状态后，执行的一系列操作，例如：滚动索引等
