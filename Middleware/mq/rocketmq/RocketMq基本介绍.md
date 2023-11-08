
#### 基本介绍
RocketMQ作为一款纯java、分布式、队列模型的开源消息中间件，支持事务消息、顺序消息、批量消息、延时消息、消息回溯等（业务性比较关注）
##### 特点 （消费者是多线程的默认20个 ）

- 支持发布/订阅(Pub/Sub)和点对点(P2P)消息模型 
- 在一个队列中可靠的先进先出(FIFO)和严格的顺序传递 (RocketMQ可 以保证严格的消息顺序，而kafka无法保证)
- 支持拉(pull)和推(push)两种消息模式 (Push好理解，比如在消费者 端设置Listener回调;而Pull，控制权在于应用，即应用需要主动的调用拉 消息方法从Broker获取消息，这里面存在一个消费位置记录的问题(如果 不记录，会导致消息重复消费))
- 单一队列百万消息的堆积能力 (RocketMQ提供亿级消息的堆积能力，这不 是重点，重点是堆积了亿级的消息后，依然保持写入低延迟) 
- 支持多种消息协议，如 JMS、MQTT 等 分布式高可用的部署架构，满足至少一次消息传递语义(RocketMQ原生就 是支持分布式的，而ActiveMQ原生存在单点性)
- 提供 docker 镜像用于隔离测试和云集群部署 
- 提供配置、指标和监控等功能丰富的 Dashboard
##### 优势

- 支持事务性消息（消息发送和 DB 操作保持两方的最终一致性，RabbitMQ 和 Kafka 不支持）（分布式事务）
- 支持结合 RocketMQ 的多个系统之间数据最终一致性(多方事务，二方事 务是前提)
- 支持 18 个级别的延迟消息(Kafka 不支持)
- 支持指定次数和时间间隔的失败消息重发(Kafka 不支持，RabbitMQ 需要 手动确认)
- 支持 Broker 端 Tag 过滤，减少不必要的 传输(RabbitMQ 和 Kafka 不支 持) 同一个 topic 里边还可以有 tag 的概念，可以订阅指定topic 里边的指定tag 的消息，由 rocketMQ 支持
- 支持重复消费(RabbitMQ 不支持，Kafka 支持)主要是消息固化相关
##### 组件 （默认创建的 topic 是4个 分区）

- NameServer   是 RocketMQ的服务注册中心，（常见的 zk ，etdb 比较重，所以自己开发一个）需要先启动NameServer 再启动 Broker。 在数据上报的时候需要像所有节点都通报。

NameServer 被设计成几乎无状态的，可以横向扩展，节点间相互无通信，通过部署多台机器来标记自己十一个集群。 Broker 在启动的时候 注册到 NameServer 注册，Producer 在发送消息 前会根据 Topic 到 NameServer 获取到 Broker 的路由信息，Consumer 也会 定时获取 Topic 的路由信息

- Broker 消息存储中心  分为 Master 与 Slave 。一对多。主从不能切换 只能配置的时候默认（版本相关，最新版本支持，但是不太友好）  需要占用是三个端口： 远程端口 9011 索引 9010 存储的话 9012  
   1. 远程处理模块，Broker 入口，处理来自客户端的请求
   2. 客户端管理，管理客户端(包括消息生产者和消费者)，维护消费者的主题
订阅
   3. 存储服务，提供在物理硬盘上存储和查询消息的简单 API
   4. HA 服务，提供主从 Broker 间数据同步
   5. 索引服务，通过指定键为消息建立索引并提供快速消息查询

从物理结构上看 Broker 的集群部署方式有四种:单 Master 、多 Master 、多 Master 多 Slave(同步刷盘)、多 Master多 Slave(异步刷盘)。

   - 单 Master 这种方式一旦 Broker 重启或宕机会导致整个服务不可用，这种方式风险较大，所以显然不建议线上环境使用。
   - 多 Master ：所有消息服务器都是 Master ，没有 Slave ，这种方式优点是配置简单，单 个 Master 宕机或重启维护对应用无影响，缺点是单台机器宕机期间，该机器上 未被消费的消息在机器恢复之前不可订阅，消息实时性会受影响。
   - 多 Master 多 Slave(异步复制) ： 每个 Master 配置一个 Slave，所以有多对 Master-Slave，消息采用异步复 制方式，主备之间有毫秒级消息延迟，这种方式优点是消息丢失的非常少，且 消息实时性不会受影响，Master 宕机后消费者可以继续从 Slave 消费，中间的 过程对用户应用程序透明，不需要人工干预，性能同多 Master 方式几乎一样， 缺点是 Master 宕机时在磁盘损坏情况下会丢失极少量消息。
   - 多 Master 多 Slave(同步复制) ： 每个 Master 配置一个 Slave，所以有多对 Master-Slave ，消息采用同步 复制方式，主备都写成功才返回成功，这种方式优点是数据与服务都没有单点 问题，Master 宕机时消息无延迟，服务与数据的可用性非常高，缺点是性能相 对异步复制方式略低，发送消息的延迟会略高。
- producer （生产者 群组的概念） ：生产者向brokers发送由业务应用程序系统生成的消息，RocketMQ提供了 发送:同步、异步和单向(one-way)的多种模式（单向发送是指只负责发送消息而不等待服务器回应且没有回调函数触发， 适用于某些耗时非常短但对可靠性要求并不高的场景，例如日志收集。这点类 似kafka的发送并忘记。）
- 消费者(Consumer)消费者组 基于心跳检活（除非自动创建topic  不建议 topic 从创建到发现需要有30s的延迟感知时间）

消费者从brokers那里拉取信息并将其输入应用程序，在用户应用的角度，提供 了两种类型的消费者:

   - Pull:Pull型消费者主动地从brokers那里拉取信息，只要批量拉取到消 息，用户应用程序就会启动消费过程 
   - Push:Push型消费者封装消息的拉取、消费进度和维护内部的其他工作， 将一个在消息到达时执行的回调接口留给终端用户来实现。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678679804353-dfef9aa8-bba7-4160-97d5-62f67fe2694a.png#averageHue=%23afcdd9&clientId=u0369d726-8814-4&from=paste&height=508&id=uc7dbc75c&originHeight=1016&originWidth=1706&originalType=binary&ratio=2&rotation=0&showTitle=false&size=587707&status=done&style=none&taskId=ua5d64269-4fd3-4877-b5fd-25857e1751c&title=&width=853)

##### 顺序类型
###### 消息消费模式
消息消费模式有两种:集群消费(Clustering)和广播消费 (Broadcasting)默认情况下就是集群消费，该模式下一个消费者集群共同消费一个主题的多个队列，一个队列只会被一个消费者消费，如果某个消费者挂掉，分组内其
它消费者会接替挂掉的消费者继续消费而广播消费消息会发给消费者组中的每一个消费者进行消费
###### 消息顺序 （默认使用的是并行消费）
消息顺序(Message Order)有两种:顺序消费(Orderly)和并行消费(Concurrently)
顺序消费表示消息消费的顺序同生产者为每个消息队列发送的顺序一致，所以如果正在处理全局顺序是强制性的场景，需要确保使用的主题只有一个消息队列，并行消费不再保证消息顺序，消费的最大并行数量受每个消费者客户端指定的线程池限制。
###### 仅保证消息至少消费一次，即可能造成消息的重复消费，需要从业务上解决消息的幂等性。
###### 无序消息
无序消息也指普通的消息，Producer 只管发送消息，Consumer 只管接收消息，至于消息和消息 之间的顺序并没有保证。
Producer 依次发送 orderId 为 1、2、3 的消息
Consumer 接到的消息顺序有可能是 1、2、3，也有可能是 2、1、3 等情况，这就是普通消息。
###### 全局顺序
需要保证一个 topic 以及 broker 是一个（或者保证给指定的breoker ），队列也是1个
对于指定的一个 Topic，所有消息按照严格的先入先出(FIFO)的顺序进行发布和消费
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678499873872-103cdb4e-ad4f-4cd7-976a-1d732b21e922.png#averageHue=%237e8380&clientId=u86067c56-346e-4&from=paste&height=237&id=ufdaea4f9&originHeight=474&originWidth=1666&originalType=binary&ratio=2&rotation=0&showTitle=false&size=514862&status=done&style=none&taskId=u666d4000-f116-445d-a4f6-8c8c581ed6f&title=&width=833)
###### 局部顺序
单topic  多队列，保证单个队列内数据的有序
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678499896832-22630b24-f801-4b70-8163-518ca4e6ff7f.png#averageHue=%23eeeebf&clientId=u86067c56-346e-4&from=paste&height=303&id=u8dc60d7b&originHeight=606&originWidth=1660&originalType=binary&ratio=2&rotation=0&showTitle=false&size=287105&status=done&style=none&taskId=u04513e99-365b-47d7-a44d-983b6b19dd6&title=&width=830)
#### RocketMq 顺序消息
RocketMQ可以严格的保证消息有序，但这个顺序，不是全局顺序，只是分区(queue)顺序，要
全局顺序只能一个分区。 分区顺序 即 queue 级别的
可以使用同一个 topic 按照地区分类 分发到不同的 queue 中，保证指定分区的消息消费的顺序性。
  MessageQueueSelector  消息队列选择器：通过一定的策略，将其 放置在一个 queue队列中 ，然后 消费者 再采用一定的策略(一个线程独立处理一个 queue ,保证处理消息 的顺序性)，能够保证消费的顺序性
                  
#### 消息投递（消费）策略
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678501480837-1001dc5a-42a0-493b-b1b5-c7fe8f5c494c.png#averageHue=%23fbf7f2&clientId=u86067c56-346e-4&from=paste&height=530&id=ucf033712&originHeight=1060&originWidth=1710&originalType=binary&ratio=2&rotation=0&showTitle=false&size=495184&status=done&style=none&taskId=u78a648a6-8072-4e47-820f-2c5641759d8&title=&width=855)一个 Topic(消息主题) 可能对应多个 Borker 的  里边 多个 消息队列(MessgeQueue)
在底层实现上，为了提高MQ的可用性和灵活性，一个Topic在实际存储的过程中，采用了多队列的 方式，具体形式如上图所示，每个消息队列在使用中应当保证先入先出(FIFO,First In First Out)的方 式进行消费。
###### 生产者投递策略
轮询算法投递，，，顺序投递策略投递

**java 自带实现类**

1. 随机分配策略 SelectMessageQueueByRandom

默认情况下，采用了最简单的轮询算法，这种算法有个很好的特性就是，保证每一个 Queue队列 的 消息投递数量尽可能均匀。

2. 基于Hash分配 策略  SelectMessageQueueByHash

根据附加参数的Hash值，按照消 息队列列表的大小取余数，得到 消息队列的index

3. 基于机器机房位置分配策略  SelectMessageQueueByMachineRoom

开源的版本没有具体的实现，基本的目的应该是机器的就近原则分配

###### 消费者分配队列（消息消费模式）

- BROADCASTING :广播式消费，这种模式下，一个消息会被通知到每一个 消费者（需要保证多消费，对业务无感）
- CLUSTERING : 集群式消费，这种模式下，一个消息最多只会被投递到一个 消费者 上进行消费 模式如下:（kafka 就是这种 一个分组内只有一个实例可以消费）

对于使用了消费模式为 MessageModel.CLUSTERING 进行消费时，需要保证一个消息在整个集群中 只需要被消费一次，实际上，在RoketMQ底层，消息指定分配给消费者的实现，是通过queue队列分配 给消费者的方式完成的:也就是说，消息分配的单位是消息所在的queue队列。 默认是集群模式
将 queue 队列指定给特定的 消费者后， queue 队列 内的所有消息将会被指定到 消费者 进行消费。
AllocateMessageQueueStrategy  同组消费的视线方式 ：集群式消费方式
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678502431216-ebc92ae2-caf8-4b3a-b844-8416dfbda667.png#averageHue=%23f2f2f2&clientId=u86067c56-346e-4&from=paste&height=390&id=u6840eb7d&originHeight=780&originWidth=1748&originalType=binary&ratio=2&rotation=0&showTitle=false&size=247705&status=done&style=none&taskId=u01cffd82-a487-4403-8322-4134028cb47&title=&width=874)

- 平均分配：平均取模计算。 分配的个数按照消费者的顺序性。默认使用的方式
- 环形平均分配，基于消费者的顺序，一次在 queue队列 组成的环形图中分配  轮询分配的意思

使用方式
```
//创建一个消息消费者，并设置一个消息消费者组，并指定使用一致性hash算法的分配策略 
DefaultMQPushConsumer consumer = new 
DefaultMQPushConsumer(null,"rocket_test_consumer_group",null,
new AllocateMessageQueueConsistentHash());
.....
```
###### 
#### Rredis MQ 消息保障
##### 生产者保障
###### 消息发送保障

- 同步发送（阻塞等待 ack） 设置同步发送 一下两个配置是异步的也可能造成 消息丢失

还与以下两个配置有关
主broker 从 broker 是否同步，主从都写入成功了。 默认情况下是 异步复制 
主broker 消息最先在 内存中 内存刷 硬盘的过程是否同步？  默认情况下 是异步刷盘  
这种方式具有内部重试机制，即在主动声明本次消息发送失败之前，内部实现将重试一定次数，默 认为2次( DefaultMQProducer#getRetryTimesWhenSendFailed )，发送的结果存在同一个消息可 能被多次发送给broker，这里需要应用的开发者自己在消费端处理幂等性问题。

- 异步发送（发送消息后，不需要等待broker 响应）
- 单向发送（只管发，没有结果反馈（相当于 kafka 的发送并忘记 acks=0））

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678503521087-a98943ec-c3f6-4234-8f69-a54b3facc6da.png#averageHue=%23f0f0f0&clientId=u86067c56-346e-4&from=paste&height=208&id=u892abbd5&originHeight=416&originWidth=1708&originalType=binary&ratio=2&rotation=0&showTitle=false&size=122701&status=done&style=none&taskId=u2b37e2f1-24a3-4912-a3d0-15301964e36&title=&width=854)
当发送的消息不重要时，采用 one-way 方式，以提高吞吐量; 
当发送的消息很重要是，且对响应时间不敏感的时候采用 sync 方式; 
当发送的消息很重要，且对响应时间非常敏感的时候采用 async 方式;
###### 发送状态
发送消息时，将获得包含SendStatus的SendResult，首先，我们假设Message的 isWaitStoreMsgOK = true(默认为true)，如果没有抛出异常，我们将始终获得SEND_OK，以 下是每个状态的说明列表:

- FLUSH_DISK_TIMEOUT：如果设置了 FlushDiskType=SYNC_FLUSH (默认是 ASYNC_FLUSH)，并且 Broker 没有在 syncFlushTimeout (默认是 5 秒)设置的时间内完成刷盘，就会收到此状态码。
- FLUSH_SLAVE_TIMEOUT：如果设置为 SYNC_MASTER ，并且 slave Broker 没有在 syncFlushTimeout 设定时间内完成同 步，就会收到此状态码。
- SLAVE_NOT_AVAILABLE：如果设置为 SYNC_MASTER ，并没有配置 slave Broker，就会收到此状态码。
- SEND_OK：这个状态可以简单理解为，没有发生上面列出的三个问题状态就是SEND_OK，需要注意的是， SEND_OK 并不意味着可靠，如果想严格确保没有消息丢失，需要开启 SYNC_MASTER or SYNC_FLUSH。
###### 重试机制
同步和异步都有重试次数 都是两次

1. 如果是异步发送默认重试次数是两次，通过递归的方式进行重试
2. 对于同步而言，超时异常也是不会再去重试。
3. 同步发送重试是在一个for 循环里去重试，所以它是立即重试而不是隔一段时间去重试。
###### 禁止自动创建topic
自动创建 topic  （不建议）：autoCreateTopicEnable 设置为true 标识开启自动创建topic

1. 消息发送时如果根据topic没有获取到路由信息，则会根据默认的topic去获取，获取到路由信息后 选择一个队列进行发送，发送时报文会带上默认的topic以及默认的队列数量。
2.  消息到达broker后，broker检测没有topic的路由信息，则查找默认topic的路由信息，查到表示开 启了自动创建topic，则会根据消息内容中的默认的队列数量在本broker上创建topic，然后进行消 息存储。
3.  broker创建topic后并不会马上同步给namesrv，而是每30秒进行汇报一次，更新namesrv上的 topic路由信息，producer会每30s进行拉取一次topic的路由信息，更新完成后就可以正常发送消 息，更新之前一直都是按照默认的topic查找路由信息。

为什么不能开启：
上述 broker 中流程会有一个问题，就是在producer更新路由信息之前的这段时间，如果消息只发 送到了broker-a，则broker-b上不会创建这个topic的路由信息，broker互相之间不通信，当producer 更新之后，获取到的broker列表只有broker-a，就永远不会轮询到broker-b的队列(因为没有路由信 息)，所以我们生产通常关闭自动创建broker，而是采用手动创建的方式。

###### 发送端规避
重试的时候选择的 broker 尽量不选择同一个，无序的消息才可以规避
RocketMQ 提供了两种规避策略，该参数由 sendLatencyFaultEnable 控制，用户可干预，表示是否开启延迟规避机制，默认为不开启。(DefaultMQProducer中设置这两个参数)

- sendLatencyFaultEnable 设置为 false:默认值，不开启，延迟规避策略只在重试时生效，例如在 一次消息发送过程中如果遇到消息发送失败，规避 broekr-a，但是在下一次消息发送时，即再次调 用 DefaultMQProducer 的 send 方法发送消息时，还是会选择 broker-a 的消息进行发送，只要继 续发送失败后，重试时再次规避 broker-a。
- sendLatencyFaultEnable 设置为 true:开启延迟规避机制，一旦消息发送失败会将 broker-a “悲 观”地认为在接下来的一段时间内该 Broker 不可用，在为未来某一段时间内所有的客户端不会向该 Broker 发送消息，这个延迟时间就是通过 notAvailableDuration、latencyMax 共同计算的，就首 先先计算本次消息发送失败所耗的时延，然后对应 latencyMax 中哪个区间，即计算在 latencyMax 的下标，然后返回notAvailableDuration 同一个下标对应的延迟值。

**注意事项**
如果所有的 Broker 都触发了故障规避，并且 Broker 只是那一瞬间压力大，那岂不是明明存在可用 的 Broker，但经过你这样规避，反倒是没有 Broker 可用来，那岂不是更糟糕了?针对这个问题，会退 化到队列轮循机制，即不考虑故障规避这个因素，按自然顺序进行选择进行兜底。

##### 消费者保障
###### 幂等性
应用程序在使用RocketMQ进行消息消费时必须支持幂等消费，即同一个消息被消费多次和消费一 次的结果一样，这一点在使用RoketMQ或者分析RocketMQ源代码之前再怎么强调也不为过。
“至少一次送达”的消息交付策略，和消息重复消费是一对共生的因果关系，要做到不丢消息就无法 避免消息重复消费，原因很简单，试想一下这样的场景:客户端接收到消息并完成了消费，在消费确认 过程中发生了通讯错误，从Broker的角度是无法得知客户端是在接收消息过程中出错还是在消费确认过 程中出错，为了确保不丢消息，重发消息是唯一的选择。
###### 消息消费模式
从不同的维度划分，Consumer支持以下消费模式:

- 广播消费模式下，消息消费失败不会进行重试，消费进度保存在Consumer端; 

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678519577647-f367f850-a774-4aa2-aa0c-3a4725004d89.png#averageHue=%23f8f8f8&clientId=u86067c56-346e-4&from=paste&height=451&id=ub8267543&originHeight=902&originWidth=1710&originalType=binary&ratio=2&rotation=0&showTitle=false&size=400352&status=done&style=none&taskId=u0991b856-c948-457c-a7fc-d175c210da1&title=&width=855)

   - 广播消费模式下不支持顺序消息。
   - 广播消费模式下不支持重置消费位点。
   - 每条消息都需要被相同逻辑的多台机器处理。 消费进度在客户端维护，出现重复的概率稍大于集群模式。
   - 广播模式下，消息队列 RocketMQ 保证每条消息至少被每台客户端消费一次，但是并不会对消费 失败的消息进行失败重投，因此业务方需要关注消费失败的情况。 广播模式下，客户端每一次重启都会从最新消息消费。客户端在被停止期间发送至服务端的消息将 会被自动跳过， 请谨慎选择。 广播模式下，每条消息都会被大量的客户端重复处理，因此推荐尽可能使用集群模式。目前仅 Java 客户端支持广播模式。
   - 广播模式下服务端不维护消费进度，所以消息队列 RocketMQ 控制台不支持消息堆积查询、消息 堆积报警和订阅关系查询功能。
- 集群消费模式下，消息消费失败有机会进行重试，消费进度集中保存在Broker端。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678519526913-780029ee-3011-4ccd-8c8f-58c6883e6a84.png#averageHue=%23f8f8f8&clientId=u86067c56-346e-4&from=paste&height=374&id=u3274fcf8&originHeight=748&originWidth=1698&originalType=binary&ratio=2&rotation=0&showTitle=false&size=299319&status=done&style=none&taskId=u9cf8d505-fc0c-4f9a-952b-5211ac1c527&title=&width=849)

- 消费端集群化部署， 每条消息只需要被处理一次。
- 由于消费进度在服务端维护， 可靠性更高。 集群消费模式下，每一条消息都只会被分发到一台机器上处理。如果需要被集群下的每一台机器都 处理，请使用广播模式。 
- 集群消费模式下，不保证每一次失败重投的消息路由到同一台机器上，因此处理消息时不应该做任 何确定性假设。


###### 消息确认机制
consumer的每个实例是靠队列分配来决定如何消费消息的，那么消费进度具体是如何管理的，又 是如何保证消息成功消费的?(RocketMQ有保证消息肯定消费成功的特性，失败则重试)
为了保证数据不被丢失，RocketMQ支持消息确认机制，即ack，发送者为了保证消息肯定消费成 功，只有使用方明确表示消费成功，RocketMQ才会认为消息消费成功，中途断电，抛出异常等都不会 认为成功——即都会重新投递。
**消费异常的情况**
会将消息放到失败队列中，
为了保证消息是肯定被至少消费成功一次，RocketMQ会把这批消息重发回Broker(topic不是原 topic而是这个消费组的RETRY topic)，在延迟的某个时间点(默认是10秒，业务可设置)后，再次投 递到这个ConsumerGroup，而如果一直这样重复消费都持续失败到一定次数(默认16次)，就会投递 到DLQ死信队列，应用可以监控死信队列来做人工干预。
##### 消息重试机制
顺序消息的重试：对于顺序消息，当消费者消费消息失败后，消息队列RocketMQ版会自动不断地进行消息重试(每 次间隔时间为1秒)，这时，应用会出现消息消费被阻塞的情况，因此，建议您使用顺序消息时，务必保 证应用能够及时监控并处理消费失败的情况，避免阻塞现象的发生。
无序消息的重试：无序消息的重试只针对集群消费方式生效;广播方式不提供失败重试特性，即消费失败后，失败消息不再重试，继续消费新的消息。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678519908366-92dd0543-d131-41a3-a2f3-bd98486f1780.png#averageHue=%23f3f3f3&clientId=u86067c56-346e-4&from=paste&height=615&id=ucd395e9a&originHeight=1230&originWidth=1668&originalType=binary&ratio=2&rotation=0&showTitle=false&size=349871&status=done&style=none&taskId=u1b069e08-39f1-4aa7-8f36-0fe1d5e388f&title=&width=834)
##### 死信队列
在正常情况下无法被消费(超过最大重试次数)的消息称为死信消息(Dead-Letter Message)，存储死信消息的特殊队列就称为死信队列(Dead-Letter Queue)
当一条消息初次消费失败，消息队列 RocketMQ 会自动进行消息重试;达到最大重试次 数后，若 消费依然失败，则表明消费者在正常情况下无法正确地消费该消息，此时，消息队列 RocketMQ 不会立 刻将消息丢弃，而是将其发送到该消费者对应的特殊队列中。 在消息队列 RocketMQ 中，这种正常情况 下无法被消费的消息称为死信消息(Dead-Letter Message)，存储死信消息的特殊队列称为死信队列 (Dead-Letter Queue)。
**死信消息特性**
不会再被消费者正常消费
有效期与正常消息相同，均为 3 天，3 天后会被自动删除，故死信消息应在产生的 3 天内及时处理
**死信队列特性**
一个死信队列对应一个消费者组，而不是对应单个消费者实例
一个死信队列包含了对应的 Group ID 所产生的所有死信消息，不论该消息属于哪个 Topic 若一个 Group ID 没有产生过死信消息，则 RocketMQ 不会为其创建相应的死信队列

#### 高级特性
##### 消息存储设计
RocketMQ采用了单一的日志文件，即把同1台机器上面所有topic的所有queue的消息，存放在一
个文件里面，从而避免了随机的磁盘写入。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678524019621-6728c292-2a31-4aa9-8ff9-4afcfc4dd5db.png#averageHue=%23f6f6f4&clientId=u936e2e69-5ce4-4&from=paste&height=234&id=ue356750e&originHeight=468&originWidth=1598&originalType=binary&ratio=2&rotation=0&showTitle=false&size=392657&status=done&style=none&taskId=u048c07bd-7a27-4b64-b75f-64393a0cae4&title=&width=799)
所有消息都存在一个单一的CommitLog文件里面，然后有后台线程异步的同步到 ConsumeQueue，再由Consumer进行消费。
这里之所以可以用“异步线程”，也是因为消息队列天生就是用来“缓冲消息”的，只要消息到了 CommitLog，发送的消息也就不会丢，只要消息不丢，那就有了“充足的回旋余地”，用一个后台线程慢 慢同步ConsumeQueue，再由Consumer消费。
###### 消息存储结构
消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容，消息内容不是定长的
RocketMQ 采取一些机制，尽量向 CommitLog 中顺序写，但是随机读，单个文件大小默认1G ，可 通过在 broker 置文件中设置 mapedFileSizeCommitLog 属性来改变默认大小。
**CommitLog（顺序读写，快速响应）**
CommitLog是存储消息内容的存储主体，Producer发送的消息都会顺序写入CommitLog文件
**ConsumeQueue（类似于二级索引，基于二级索引定义一级索引（commitLog）去 ）**
consumequeue文件可以看成是基于topic的commitlog索引文件。
RocketMQ基于主题订阅模式实现消息的消费，消费者关心的是主题下的所有消息，但是由于不同 的主题的消息不连续的存储在commitlog文件中，如果只是检索该消息文件可想而知会有多慢，为了提 高效率，对应的主题的队列建立了索引文件，为了加快消息的检索和节省磁盘空间，每一个 consumequeue条目存储了消息的关键信息commitog文件中的偏移量、消息长度、tag的hashcode 值。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678524192056-d3a8a882-8fc9-4bbd-a99e-580f0b4148b7.png#averageHue=%23f8f8f8&clientId=u936e2e69-5ce4-4&from=paste&height=380&id=uba63669c&originHeight=760&originWidth=1550&originalType=binary&ratio=2&rotation=0&showTitle=false&size=231145&status=done&style=none&taskId=ud6725c49-7b2d-44ac-9b00-827aba06f09&title=&width=775)
**indexFile（messageId 就是基于 indexFile 文件检索可以在服务端检索消息）**
index 存的是索引文件，用于为生成的索引文件提供访问服务，这个文件用来加快消息查询的速 度，通过消息Key值查询消息真正的实体内容
**config**
config 文件夹中 存储着 Topic 和 Consumer 等相关信息，主题和消费者群组相关的信息就存在在 此。

- topics.json : topic 配置属性 subscriptionGroup.json :消息消费组配置信息。 
- delayOffset.json :延时消息队列拉取进度。 
- consumerOffset.json :集群消费模式消息消进度。
-  consumerFilter.json :主题消息过滤信息。

###### 消息进度管理
消息的存储是一直存在于CommitLog中的，而由于CommitLog是以文件为单位(而非消息)存在 的，CommitLog的设计是只允许顺序写的，且每个消息大小不定长，所以这决定了消息文件几乎不可能 按照消息为单位删除(否则性能会极具下降，逻辑也非常复杂)，所以消息被消费了，消息所占据的物 理空间并不会立刻被回收。
当新实例启动的时候，PushConsumer会拿到本消费组broker已经记录好的消费进度，如果这个消
费进度在Broker并没有存储起来，证明这个是一个全新的消费组，这时候客户端有几个策略可以选择:
```
CONSUME_FROM_LAST_OFFSET //默认策略，从该队列最尾开始消费，即跳过历史消息 
CONSUME_FROM_FIRST_OFFSET //从队列最开始开始消费，即历史消息(还储存在broker的)全部消费一 遍 
CONSUME_FROM_TIMESTAMP//从某个时间点开始消费，和setConsumeTimestamp()配合使用，默认是半 个小时以前
```
**消息 ack机制（至少消费一次，不允许消息丢失，途中有消费失败的，即使之后的消费成功了，都会从失败的那个点开始重新消费）**
RocketMQ是以consumer group+queue为单位是管理消费进度的，以一个consumer offset标记这个这个消费组在这条queue上的消费进度
每次消息成功后，本地的消费进度会被更新，然后由定时器定时同步到broker，以此持久化消费进 度，但是每次记录消费进度的时候，只会把一批消息中最小的offset值为消费进度值，如下图:
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678524963563-62edf791-b771-4ef1-bc19-2192e9713961.png#averageHue=%23f4f4f4&clientId=u936e2e69-5ce4-4&from=paste&height=288&id=u7708300f&originHeight=576&originWidth=1486&originalType=binary&ratio=2&rotation=0&showTitle=false&size=118769&status=done&style=none&taskId=u702b3679-7914-4dff-95f5-976d6e4482c&title=&width=743)
**重复消费的问题（上述ack的特性导致的）**
  	定时方式和传统的一条message单独ack的方式有本质的区别，性能上提升的同时，会带来一个潜 在的重复问题，由于消费进度只是记录了一个下标，就可能出现拉取了100条消息如 2101-2200的消 息，后面99条都消费结束了，只有2101消费一直没有结束的情况。
对于这个场景，RocketMQ暂时无能为力，所以业务必须要保证消息消费的幂等性，这也是 RocketMQ官方多次强调的态度。自己实现消息的幂等性


###### 文件刷盘机制(基于mmap)
RocketMQ 的消息是存储在磁盘上的，这样做有两个优点:保证断电后恢复，让存储的消息量超出内存的限制
RocketMQ 存储与读写是基于 JDK NIO 的内存映射机制，具体使用 MappedByteBuffer(基于 MappedByteBuffer 操作大文件的方式，其读写性能极高)RocketMQ 的消息是存储到磁盘上的，这样 既能保证断电后恢复，又可以让存储的消息 超出内存的限制 RocketMQ 为了提高性能，会尽可能地保证 磁盘的顺序写 消息在通过 Producer 写人 RocketMQ 的时候，有两种写磁盘方式:
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678525225486-dd72cc94-9e0a-4108-876b-e4be6202af7f.png#averageHue=%23f1e9e9&clientId=u936e2e69-5ce4-4&from=paste&height=678&id=u529ec03f&originHeight=1356&originWidth=1480&originalType=binary&ratio=2&rotation=0&showTitle=false&size=378950&status=done&style=none&taskId=u11987f26-e55d-408d-9bf9-f31fbaeb742&title=&width=740)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678525244610-5ea8a79f-e91a-4a62-ad25-f7d6db126cf4.png#averageHue=%23ebebeb&clientId=u936e2e69-5ce4-4&from=paste&height=374&id=u36a86725&originHeight=748&originWidth=1680&originalType=binary&ratio=2&rotation=0&showTitle=false&size=283406&status=done&style=none&taskId=uedd089c0-c446-4666-958c-f9ef5f40e7e&title=&width=840)
###### 过期文件删除
默认凌晨4点扫描删除过期文件。文件日期依次删除

- 开启定时任务每10s扫描是否有文件需要删除 
- 有三种情况会进入删除文件操作:到了deleteWhere指定的时间点(默认是凌晨4点)、磁盘不 足、手动触发 
- 对于磁盘不足的情况，当磁盘使用率大于磁盘空间警戒线水位(默认是90%)，会阻止消息写入， 当超过85%时会强制删除文件(需要设置允许强制删除参数，否者不生效)，其他两种情况都只能 删除过期的文件(文件最后更新时间+文件最大的存活时间 < 当前时间) 当被删除的文件存在引用时，会有一个文件删除缓存时间，在这段时间内，该文件不会被删除，主 要是留给引用该文件程序一些时间，当超过了文件删除缓存时间后，每次都会将该文件的引用减少 1000，直到减少小于等于0后才释放该文件引用的相关资源，然后将该文件放入一个“文件删除集 合”中一次连续删除文件中间会存在一定的间隔，不会连续释放文件相关的资源 一次连续删除的文件总和不大于10将“文件删除集合”中的文件从硬盘上删除


##### 高可用

- NameServer 高可用

由于 NameServer 节点是无状态的，且各个节点直接的数据是一致的，故存在多个 NameServer 节点的情况下，部分 NameServer 不可用也可以保证 MQ 服务正常运行

-  BrokerServer 高可用
- 消息消费高可用
- 消息发送高可用
- 消息主从复制
#####  Dledger高可用集群
Dledger是 RocketMQ 4.5 引入的实现高可用集群的一项技术，该模式下集群会随机选出一个节点作为Master，当Master节点挂了后，会从Slave中自动选出一个节点升级成为Master。 Dledger会从集群中选举出 Master 节点，完成 Master 节点往 Slave 节点的消息同步，且接管Broker的 CommitLog 消息存储，Dledger是使用 Raft 算法来进行节点选举的。
