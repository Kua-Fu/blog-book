# introduction

## 一、参考

>[Introduction to Kafka Security](https://developer.confluent.io/courses/security/intro/)

## 二、简介

Security is a primary consideration for any system design, and Apache Kafka® is no exception. Out of the box, Kafka has relatively little security enabled, so you need a basic familiarity with authentication, authorization, encryption, and audit logs in Kafka in order to securely put your system into production. This course has been created to quickly get you up to speed on what you need to know.

To secure your Apache Kafka-based system, you must formulate your overall strategy according to several factors: your internal corporate policy, the industry or regulatory requirements that govern your data processing capabilities, and finally the environment in which you plan to deploy your solution.

Of course, adding security often brings performance costs. For example, the CPU overhead of encrypting data can be significant when using a high throughput system like Kafka, up to 30% in some cases—even with the optimizations that Kafka uses to reduce this cost. On the other hand, you can't focus purely on performance, or you will probably wind up with a poorly protected system.

安全性是任何系统设计的主要考虑因素，Apache Kafka® 也不例外。Kafka默认情况下的安全性相对较低，因此您需要对Kafka中的认证、授权、加密和审计日志有基本的了解，以确保将系统安全地投入生产。本课程旨在快速让您了解所需的知识。

为了确保基于 Apache Kafka系统安全，您必须根据多个因素制定整体策略：包括内部企业政策、规管数据处理能力的行业或监管要求，以及计划部署解决方案的环境。


当然，增加安全性通常会带来性能损耗。例如，在使用像Kafka这样高吞吐量的系统时，加密数据的CPU开销可能相当大，有些情况下甚至可达30%，即使使用Kafka优化以降低这种成本。另一方面，不能完全专注于性能，否则可能会导致系统安全性不足。

## 三、Securing Data Streams in Kafka

保护 Kafka 中的数据流

A good way to conceptualize the parts that need securing is to consider the way that data (i.e., a message) flows through your Kafka system:

一个很好的理解需要保护的部分的方法是考虑数据（即消息）在您的 Kafka 系统中的流动方式：

![data flow](https://github.com/Kua-Fu/blog-book-images/blob/main/kafka/security/data-flows-kafka.jpg?raw=true)

The producer begins the message's journey by creating and sending it to the cluster. The message is received by the leader broker, which writes the message to its local log file. Then a follower broker fetches the message from the leader and similarly writes it to its local log file. The leader broker updates the partition state in ZooKeeper and finally, a consumer receives the message from the broker.

生产者通过创建并将消息发送到集群，开始消息的旅程。消息被 leader broker 接收，然后将消息写入其本地日志文件。接着，一个 follower broker 从 leader broker 获取消息，并将其写入本地日志文件。leader broker 更新 ZooKeeper 中的分区状态，最后，消费者从 broker 接收消息。


## 四、How Kafka Security Works

Each step of this data journey requires that a decision be made. For example, the broker authenticates the client to make sure the message is actually originating from the configured producer. Likewise, the producer verifies that it has a secure connection to the broker before sending any messages. Then before the leader broker writes to its log, it makes sure that the producer is authorized to write to the desired topic. This check also applies to the consumer – it must be authorized to read from the topic.


在这个数据传递的每个步骤中，都需要做出决策。例如，broker 会对客户端进行身份验证，以确保消息确实来自配置的生产者。同样，生产者在发送任何消息之前，会验证其与 broker 之间的安全连接。然后，在 leader broker 写入其日志之前，它会确保生产者有权写入所需的 topic 。这个检查同样也适用于消费者 - 它必须有权从 topic 中读取消息。


![auth](https://github.com/Kua-Fu/blog-book-images/blob/main/kafka/security/authorized-read-topic.jpg?raw=true)

## 五、Data Security and Encryption

数据安全和加密

Throughout the system, data should be encrypted so that it can't be read in transit or at rest. Additionally, all operations should be recorded in an audit log so that there is an audit trail in case something happens, or the behavior of the cluster needs to be verified.

在整个系统中，数据应该被加密，以确保在传输或静态状态下无法读取。此外，所有操作应该被记录在审计日志中，以便在发生问题或需要验证集群行为时存在审计跟踪。

![audit](https://github.com/Kua-Fu/blog-book-images/blob/main/kafka/security/audit-log.jpg?raw=true)

In conclusion, you need to choose the security measures for your system according to the corporate, industry, and environmental requirements particular to your scenario. The informational modules presented in this course detail the tools available to you with respect to Kafka and adjacent technologies, and the exercises teach you how to go about implementing them.

总之，您需要根据您的情况选择系统的安全措施，考虑到公司、行业和环境的要求。本课程中呈现的信息模块详细介绍了与Kafka和相关技术相关的可用工具，而练习则教会您如何实施这些工具。
