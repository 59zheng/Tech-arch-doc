### 基础概念
#### AMQP(高级消息队列协议)
AMQP（Advanced Message Queuing Protocol，高级消息队列协议）是一种面向消息中间件的协议。它定义了一种统一的消息格式，以及在不同系统之间传递消息的规范。AMQP 可以在不同的编程语言和操作系统之间使用，这使得它成为一个非常灵活和可扩展的协议。
AMQP 的核心思想是将消息发送者和接收者解耦，通过中间件来传递消息。消息发送者将消息发送到中间件，中间件再将消息传递给消息接收者。这种方式可以提高系统的可靠性和可扩展性，因为消息发送者和接收者不需要直接通信，它们之间的通信由中间件来处理。
**AMQP 协议具有以下特点**：

- 可靠性：AMQP 可以保证消息传递的可靠性，即使在出现网络故障或其他异常情况时也能够保证消息不会丢失。
- 灵活性：AMQP 可以在不同的编程语言和操作系统之间使用，这使得它非常灵活和可扩展。
- 安全性：AMQP 支持加密传输和身份验证，可以保证消息的安全性。
- 性能：AMQP 的性能非常好，可以处理大量的消息，而且具有较低的延迟。

总之，AMQP 是一种非常强大和灵活的消息传递协议，它可以帮助开发人员构建可靠、高性能、安全的消息中间件系统。
**定义的一些组件及基本概念**
（Exchange）和队列（Queue），用于路由和存储消息。生产者将消息发送到交换机，交换机根据预定义的路由规则将消息发送到一个或多个队列中。消费者可以订阅一个或多个队列，从队列中接收消息并进行处理。

AMQP 消息由消息头、消息体和属性组成。消息头包含了消息的元数据，例如消息的类型、优先级、过期时间等。消息体是消息的主要内容，可以是任何格式的数据。属性是一些可选的消息属性，例如消息的持久性、发送者和接收者的标识等。

#### 消息队列
##### 特点

- 先进先出 ： 队列（链表么），先进先出，消息队列的殊勋在入队时就基本确定，数据只是一条数据在使用中，
- 订阅： MQ的广播模式，类似于java的观察者模式，发布订阅是一种很高效的处理方式，如果不发生阻塞，基本可以当做同步操作。
- 持久化： 使 MQ 能像数据库一样存储核心数据。
- 分布式： 抗压，分流，负载。MQ的 定时是 高性能的中间件
##### 通信模式

- 点对点： 常见的通信模式，支持一对一，一对多，多对多，多对一等配置方式
- 发布订阅模式： 使发送者和接受者之间的耦合关系变得更加松散，发送者不必关心接受者的目的地址，接受者也不必关心系哦爱心的发送地址，基于消息的主题就行收发。
- 集群（Cluster） 群集类似于一个域(Domain)，群集内部的队列管理器之间通讯时，不需要两两之间建立消息通道， 而是采用群集(Cluster)通道与其它成员通讯，从而大大简化了系统配置

##### 使用场景：

- 应用耦合
- 消息异步
- 流量削峰

#### 消息存储方式
###### 概念
分布式队列因为有高可靠性的要求，所以数据要进行持久化存储，RocketMQ采用的是类似于 Kafka的文件存储机制，即直接用磁盘文件来保存消息，而不需要借助MySQL这一类索引工具。
目前的MQ中间件从存储模型来，分为需要持久化和不需要持久化的两种模型，现在大多数的是支 持持久化存储的，比如ActiveMQ、RabbitMQ、Kafka、RocketMQ等，而ZeroMQ却不需要支持持久化 存储而业务系统也大多需要MQ有持久存储的能力，这样可以大大增加系统的高可用性。
从存储方式和效率来看，文件系统高于KV存储，KV存储又高于关系型数据库，直接操作文件系统肯 定是最快的，但如果从是使用复杂性的角度出发直接操作文件系统是最复杂的，而关系型数据库的复杂 度最低，从可靠性的角度来分析，文件系统高于KV存储，KV存储高于关系型数据库
###### 存储介质类型和对比
常用的存储类型分为关系型数据库存储、分布式KV存储和文件系统存储
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678520694691-c356d931-3006-45b3-bcff-75508ed36b1f.png#averageHue=%23ebebeb&clientId=u34a81f95-4771-4&from=paste&height=576&id=u545043c5&originHeight=1152&originWidth=1610&originalType=binary&ratio=2&rotation=0&showTitle=false&size=430308&status=done&style=none&taskId=u0bfc8cde-c6ca-4ea9-8300-357235da499&title=&width=805)
存储效率： 文件系统>分布式 kv 存储 > 关系型数据库DB
开发难度和集成： 文件系统> 分布式 kv 存储 > 关系型数据 DB
###### 消息存储技术
目前的高性能磁盘，顺序写速度可以达到 600MB/s，足以满足一般网卡的传输速度，而磁盘随机 读写的速度只有约 100KB/s，与顺序写的性能相差了 6000 倍，故好的消息队列系统都会采用顺序读写的方式 （MB和Mb 区别 上述是 bit 为单位 后续 为 Byte 为单位，前者是后者 的*8 倍）
顺序读写 对硬盘速度的影响：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678520947780-88b02a43-4686-46be-82d4-d416732bc39b.png#averageHue=%23efefef&clientId=ua3b42098-aa71-4&from=paste&height=690&id=uc6da029d&originHeight=1380&originWidth=1660&originalType=binary&ratio=2&rotation=0&showTitle=false&size=457722&status=done&style=none&taskId=u4ffeec0c-efb6-42a1-8bb1-d53fa39e138&title=&width=830)


#### 零拷贝技术
Linux操作系统分为用户态和内核态，文件操作、网络操作需要涉及这两种形态的切换，免不了进 行数据复制，零拷贝技术就是减少拷贝的次数
**传统方式**
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678521190308-503ceea0-2ad3-4f8e-b8df-f1692962a2c7.png#averageHue=%23f3efe2&clientId=ua3b42098-aa71-4&from=paste&height=525&id=uba8b784c&originHeight=1050&originWidth=1738&originalType=binary&ratio=2&rotation=0&showTitle=false&size=410785&status=done&style=none&taskId=ufbc99967-16a2-401e-9e46-d2d6e4a5fa6&title=&width=869)
**上下文切换**
用户态和内核态的切换动作，就是上下文切换。 上下文切换会造成较大的消耗。
此时进行了四次 切换，文件拷贝了四次
###### 零拷贝实现
零拷贝不是消除拷贝，只是减少拷贝次数，实现技术方案有

- mmap+write
- sendfile
###### mmap+write
mmap() 系统调用函数会直接把内核缓冲区里的数据「映射」到用户空间，这样，操作系统内核与 用户空间就不需要再进行任何的数据拷贝操作。（共享的概念：将内核态的空间共享给用户态，这种方式不能在用户态进行数据的修改。。如果处理的话没法使用这种技术。只适用于消息中间件，不进行数据处理，直接固化的形式，系统限制 虚拟机内存映射技术，支撑的文件大小有限制，一般是1 -1.5g ） 3次数据复制，4次上下文切换次数 。。 
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678521535416-5e9c3e4d-b5ce-4420-a753-6b9cc2876ead.png#averageHue=%23f1eee0&clientId=ua3b42098-aa71-4&from=paste&height=517&id=u50a122c8&originHeight=1034&originWidth=1692&originalType=binary&ratio=2&rotation=0&showTitle=false&size=405403&status=done&style=none&taskId=ue55459bd-387e-495b-9285-7819e81b405&title=&width=846)
**复制过程**
通过使用 mmap() 来代替 read()， 可以减少一次数据拷贝的过程

- 应用进程调用了 mmap() 后，DMA 会把磁盘的数据拷贝到内核的缓冲区里，接着，应用进程跟操 作系统内核「共享」这个缓冲区;
- 应用进程再调用 write()，操作系统直接将内核缓冲区的数据拷贝到 socket 缓冲区中，这一切都发 生在内核态，由 CPU 来搬运数据;
- 最后，把内核的 socket 缓冲区里的数据，拷贝到网卡的缓冲区里，这个过程是由 DMA 搬运的 

但这还不是最理想的零拷贝，因为仍然需要通过 CPU 把内核缓冲区的数据拷贝到 socket 缓冲区里，而且仍然需要 4 次上下文切换，因为系统调用还是 2 次。 
**RocketMQ的MMAP**
这种机制在Java中是通过 NIO 包中的 MappedByteBuffer 实现的，RocketMQ充分利用该特性，也就是所谓的零拷贝技术，提高消息存盘和网络发送速度。
采用 MappedByteBuffer 这种内存映射的方式有几个限制:一次只能映射 1.5~2G 的文件至用户态的虚拟内存，这也是为何RocketMQ默认设置单个CommitLog日志数据文件为1G的原因。 零拷贝在Java NIO中提供了 mmap 和 sendfile 两种实现方式， mmap 适合比较小的文件，sendfile 适合传递比较大的文件。
###### sendfile
一个指令可以告诉 从哪读，写到哪。三次数据拷贝，两次上下文切换（内核函数 sendfile（）替换了write 和read 的动作。）
**操作系统支持**
在 Linux 内核版本 2.1 中，提供了一个专门发送文件的系统调用函数 sendfile()，函数形式如下:
```
#include <sys/socket.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```
前两个参数分别是目的端和源端的文件描述符，后面两个参数是源端的偏移量和复制数据的长度，返回值是实际复制数据的长度。
首先，它可以替代前面的 read() 和 write() 这两个系统调用，这样就可以减少一次系统调用，也就 减少了 2 次上下文切换的开销。
其次，该系统调用，可以直接把内核缓冲区里的数据拷贝到 socket 缓冲区里，不再拷贝到用户态， 这样就只有 2 次上下文切换，和 3 次数据拷贝。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678521795414-a895d488-beb1-4117-8e45-9f5cb3841414.png#averageHue=%23f2eee4&clientId=ua3b42098-aa71-4&from=paste&height=524&id=u9fc82a90&originHeight=1048&originWidth=1650&originalType=binary&ratio=2&rotation=0&showTitle=false&size=457779&status=done&style=none&taskId=u697feaf3-d1ad-4c51-be4e-c2c85097e76&title=&width=825)
###### SG-DMA（网卡技术，如果网卡支持，此时的senfile（）函数会在上述基础上又减少一次数据拷贝）
如果网卡支持 SG-DMA(The Scatter-Gather Direct Memory Access)技术(和普通的 DMA 有所不同)，我们可以进一步减少通过 CPU 把内核缓冲 区里的数据拷贝到 socket 缓冲区的过程。
可以在Linux 系统通过下面这个命令，查看网卡是否支持 scatter-gather 特性:
```
$ ethtool -k eth0 | grep scatter-gather
scatter-gather: on
```
于是，从 Linux 内核 2.4 版本开始起，对于支持网卡支持 SG-DMA 技术的情况下， sendfile() 系 统调用的过程发生了点变化，具体过程如下:

- 第一步，通过 DMA 将磁盘上的数据拷贝到内核缓冲区里; 
- 第二步，缓冲区描述符和数据长度传到 socket 缓冲区，这样网卡的 SG-DMA 控制器就可以直接将 内核缓存中的数据拷贝到网卡的缓冲区里，此过程不需要将数据从操作系统内核缓冲区拷贝到 socket 缓冲区中，这样就减少了一次数据拷贝;

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678522216632-48fb736d-6503-4ef9-b8b4-d8a2a0ca7c38.png#averageHue=%23efeddf&clientId=ua3b42098-aa71-4&from=paste&height=492&id=uc63f05eb&originHeight=984&originWidth=1740&originalType=binary&ratio=2&rotation=0&showTitle=false&size=445497&status=done&style=none&taskId=ub242df00-3507-4c8a-af7c-498468447d8&title=&width=870)


##### 
### 常见的消息队列及对比
##### ActiveMQ  最早的mq ，社区也不是很活跃，没有经历大规模吞吐量的验证，基本很少使用
##### RabbitMQ （推形式，消息存储 内存，异步刷盘；业务相关的功能提交较好，可以基于messageId 检索消息）
[Rabbit MQ](https://zhengy.yuque.com/hy8hzc/bcyh16/ng37mqkwug3xy5zo?view=doc_embed)
RabbitMQ是AMQP(高级消息队列 协议)的标准实现。支持多种客户端，如:Python、Ruby、.NET、Java、JMS、C、PHP、ActionScript、XMPP、 STOMP等，支持AJAX，持久化，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方 面表现不俗。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678675544023-ebf199a5-16c0-44df-b549-9fe72cb48330.png#averageHue=%23f5f6e1&clientId=uc53a64e5-e84c-4&from=paste&height=285&id=ua71dc4ac&originHeight=570&originWidth=1602&originalType=binary&ratio=2&rotation=0&showTitle=false&size=745784&status=done&style=none&taskId=ub8ada5c6-19b7-4f10-bb80-351fd345e5e&title=&width=801)
大家开始用 RabbitMQ，但是确实 erlang 语言阻止了大量的 Java 工程师去深入研究和掌控它，对 公司而言，几乎处于不可控的状态，但是确实人家是开源的，比较稳定的支持，活跃度也高，所以中小 型公司，技术实力较为一般，技术挑战不是特别高，用 RabbitMQ 是不错的选择，对自家技术没有过高 自信的话，可以使用RabbitMQ，人家有活跃的开源社区，绝对不会黄。
##### KafKa（拉形式，消息存储硬盘，性能高，）
[KafKa](https://zhengy.yuque.com/hy8hzc/bcyh16/sowac4xgztdcw8ms?view=doc_embed)
** 特性**

- 通过O(1)的磁盘数据结构提供消息的持久化，这种结构对于即使数以TB的消息存储也能够保持长时 间的稳定性能。(文件追加的方式写入数据，过期的数据定期删除) 
- 高吞吐量:即使是非常普通的硬件Kafka也可以支持每秒数百万的消息 
- 支持通过Kafka服务器和消费机集群来分区消息
- 支持Hadoop并行数据加载

如果是大数据领域的实时计算、日志采集等场景，用 Kafka 是业内标准的，绝对没问题，社区活跃 度很高，绝对不会黄，何况几乎是全世界这个领域的事实性规范。
**场景**

- 日志收集
- 消息系统
- 用户活动跟踪（记录web用户或者app用户的各种活动，基于消费者通过订阅来对这些数据进行监控分析）
- 运营指标（记录运营监控数据，包括手机各种分布式应用的数据，生产各种操作的集中反馈，如报警和报告）
- 流式处理（如 spark streaming 和storm）
##### RocketMQ（拉形式，支持消息ttl 过期，死信）
[RocketMq](https://zhengy.yuque.com/hy8hzc/bcyh16/knl6qo7lg3wd7b0o?view=doc_embed)
RocketMQ思路起源于Kafka，但并不是简单的复制，它对消息的可靠传输及事务性做了优化，目前 在阿里集团被广泛应用于交易、充值、流计算、消息推送、日志流式处理、binglog分发等场景，支撑了 阿里多次双十一活动。
因为是阿里内部从实践到产品的产物，因此里面很多接口、api并不是很普遍适用，可靠性毋庸置 疑，而且与Kafka一脉相承(甚至更优)，性能强劲，支持海量堆积。
现在确实越来越多的公司会去用 RocketMQ，确实很不错，毕竟是阿里出品，但社区可能有突然黄 掉的风险(目前 RocketMQ 已捐给 Apache，但 GitHub 上的活跃度其实不算高)对自己公司技术实力 有绝对自信的，推荐用 RocketMQ，否则回去老老实实用 RabbitMQ 吧，大型公司，基础架构研发实力 较强，用 RocketMQ 是很好的选择。




##### ![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678676196822-d8d3c8d1-69cb-40ee-98a6-a06b34feef01.png#averageHue=%23f3f3f3&clientId=uc53a64e5-e84c-4&from=paste&height=756&id=u874b9af3&originHeight=1512&originWidth=1626&originalType=binary&ratio=2&rotation=0&showTitle=false&size=396080&status=done&style=none&taskId=u37e56361-04e6-4c86-9311-4aa7b902c2f&title=&width=813)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678676220577-15afc4a2-17e7-4a2a-99a3-ae3534a36299.png#averageHue=%23ececec&clientId=uc53a64e5-e84c-4&from=paste&height=557&id=ucda7c1c6&originHeight=1114&originWidth=1616&originalType=binary&ratio=2&rotation=0&showTitle=false&size=474597&status=done&style=none&taskId=u2d328ea1-25fa-4419-afbc-cfd37d631b1&title=&width=808)
