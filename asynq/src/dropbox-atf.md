# Dropbox ATF

## 参考

> [How we designed Dropbox ATF: an async task framework](https://dropbox.tech/infrastructure/asynchronous-task-scheduling-at-dropbox)

> [Dropbox gRPC](https://dropbox.tech/infrastructure/courier-dropbox-migration-to-grpc)

> [Dropbox edgeStore](https://dropbox.tech/infrastructure/reintroducing-edgestore)

> [Amazon Simple Queue Service](https://dropbox.tech/infrastructure/asynchronous-task-scheduling-at-dropbox)

How we designed Dropbox ATF: an async task framework

我们如何设计一个异步任务框架

I joined Dropbox not long after graduating with a Master’s degree in computer science. Aside from an internship, this was my first big-league engineering job. My team had already begun designing a critical internal service that most of our software would use: It would handle asynchronous computing requests behind the scenes, powering everything from dragging a file into a Dropbox folder to scheduling a marketing campaign.

This Asynchronous Task Framework (ATF) would replace multiple bespoke async systems used by different engineering teams. It would reduce redundant development, incompatibilities, and reliance on legacy software. There were no open-source projects or buy-not-build solutions that worked well for our use case and scale, so we had to create our own. ATF is both an important and interesting challenge, though, so we were happy to design, build and deploy our own in-house service.、

ATF not only had to work well, it had to work well at scale: It would be a foundational building block of Dropbox infrastructure. It would need to handle 10,000 async tasks per second from the start, and be architected for future growth.  It would need to support nearly 100 unique async task types from the start, again with room to grow. There were at least two dozen engineering teams that would want to use it for entirely different parts of our codebase, for many products and services. 

As any engineer would, we Googled to see what other companies with mega-scale services had done to handle async tasks. We were disappointed to find little material published by engineers who built supersized async services.

Now that ATF is deployed and currently serving 9,000 async tasks scheduled per second and in use by 28 engineering teams internally, we’re glad to fill that information gap

我在硕士毕业不久后，就加入了Dropbox公司。除了实习，这是我的第一份大公司工作。我所在的团队，开始设计公司内部都会使用的关键内部组件：它将会在后台处理异步请求，为将文件移动到Dropbox文件夹或者准备一个促销活动等服务提供动力。

这个异步任务框架，将取代其他工程技术团队内部使用的定制异步系统。它将减少冗余开发、不兼容性和软件耦合性。没有任何开源项目能够很好的满足我们的用例和规模，所以，我们必须创建自己的异步任务框架。这是一个重要并且有趣的挑战，因此，我们很高兴设计、构建、部署自己的内部服务。

异步任务框架，必须在异步任务规模变大时候，保证稳定性。它将是Dropbox基础组件。一开始，就必须考虑到每秒1万个异步任务处理能力，并且还需要考虑到未来扩展。同样，一开始，就需要考虑有100多种不同类型的异步任务，并且也要考虑未来扩展。至少有24个团队，希望使用我们开发的异步任务框架，应用于更多的产品和服务。

就像任何工程师一样，我们google搜索，查看其他拥有大规模服务的公司，在处理异步任务方面做了什么。但是，我们很失望的发现，有关如何构建超大型异步服务的文章非常少。

现在，我们的异步任务框架，已经可以处理每秒9000多个任务，并且公司内部有28个团队在使用。所以，我们将很高兴填补大规模异步任务框架的空白。下面，我们将详细记录Dropbox的异步任务框架设计工作，为广大工程师设计自己的异步任务框架，提供参考和指南。

## 一、Introduction

介绍

Scheduling asynchronous tasks on-demand is a critical capability that powers many features and internal platforms at Dropbox. Async Task Framework (ATF) is the infrastructural system that supports this capability at Dropbox through a callback-based architecture. ATF enables developers to define callbacks, and schedule tasks that execute against these pre-defined callbacks.

Since its introduction over a year ago, ATF has gone on to become an important building block in the Dropbox infrastructure, used by nearly 30 internal teams across our codebase. It currently supports 100+ use cases which require either immediate or delayed task scheduling. 

按照需求，异步调度任务，是Dropbox许多功能和内部平台的关键功能。ATF是基于回调的体系结构创建的。ATF使得开发人员可以定义回调函数，当执行异步任务时候，会执行这些预定义的回调函数。

自从推出ATF一年多以来，它已经成为基础设施的重要组成部分。我们的代码库已经有近30个团队在调用。它目前支持100多个需要立即执行或者延迟调度的用例。

## 二、Glossary

名词解释

Some basic terms repeatedly used in this post, defined as used in the context of this discussion.

* Lambda: A callback implementing business logic.

* Task: Unit of execution of a lambda. Each asynchronous job scheduled with ATF is a task.

* Collection: A labeled subset of tasks belonging to a lambda. If send email is implemented as a lambda, then password reset email and marketing email would be collections.

*  Priority: Labels defining priority of execution of tasks within a lambda. 

	
下面是本文将重复使用的一些术语。

* Lambda 实现业务逻辑的回调

* task lambda执行单位，使用ATF调度的每个异步任务都是一个task

* collection, 属于一个lambda执行的一组标签任务，例如：

	如果发送邮件是一个lambda回调，那么，密码重置和市场推广，都会发送邮件，它们是一组 collection.
	
* priority, 优先级，定义同一个lambda回调中的任务优先级。

## 三、Features

特性

### 3.1 Task scheduling

任务调度

Clients can schedule tasks to execute at a specified time. Tasks can be scheduled for immediate execution, or delayed to fit the use case.

客户端可以指定任务在指定时间执行，异步任务可以安排为立即执行，也可以延迟执行。

### 3.2 Priority based execution

基于优先级的执行

Tasks should be associated with a priority. Tasks with higher priority should get executed before tasks with a lower priority once they are ready for execution.

每个任务都和优先级关联，优先级高的任务首先执行，优先级低的任务后执行。

### 3.3 Task gating

任务控制

ATF enables the the gating of tasks based on lambda, or a subset of tasks on a lambda based on collection. Tasks can be gated to be completely dropped or paused until a suitable time for execution.

支持（1） 基于lambda回调的任务控制；（2）基于collection的任务控制

异步任务可以被控制为被删除，或者暂停到合适的时间。

### 3.4 Track task status

追踪任务执行状态

Clients can query the status of a scheduled task.

客户端可以查看任务执行状态。

## 四、System guarantees

稳定性保证

### 4.1 At-least once task execution

至少执行一次

The ATF system guarantees that a task is executed at least once after being scheduled. Execution is said to be complete once the user-defined callback signals task completion to the ATF system.

ATF系统，保证异步任务至少会执行一次，一旦用户定义的回调函数，发送回任务执行完成信号后，该异步任务状态变更为完成状态。

### 4.2 No concurrent task execution

没有并发任务执行

The ATF system guarantees that at most one instance of a task will be actively executing at any given in point.This helps users write their callbacks without designing for concurrent execution of the same task from different locations.

ATF系统保证在任何时间点，异步任务最多只有一个实例在运行中。当用户编写回调函数，不需要考虑同一个任务会被并发调用。

### 4.3 Isolation

隔离性

Tasks in a given lambda are isolated from the tasks in other lambdas. This isolation spans across several dimensions, including worker capacity for task execution and resource use for task scheduling. Tasks on the same lambda but different priority levels are also isolated in their resource use for task scheduling.

不同lambda回调中的异步任务相互隔离。这种隔离跨越多个纬度，包括任务执行的worker数量，任务调度的资源使用。

相同的lambda中，优先级不同的异步任务，执行时候的资源使用，也会隔离。

### 4.4 Delivery latency

传递延迟

95% of tasks begin execution within five seconds from their scheduled execution time.

95% 的任务在计划执行时间的5秒钟内，会执行。

### 4.5 High availability for task scheduling

任务调度的高可用性

The ATF service is 99.9% available to accept task scheduling requests from any client.

ATF 服务能保证服务可用性为 99.9%

## 五、Lambda requirements

回调需求

Following are some restrictions we place on the callback logic (lambda):

下面是回调函数的一些限制

### 5.1 Idempotence

幂等

A single task on a lambda can be executed multiple times within the ATF system. Developers should ensure that their lambda logic and correctness of task execution in clients are not affected by this.

一个异步任务对应的lambda回调可以多次执行。开发人员需要确保，lambda逻辑和客户端任务多次执行，不会影响准确性。

### 5.2 Resiliency

弹性

Worker processes which execute tasks might die at any point during task execution.ATF retries abruptly interrupted tasks, which could also be retried on different hosts.  Lambda owners must design their lambdas such that retries on different hosts do not affect lambda correctness.

执行异步任务的worker可能在任何时间点挂掉。ATF将会重试这些失败的异步任务，当然，可能retry发生在其他主机。Lambda 设计者，需要考虑在不同主机上执行相同的异步任务，不会影响准确性。

### 5.3 Terminal state handling

状态处理

ATF retries tasks until they are signaled to be complete from the lambda logic. 

ATF重新发送异步任务，一直到它接收到任务完成的信号。客户端可以将任务标记为，完成、终止、重试。lambda设计者需要考虑，合适时候发送任务完成信号，需要避免错误行为，例如：无限次重试。这一点非常重要。


## 六、Architecture

架构图

![dropbox atf](https://github.com/Kua-Fu/blog-book-images/blob/main/atf/dropbox-atf.png?raw=true)

In this section, we describe the high-level architecture of ATF and give brief description of its different components. Dropbox uses gRPC for remote calls and our in-house Edgestore to store tasks.

ATF consists of the following components: 

* Frontend

* Task Store

* Store Consumer

* Queue

* Controller

* Executor

* Heartbeat and Status Controller (HSC)


本小节，将描述ATF架构，以及架构中的具体组件。Dropbox 使用gRPC进行远程调度，使用 Edgestore 用于任务存储。

ATF具体下列组件

### 6.1 Frontend

前端

This is the service that schedules requests via an RPC interface. The frontend accepts RPC requests from clients and schedules tasks by interacting with ATF’s task store described below.

这是发送异步任务请求的客户端，但同时也是，接收RPC接口发送请求的服务端。前端，接收用户发送的RPC请求，然后，构造异步任务，并且通过和任务存储系统交互，调度异步任务。

### 6.2 Task Store

任务保存

ATF tasks are stored in and triggered from the task store. The task store could be any generic data store with indexed querying capability. In ATF’s case, We use our in-house metadata store Edgestore to power the task store. More details can be found in the Data Model section below.

ATF的异步任务保存在数据库中。保存任务的数据库，可以是任何具有查询能力的数据库。在我们公司，使用内部自研的Edgestore数据库保存任务。

### 6.3 Store Consumer

数据库消费者

The Store Consumer is a service that periodically polls the task store to find tasks that are ready for execution and pushes them onto the right queues, as described in the queue section below. These could be tasks that are newly ready for execution, or older tasks that are ready for execution again because they either failed in a retriable way on execution, or were dropped elsewhere within the ATF system. 

Below is a simple walkthrough of the Store Consumer’s function: 

```
repeat every second:

  1. poll tasks ready for execution from task store
  
  2. push tasks onto the right queues
  
  3. update task statuses
  
```

The Store Consumer polls tasks that failed in earlier execution attempts. This helps with the at-least-once guarantee that the ATF system provides. More details on how the Store Consumer polls new and previously failed tasks is presented in the Lifecycle of a task section below.


数据库消费者提供服务，定期轮询数据库，查看准备好执行的任务，将它们放入正确的队列中。这些异步任务，可能是，最新准备好执行的任务，也可能是旧任务，这些任务，在之前执行过程中失败，或者其他方式被重新执行。

下面是数据库消费者的简单逻辑

```
每秒执行

1. 从数据库中查询准备好的任务

2. 将任务推送到正确的队列中

3. 更新任务状态
```

数据库消费者会轮询先前执行失败的任务。这是保证ATF至少执行一次的特性的设计。关于，消费者，如何轮询新任务、失败的旧任务，可以继续阅读下面内容。

### 6.4 Queue

ATF uses AWS Simple Queue Service (SQS) to queue tasks internally. These queues act as a buffer between the Store Consumer and Controllers (described below). Each <lambda, priority>  pair gets a dedicated SQS queue. The total number of SQS queues used by ATF is #lambdas x #priorities.

ATF使用 aws的简单队列服务，作为内部任务队列。这些队列充当数据库消费者和控制器之间的缓存区。每个  <lambda, priority> 都有一个专有的队列。ATF使用的aws SQS队列数量为 

\\[ \sum_{i=0}^n  lambda_i \ast priority_i  \\]

### 6.5 Controller

控制器

Worker hosts are physical hosts dedicated for task execution. Each worker host has one controller process responsible for polling tasks from SQS queues in a background thread, and then pushing them onto process local buffered queues. The Controller is only aware of the lambdas it is serving and thus polls only the limited set of necessary queues. 

The Controller serves tasks from its process local queue as a response to NextWork RPCs. This is the layer where execution level task prioritization occurs. The Controller has different process level queues for tasks of different priorities and can thus prioritize tasks in response to NextWork RPCs.

工作主机是专用于任务执行的物理主机。每个主机，都有一个控制器进程，该控制器进程负责在后台轮询来自aws SQS队列的任务，然后将 SQS队列 的任务，推送到本地的进程缓冲队列。控制器，只知道它正在服务的lambdas, 因此它只会轮询一部分 SQS 队列。

控制器将本地进程队列中的任务，作为对NextWork RPC的响应。这是执行任务优先级排序发生的层。控制层，对于不同优先级的任务，具有不同的进程队列，因此，可以响应Nextwork RPC不同的优先级队列。

### 6.6 Executor
 
执行器

The Executor is a process with multiple threads, responsible for the actual task execution. Each thread within an Executor process follows this simple loop:

```python

while True:
  w = get_next_work()
  do_work(w)
  
```

Each worker host has a single Controller process and multiple executor processes. Both the Controller and Executors work in a “pull” model, in which active loops continuously long-poll for new work to be done.

执行器是一个具有多个线程的进程，负责实际任务的执行。执行器中的每个线程都遵循着下面简单的循环

```python

while True:
  w = get_next_work()
  do_work(w)
  
```

每个工作主机，只有一个控制器进程，但是有多个执行器进程。控制器、执行器都是pull模式运行，在pull模式下，会不断的轮询，获取需要执行的任务。

### 6.7 Heartbeat and Status Controller (HSC)

心跳和状态控制器

The HSC serves RPCs for claiming a task for execution (ClaimTask), setting task status after execution (SetResults) and heartbeats during task execution (Heartbeat). ClaimTask requests originate from the Controllers in response to NextWork requests. Heartbeat and SetResults requests originate from executor processes during and after task execution. The HSC interacts with the task store to update the task status on the kind of request it receives.

心跳和状态控制器，HSC，服务于RPC，可以用于

（1）声明要执行的任务

（2）设置执行后的任务状态

（3）任务执行期间的心跳

声明要执行的任务，来源于响应Nextwork请求的控制器

设置任务状态、心跳，来源于任务执行期间和任务执行后的执行者进程

HSC和数据库交互，根据接收到的任务状态请求类型，更新任务状态。

## 七、Data model

数据模型

ATF uses our in-house metadata store, Edgestore, as a task store. Edgestore objects can be Entities or Associations (assoc), each of which can have user-defined attributes. Associations are used to represent relationships between entities. Edgestore supports indexing only on attributes of associations.

Based on this design, we have two kinds of ATF-related objects in Edgestore. The ATF association stores scheduling information, such as the next scheduled timestamp at which the Store Consumer should poll a given task (either for the first time or for a retry).The ATF entity stores all task related information that is used to track the task state and payload for task execution. We query on associations from the Store Consumer in a pull model to pick up tasks ready for execution.

ATF使用内部自研的edgestore数据库存储任务。edgestore数据库中对象，可以是实体或者关联，每个实体和关联都可以具有用户定义的属性。关联用于表示主体之间的关系。edgestore只支持关联属性的索引。

基于上面描述，在 edgestore中有两种和ATF有关的对象。

（1）ATF关联，存储调度信息，例如：数据库消费者应该轮询给定任务（第一次或者重试）的下一个调度时间戳。

（2）ATF实体，存储任务执行状态和任务其他信息。

在 pull模型中，数据库消费者查询ATF关联，获取准备执行的任务。

## 八、Lifecycle of a task

任务生命周期

1. Client performs a Schedule RPC call to Frontend with task information, including execution time. 

1. Frontend creates Edgestore entity and assoc for the task. 

1. When it is time to process the task, Store Consumer pulls the task from Edgestore and pushes it to a related SQS queue. 

1. Executor makes NextWork RPC call to Controller, which pulls tasks from the SQS queue, makes a ClaimTask RPC to the HSC and then returns the task to the Executor. 

1. Executor invokes the callback for the task. While processing, Executor performs Heartbeat RPC calls to Heartbeat and Status Controller (HSC). Once processing is done, Executor performs TaskStatus RPC call to HSC. 

1. Upon getting Heartbeat and TaskStatus RPC calls, HSC updates the Edgestore entity and assoc.

Every state update in the lifecycle of a task is accompanied by an update to the next trigger timestamp in the assoc. This ensures that the Store Consumer pulls the task again if there is no change in state of the task within the next trigger timestamp. This helps ATF achieve its at-least-once delivery guarantee by ensuring that no task is dropped.



1. 用户执行RPC 请求于前端，该请求包含异步任务信息（包括执行时间）

1. 前端构造 edgestore保存对象，实体和关联

1. 当需要处理任务时候，数据库消费者从 edgestore中获取任务，并将任务推送到相关的aws SQS队列。

1. 执行器向控制器，发送Nextwork RPC请求，控制器从SQS队列中提取任务，向 心跳和任务状态控制器HSC 发出 声明要执行的任务 RPC，然后，将任务发送给执行器。

1. 执行器调用任务的回调，在执行过程中，执行器对HSC发送心跳RPC，异步任务处理完成后，执行器对HSC发送任务执行状态RPC

1. 在获得心跳或者状态变更RPC后，HSC将会更新数据库中的实体和关联，两个对象。

异步任务生命周期的每一个状态更新，都伴随着对 edgestore中的任务对应的关联，中下一个触发时间戳的更新。这确保了，如果下一个触发器时间戳内任务的状态没有变化，则数据库消费者，将再次拉动任务。这有助于ATF实现至少一次执行的保证，确保没有异步任务丢失。

Following are the task entity and association states in ATF and their corresponding timestamp updates:


|Entity status | Assoc status | next trigger timestamp in Assoc| Comment|
|---|---|---|---|
|new | new | scheduled_timestamp of the task| Pick up new tasks that are ready. |
|enqueued| started| enqueued_timestamp + enqueue_timeout| Re-enqueue task if it has been in enqueued state for too long. This can happen if the queue loses data or the controller goes down after polling the queue and before the task is claimed.|
|claimed|started| claimed_timestamp + claim_timeout| Re-enqueue if task is claimed but never transfered to processing. This can happen if Controller is down after claiming a task. Task status is changed to enqueued after re-enqueue.|
|processing|started| heartbeat_timestamp + heartbeat_timeout| Re-enqueue if task hasn’t sent heartbeat for too long. This can happen if Executor is down. Task status is changed to enqueued after re-enqueue. |
|retriable failure|started|compute next_timestamp according to backoff logic|Exponential backoff for tasks with retriable failure.|
|success|completed|N/A||
|fatal_failure|completed| N/A| |


下面是任务主体/任务关联，状态变更和时间戳变化流程。

| 实体状态 |关联状态 | 关联中的下一次触发器时间戳| 备注|
|---|---|---|---|
|新建 | 新建 | 任务的执行时间戳| 选择出准备好的任务|
|排队| 启动 | 排队时间戳+排队超时| 如果任务在排队状态时间太长了，需要重新入队列；如果队列丢失数据，或者控制器在轮询队列之后和任务声明之前停机，则可能会发生排队时间过长|
|声明|启动 | 声明时间戳+声明超时| 如果任务已经声明，但是一直没有变更为处理中，那么会重新入队列; 如果控制器在声明任务后关闭，则可能会发生这种状况；重新排队后，任务状态更新为排队中|
|处理中|启动| 心跳时间+心跳超时| 如果任务长时间未发送心跳信号，需要重新排队，如果执行器关闭，可能会发生这种情况；重新排队后，任务状态更改为排队|
|失败|启动|根据退让逻辑，计算下一次时间戳| 退让逻辑，会指数级变化|
|成功|完成|||
|致命故障| 完成|||

Below is the state machine that defines task state transitions: 

下面是，任务状态变化的状态机。

![atf-2](https://github.com/Kua-Fu/blog-book-images/blob/main/atf/dropbox-atf-2.png?raw=true)
