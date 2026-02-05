# 观测云兼容阿里云sls服务讨论点

## 1. 时间节点

9月30日之前，完成sls接入与查询？

## 2. 是否需要接入时序数据？

不需要

## 3. 数据映射关系


### 3.1 基本概念

* 项目（Project）是日志服务的资源管理单元，是进行多用户隔离与访问控制的主要边界。

  您在1个阿里云账号下最多可创建50个Project。

  Project中包含Logstore、MetricStore和机器组等资源，同时它也是您访问日志服务资源的入口。建议使用不同的Project管理不同的应用、产品或项目中的数据。具体说明如下：
  组织、管理不同的Logstore或MetricStore。在实际使用中，您可能需要使用日志服务采集及存储不同项目、产品或者环境的日志。您可以把不同项目、产品或者环境中的日志分类管理在不同Project中，便于后续的日志消费、导出或者分析。
  用于访问控制隔离。您可以为RAM用户授予指定Project的操作权限。
  提供日志服务资源的访问入口。日志服务为每个Project配置一个独立的访问入口。该访问入口支持通过网络写入、读取及管理日志。关于访问入口的更多信息，请参见服务入口。

* 每个Logstore隶属于一个Project，每个Project中可创建多个Logstore。

  您在1个Project中最多可创建200个Logstore。

  您可以根据实际需求在目标Project中创建多个Logstore，一般是为同个应用中不同类型的日志创建独立的Logstore。例如您要采集App A所涉及的操作日志（operation_log）、应用程序日志（application_log）以及访问日志（access_log），您可以创建一个名为app-a的Project，并在该Project下创建名为operation_log、application_log和access_log的Logstore，用于分别存储操作日志、应用程序日志和访问日志。

   您在执行写入日志、查询和分析日志、加工日志、消费日志、投递日志等操作时，都需要指定Logstore。具体说明如下：

   以Logstore为采集单元，采集日志。
   以Logstore为存储单元，存储日志以及执行加工、消费、投递等操作。
   在Logstore中建立索引，用于查询和分析日志。
   

### 3.2 用户已经使用sls服务，只使用观测云查询

![sls](https://github.com/Kua-Fu/blog-book-images/blob/main/sls1.png?raw=true)

### 3.3 用户创建project，由观测云写入数据

![sls2](https://github.com/Kua-Fu/blog-book-images/blob/main/sls2.png?raw=true)

## 4. 下面是观测云写入数据，需要考虑的问题

### 4.1 是否需要考虑智能冷热分层存储？

[智能冷热分层存储](https://help.aliyun.com/document_detail/308645.htm?spm=a2c4g.11186623.0.0.50d229db8eoWI2#concept-2092727)

### 4.2 logstore 类型选择？

Standard和Query两种类型， 应该使用Standard类型，因为query类型无法分析

### 4.3 分片方式？

日志服务使用Shard读写数据。

一个Shard提供的写入能力为5 MB/s、500次/s，读取能力为10 MB/s、100次/s。

每个Logstore中最多创建10个Shard，每个Project中最多创建200个Shard

我们使用自动分裂Shard方式 （打开自动分裂Shard开关后，如果您写入的数据量超过已有Shard服务能力，日志服务会自动根据数据量增加Shard数量）

需要考虑是否会超过限制？

### 4.4 倒排索引配置?

![sls3](https://github.com/Kua-Fu/blog-book-images/blob/main/sls3.png?raw=true)

[配置索引](https://help.aliyun.com/document_detail/90732.htm?spm=a2c4g.11186623.0.0.39ddad12OheaDL#task-jqz-v55-cfb)

需要我们代码控制索引更新，没有自动更新功能

### 4.5 日志聚类功能？

开启日志聚类功能后，索引总量会增加原始日志大小的10%。例如原始数据为100 GB/天，开启该功能后，索引总量增加10 GB。

感觉我们应该不需要



