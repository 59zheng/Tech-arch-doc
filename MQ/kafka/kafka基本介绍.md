消息中间件： 减少业务改变需要对上游进行修改
kafka  rocketmq  rabbitmq
http 也算一种消息的传递，需要等待消息的应答。
#### 基础概念
##### 定义:
传统上定义是一个分布式的发布/订阅模式的消息队列，主要应用在大数据，现在卡夫卡已经定义一个分布流平台，用于数据通道处理，数据流分析，数据集成和关键任务应用。

##### kafka 特性

- 多生产者，消费者
- 可持久化操作

允许消费者非实时读取消息，因为kafka将消息按一定顺序持久化到磁盘，保证了数据不会丢失，顺序写磁盘的效率比随机写内存还要高，而且以时间复杂度为0（1）的方法提供消息持久化能力，对TB级以上的数据也能保证常数时间的访问呢能力

- 高吞吐量

kafka 基于发布/订阅模式提供了高吞吐量，kafka每秒可以生产25w消息，处理55w消息

- 可伸缩性

kafka设计为灵活伸缩性的分布式系统，易于向外扩展，对在线的集群进行扩展丝毫不会影响到整体系统的可用性。

- 实时性

由于kafka 可横向扩展生产者，消费者，broker，使的集群可以轻松处理巨大的消息流，在处理大量数据的同时，还能保证亚妙级的消息延迟，实时性极高。

- 容错性

kafka 消息会在集群中进行备份，每个分区都有一台server 作为 leader ，其他 server 作为follwers ，当leader 宕机了，follower 中的一台 server 会自动成为新的leader，继续工作，所以容错性很高且集群的负载是平均的。
##### 应用场景
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677655411174-ff56e73d-2af3-40c7-acfd-601cc68cac8b.png#averageHue=%23fcfcfc&clientId=ua7237156-e459-4&from=paste&height=413&id=u78cc870f&originHeight=826&originWidth=1628&originalType=binary&ratio=2&rotation=0&showTitle=false&size=443635&status=done&style=none&taskId=u60c199bc-93dc-490e-b40f-9dc1c866a6a&title=&width=814)
###### 消息处理
提供比较低的 端到端的延迟，并且提供强大的耐用性的保证。
###### 指标分析
用于检测数据，分布式应用程序生成的统计数据集中聚合
###### 日志聚合
kafka 作为日志聚合解决方案的替代品，比传统的日志采集，传输延迟性更低，并且更容易支持多个数据源和分布式数据消费。
###### 流处理
kafka中消息处理一般包含多个阶段，其中原始输入数据是从kafka主题消费的，然后汇总，丰富，或者以其他的方式处理转化为新主题，
###### 事件采集
状态的变化根据时间的顺序记录下来，kafka支持这种非常大的存储日志数据的场景。

##### kafka 架构图
broker：实例即为一个 broker 屏蔽物理限制， k8s client 的概念
topic：主题： broker 通信的通道； 定义的消息从哪发 类似于 netty 里边的channal ；类似于注册中心，所有需要使用改channel 的人，都需要订阅这个 topic
psrtsion ： 分区概念，将topic 根据消费者多个 分成多个分区，避免抢占消息。如果消费者多余分区，那么多余的消费者处于空闲状态，避免竞争的状态产生，分区多余消费者的时候，消费者消费多个分区。
消息的顺序性：只能保证单个分区内的顺序性，多个分区的消息消费顺序性无法保证。
消费组的概念：一个分区内的消息只能被同一个分组的其中一个消费者消费一次。消息幂等性。同一业务组，避免重复消费
follower 副本的概念，topic 的消息复制，不提供服务，只是冗余的手段
controller ： 控制中心，就是普通的一个broker ，只不过负责一些额外的工作，
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677586036152-79df6d09-55a4-448b-b037-e98ac87c2821.png#averageHue=%23b5d4d6&clientId=ueadebdaf-29b2-4&from=paste&height=518&id=u1c34348d&originHeight=1035&originWidth=1306&originalType=binary&ratio=2&rotation=0&showTitle=false&size=610607&status=done&style=none&taskId=ub7ea13f5-708c-4d07-b746-2f22a20bf86&title=&width=653)

- broker ：kafka的实体服务

消息队列产品本身，指的是具体的kafka server 

- producer ： 生产者，消息的生产方
- consumer ： 消费者, 消息的消费方
- zookeeper ： 用于管理协调 kafka 的broker

用于通知生产者和消费者kafka系统中存在任何新代理或 kafuka系统中代理失败，根据 zookeeper 接收到的关于代理的存在或失败的通知，然后生产者和消费者采取决定并开始喝某些其他代理协调任务。

- controller ： 在集群中选择一个 broker 担任控制器的角色，控制器是kafka 集群的中心

本身也是一个普通的broker，只不过需要复杂一些额外的工作（追踪集群中的其他 broker ，并在合适的时候处理新加入和失败的broker 节点，Rebalance 分区，分配新的leader 分区等）。kafka集群中始终只有一个 controller broker

- cluster：集群指的由多个 broker 共同构成的一个整体，对外提供统一的服务。
- Topic ： kafka下消息的类别，类似于 RabbitMQ 中的 Exchange 的概念，netty 中的 channel 的概念。

kafka 中的消息以主题来进行归类，生产者将消息发送到特定的topic 上，订阅该主题的消费者在topic 上拉取消费消息。（不是推送的模式，下述由说明）

- partition 分区概念，（避免多消费者抢占同一个队列中的消息，加快消息的消费速率）有利于水平扩展，避免单台机器在磁盘空间和性能上的限制，同时可以通过复制来增加数据的冗余性，提高容灾能力，为了做到均匀分布，通常 partition 数量是 broker server 数量的整数倍（如果消费者多余分区，那么多余的消费者处于空闲状态，避免竞争的状态产生，分区多余消费者的时候，消费者消费多个分区。）
- replication 副本：主分区（leader）会将数据同步到分区（follower） 不参与副本的概念，topic 的消息复制，不提供服务，只是冗余的手段
##### 消费模型
###### 推模式
broker 主动将消息推向 Consumer ，即 Consumer 被动的接受消息，由broker 来主导消息的发送，

- 优点： 消息实时性高，消费者使用比较简单
- 缺点：
   - 推送速率难以适应消费速率，容易消息积压，压垮消费者
   - broker 需要维护多个消费者的消费速率
###### 拉模式 （kafka 和 rocketMQ 都选择此模式，activeMq 是推的模式）
Consumer 主动向 broker 拉取消息，为空的话会有一段时间的停顿等待时间 降低消费者的访问次数
长轮训模式： 简单的说就是消费者去 Broker 拉消息，定义了一个超时时间，也就是说消费者去请求消息，如果有的话马上返回消息，如果没有的话消费者等着直到超时，然后再次发起拉消息请求。并且 Broker 也得配合，如果消费者请求过来，有消息肯定马上返回，没有消息那就建立一个延迟 操作，等条件满足了再返回
##### 消息发送模型
定时发送 （默认是 0 毫秒）或 16k缓存  数据截断问题存在，投递过去然后解析成多个消息。kafka应答成功之后再删除消息。
内存缓存池： 所有队列能够占用的总大小。从生产到投递的时候会有一个速度差，一个是进程间通信，一个会涉及到网络通信，所以存在速度差。缓存用来存储由于速度差滞留的消息。达到上限，生产者需要等待一会。
发到 send 区域，然后服务器死了，会丢失数据
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677588865416-1ddf59a4-2cda-4f69-a74d-dec9f58d4423.png#averageHue=%23f0f0f0&clientId=ueadebdaf-29b2-4&from=paste&height=397&id=u221954bb&originHeight=794&originWidth=1319&originalType=binary&ratio=2&rotation=0&showTitle=false&size=263727&status=done&style=none&taskId=u828d419d-eec0-4678-9786-7f292b5c437&title=&width=659.5)

###### kafka 消息拦截器
interceptor 拦截器，在不修改逻辑的情况下，动态的视线一组可插拔的事件处理逻辑链，在主业务操作的前后多个时间带你上插入不同的对应的拦截逻辑
**分类：**

- 生产者拦截器： 在发送消息前以及消息提交成功后植入拦截器逻辑
- 消费者拦截器：支持在消费钱以及提交位移后编写特定拦截

-- 都支持链的方式：即你可以将一组拦截器串连成一个大的拦截器，Kafka 会按照添 加顺序依次执行拦截器逻辑
implements ProducerInterceptor  生产者拦截器
implements ConsumerInterceptor 消费者拦截器
###### 序列化机制
默认序列化器（一般使用StringSerializer 序列化方式，如果入参和序列化方式不一样会导致报错）
```
ByteArraySerializer // 序列化Byte数组，本质上什么都不用做。 
ByteBufferSerializer // 序列化ByteBuffer。 
BytesSerializer // 序列化Kafka自定义的Bytes类。
StringSerializer // 序列化String类型。
LongSerializer // 序列化Long类型。 
IntegerSerializer // 序列化Integer类型。
ShortSerializer // 序列化Short类型。
DoubleSerializer // 序列化Double类型。 
FloatSerializer // 序列化Float类型。
```
###### 消息分区算法

- 默认分区策略
   - 指明partition的情况下，直接将指明的值直接作为partiton 值;
   - 没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进 行取余得到 partition 值;
   - 既没有 partition 值又没有 key 值的情况下， kafka 采用 Sticky Partition (黏性分区器)， 会随机选择一个分区，并尽可能一直使用该分区，待该分区的batch已满或者已完成，kafka再随机 一个分区进行使用，(以前是一条条的轮询，现在是一批次的轮询)
###### 消息发送方式

- 发送并忘记；发送消息，不关心消息是否发送成功

本质上是一种异步发送的方式，将消息先存储在缓存区中，达到设计条件后批量发送，是kafka吞吐量最高的一种形式，并配合 acks=0 生产者不需要等待服务器的影响

- 同步发送： 同步发送，send()方法会返回 Futrue 对象，通过调用 Futrue 对象的 get() 方法，等待直到结果 返回，根据返回的结果可以判断是否发送成功

业务要求消息必须是按顺序发送的，那么可以使用同步的方式，并且只能在一个partation上， 结合参数设置 retries 的值让发送失败时重试，设置 max_in_flight_requests_per_connection=1 ，可以控制生产者在收到服务器晌应之前只能发送1个 消息，在消息发送成功后立刻flush，从而控制消息顺序发送。
在调用send()方法后再调用get()方法等待结果返回，如果发送失败会抛出异常，如果发送成功会返 回一个 RecordMetadata 对象，然后可以调用offset()方法获取该消息在当前分区的偏移量。

- 异步发送：异步发送，在调用send()方法的时候指定一个callback函数，当broker接收到返回的时候，该 callback函数会被触发执行

如果业务需要知道消息发送是否成功，并且对消息的顺序不关心，那么可以用异步+回调的方式来 发送消息，配合参数 retries=0 ，并将发送失败的消息记录到日志文件中;要使用callback函数，先要 实现 org.apache.kafka.clients.producer.Callback 接口，该接口只有一个 onCompletion 方法， 如果发送异常， onCompletion 的参数 Exceptione 会为非空。
##### 消息缓存池
producer 发送消息时候，在kafka客户端内部，把属于同一个 topic 分区的消息先汇总起来，形成一个batch，真正向kafka服务器发送的消息，都是一 batch 为单位的。
为了性能，可以批量发送，减少网络socket 的创建。 默认批次大小 由参数 batch.size控制，默认16 kb 是否发送由 batch.size 和 acks 两个参数控制，满足其一可以发送，可以根据业务修改这部分参数。
###### 内存池复用：
场景： kafka 是批量发送的，会涉及到 内存的回收和重新分配，如果交给 jvm 管理的话。这样会影响效率。
解决： 类似于 netty 的buffer ，申请一个固定大小的内存块，读写两端，不需要清空内存，直接覆盖即可。即只需要从指定位置拿指定长度的内存即可，始终是连续的的指针。
##### 提高吞吐量

- 分区发送

多分区发送，一般分区数不大于 broker 的节点数，注意分区数只能动态添加，不能减少

- 批量发送

增大 batch.size 的大小，以及增大 acks 的时间，减少发送次数，增加单次发送的容量，需要结合业务

- 数据压缩

序列化方式，对序列化的二进制数据进行二次压缩。
Kafka还支持对消息集合进行压缩，Producer能够经过GZIP或Snappy格式对消息集合进行压缩

   - compression.type: 压缩，默认none，修改为snappy
```
public class CustomProducerParameters {
    static {
// batch.size:批次大小，默认16K
 KafkaConstant.PRODUCT_PROPERTIES.setProperty(ProducerConfig.BATCH_SIZE_CONFIG,
"16384");
// linger.ms:等待时间，默认0，修改为20ms KafkaConstant.PRODUCT_PROPERTIES.setProperty(ProducerConfig.LINGER_MS_CONFIG,
"20");
// RecordAccumulator:缓冲区大小，默认32M，buffer.memory，修改为64m
 KafkaConstant.PRODUCT_PROPERTIES.setProperty(ProducerConfig.BUFFER_MEMORY_CONFI
G, "67108864");
// compression.type:压缩，默认none，可配置值gzip、snappy、lz4和zstd，修改为
snappy
 KafkaConstant.PRODUCT_PROPERTIES.setProperty(ProducerConfig.COMPRESSION_TYPE_CO
NFIG, "snappy");
    }
    private static KafkaProducer<String, String> producer = new
KafkaProducer<String, String>(KafkaConstant.PRODUCT_PROPERTIES);
public static void main(String[] args) { // 4.调用send方法，发送消息
for (int i = 0; i < 5; i++) {
            producer.send(new ProducerRecord<>("test", "testMessage" + i));
        }
// 5.关闭资源
        producer.close();
    }
}
```
#### 数据可靠性保证
##### -生产者保证
###### 分区副本：
kafuka的topic 是可以分区的，并且可以为分区配置多个副本，改配置可以通过replication.factor参数实现
 kafuka中的分区副本包括两种类型：领导者副本（Leader Replica）和追随者副本（FollowerReplica），每个分区在创建时都要选举一个副本作为 leader ，其余的副本自动变为 follower 。
每个分区有三个副本，follower不对外提供服务的，也就是说，任何一个follower 都不能影响消费者和生产者的读写请求，所有的请求都必须由领 leader 来处理，所有的读写请求都必须 发往leader 所在的 broker ，由该 broker 负责处理，flolower 不处理客户端请求，唯一的任务就是从 leader 异步拉取消息，并写入自己的提交日志中，完成数据同步。leader 挂了之后，会随机选择一个新的 leader。
###### 副本定义
kafuka topic 下有多个分区，每个分区可以有多个副本，kafuka的副本以分区为维度进行划分
同一个分区下的所有副本保存有相同的消息序列，分散保存在不同的 broker 上，从而对抗部分 broker 宕机带来的数据不可用的情况，容灾。
###### 副本同步队列 ISR
isr 是kafka 提供的数据复制算法，如果 leader 发生故障或挂掉，就会从ISR列表中选举出来一个新 leader ，并被接受客户端的消息成功写入。
确保从同步副本中选举出的可用副本 和之前 leader 的数据同步。每条follwer 拉取的消息，都会会不在ISR队列中，当follwer 落后过多或者失败的情况下，leader 会将它从 ISR中删除。
###### 同步条件

- 副本节点必须能与zookeeper 保持会话（心跳机制不断）
- 副本能同步 leader 上的所有写操作，并且不能落后太多（卡住或之后的副本有 relica.lag.time.max.ms配置）
###### 副本的角色
三种角色组成

- AR（assigned replica）:所有副本的统称，AR=LSR+OSR
- ISR(in -sync Replica）:同步中的副本，可以参与 leader 选主，一但落后太多会被踢到 OSR，默认落后10s之后就会被踢到 OSR中
- OSR（Out-Sync Relipcas）提出同步的副本，会一直追赶leader ，追上的话会进入ISR中

默认情况下，Kafka topic 的replica 数量为1，每个分区都有一个唯一的 leader，确保消息的可靠性，由broker的参数offsets.topic.replication.factor指定 大小设定为大于等于2 的值。
SR是AR的一个子集，由leader维护ISR列表，follower从Leader同步数据有一些延迟，任意一个超 过阈值都会把follower踢出ISR，存入OSR(Outof-Sync Replicas)列表，新加入的follower也会先存放 到OSR中， AR = ISR + OSR
###### kafka 主从同步
kafka的副本因子是3，即每个分区只有一个leader 副本和2两个 follower 副本
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677830493749-98c99c92-8ffb-4167-8271-0cd875d5b052.png#averageHue=%23f2f1e8&clientId=u4e3ea9d8-9858-4&from=paste&height=474&id=ucdbeac34&originHeight=948&originWidth=1578&originalType=binary&ratio=2&rotation=0&showTitle=false&size=324413&status=done&style=none&taskId=uf8eb7d96-7da6-435d-ae98-51a8501bbe5&title=&width=789)
###### acks确认机制：
acks参数指定了必须有多少个分区副本收到消息，生产者才任务该消息是写入成功，这个参数保证消息不丢失

- acks=0：换句话说，一旦出现了问题导致服务器没有收到消息，那么生产者就无从得知，消息也就丢失了，

该配置由于不需要等到服务器的响应，所以可以以网络支持的最大速度发送消息，从而达到很高的吞吐
量

- acks=1 ： 表示只要集群的 leader 分区副本接收到了消息，就会向生产者发送一个成功响应的ack，此时生 产者接收到ack之后就可以认为该消息是写入成功的。  优先写入 的是 leader 节点 。

存在问题: 一旦消息无法写入leader分区副本(比如网络原因、leader节点崩溃)，生产者会收到一个错误响应， 当生产者接收到该错误响应之后，为了避免数据丢失，会重新发送数据，这种方式的吞吐量取决于使用 的是异步发送还是同步发送。
如果生产者收到了错误响应，即便是重新发消息，还是会有可能出现丢数据的现象，比如，如果一 个没有收到消息的节点成为了新的Leader，消息就会丢失。

- acks=all /-1 所有副本以及 leader 都写入后返回。延迟会很高  isr 挂了一个，还没更新过来 all 的时候会卡住，等待isr 补全
- 最小同步副本： 

Kafka的Broker端提供了一个参数min.insync.replicas，该参数控制的是消息至少被写入到多少个 副本才算是"真正写入",该值默认值为1，生产环境设定为一个大于1的值可以提升消息的持久性，如果同 步的副本的数量低于该配置值，则生产者会收到错误响应，从而确保消息不丢失。 配合 all 使用。不够的话，会一直抛日志。只有到 asks=all 的生效 
acks=0，生产者在成功写入消息之前不会等待任何来自服务器的响应。 
acks=1，只要集群的leader分区副本接收到了消息，就会向生产者发送一个成功响应的ack。
 acks=all，表示只有所有参与复制的节点(ISR列表的副本)全部收到消息时，生产者才会接收到来自 服务器的响应，此时如果ISR同步副本的个数小于 min.insync.replicas 的值，消息会被拒绝写入
###### 生产重试
失败的时候进行重试
默认情况下，生产者将等待重试的时间间隔为100ms，但是你可以使用 retry.backoff.ms 参数来 控制重试的时间，默认重试次数为 Integer.MAX_VALUE ，们建议对broker的选举恢复时间进行测试。
并设置重试次数和重试间隔时间，使重试花费的总时间大于 kafka 集群的故障恢复时间，否则生产 者可能过早放弃消息，并不是所有的错误都能够进行重试，有些错误不是暂时性的，此类错误不建议重 试(如消息太大的错误)，通常由于生产者为你处理重试，所以在你的应用程序逻辑中自定义重试将没 用任何意义，你最好是将精力放在处理不可重试的错误或者失败的情况上面。
 会造成重复消息。
##### 副本同步机制：
 leader 挂了之后 follower 选举为leder 的时候，这个过程中保证不会发生消息丢失或者离散保证。
###### 相关概念

- HW (high watermark) 消费高位，最高消费到哪，之后的数据对消费者影藏 (所有 follower 读的最低水平) 数据被所有的副本都同步了，才会被消费者消费 （木桶效应 -标记最短的） 保证消费者的一致性
- LEO (last end offset) 最后偏移量，记录了该副本对象底层日志文件中下一条消息的位移量，副本写入消息的时候，会自动更新lED值
- Remoter LEO  远程 leo
###### LEO 更新

1. leader 副本自身的 LEO 值更新:在 Producer 消息发送过来时，即 leader 副本当前最新存储的消 息位移位置 +1;
2. follower 副本自身的 LEO 值更新:从 leader 副本中 fetch 到消息并写到本地日志文件时，即 follower 副本当前同步 leader 副本最新的消息位移位置 +1;
3. leader 副本中的 remote LEO 值更新:每次 follower 副本发送 fetch 请求都会包含 follower 当前 LEO 值，leader 拿到该值就会尝试更新 remote LEO 值。
###### HW 更新时机 
故障时更新

1. 副本被选为 leader 副本时:当某个 follower 副本被选为分区的 leader 副本时，kafka 就会尝试更 新 HW 值;
2. 副本被踢出 ISR 时:如果某个副本追不上 leader 副本进度，或者所在 broker 崩溃了，导致被踢出 ISR，leader 也会检查 HW 值是否需要更新，毕竟 HW 值更新只跟处于 ISR 的副本 LEO 有关系。

正常时更新

1. producer 向 leader 副本写入消息时:在消息写入时会更新 leader LEO 值，因此需要再检查是否 需要更新 HW 值;
2. leader 处理 follower FETCH 请求时:follower 的 fetch 请求会携带 LEO 值，leader 会根据这个值 更新对应的 remote LEO 值，同时也需要检查是否需要更新 HW 值

follower HW 更新

1. follower 更新 HW 发生在其更新 LEO 之后，每次 follower Fetch 响应体都会包含 leader 的 HW
值，然后比较当前 LEO 值，取最小的作为新的 HW 值。
###### 同步过程
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677834892925-3d9c1373-18f8-405c-a628-464662f0c681.png#averageHue=%23f2f1ef&clientId=u4e3ea9d8-9858-4&from=paste&height=776&id=u8ec6fc00&originHeight=1552&originWidth=1542&originalType=binary&ratio=2&rotation=0&showTitle=false&size=699125&status=done&style=none&taskId=ue633ff6e-14ab-464e-8805-d9ec3859867&title=&width=771)
###### 存在问题
leader 挂了 follower 由于是根据 HW 同步的，不会比较内容，会造成消息混乱或者丢失
###### 解决
leader epoch 皇帝纪元/时代

- Epoch 单调递增的版本号，leader 变更的时候递增
- 起始位移（start Offset）Leader 副本在该 Epoch 值上写入的首条消息的位移(LEO)。

在同步数据的时候不直接截断数据，先找leader 同步当前的消息个数 解决数据丢失

###### 数据去重
至少一次可以保证数据不丢失，但是不能保证数据不重复
 最多一次可以保证数据不重复，但是不能保证数据不丢失


###### 消息幂等性支持
不能任意扩充 分区  需要消息少的时候，
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677895172034-05a2029d-7829-4024-9044-91f683074548.png#averageHue=%23e8e5e0&clientId=u8443bc76-9848-4&from=paste&height=303&id=u4a7b024c&originHeight=606&originWidth=1606&originalType=binary&ratio=2&rotation=0&showTitle=false&size=195420&status=done&style=none&taskId=u970cd1c2-0b94-4eb5-a5f8-0f85023a618&title=&width=803)

- 其中PID是Kafka每次重启都会分配一个新的; 
- Partition表示分区号;
- Sequence Number是单调自增的 
- 幂等性只能保证是在单分区单会话内不重复
###### 数据有序性
单分区多条数据的有序性
kafka 一次投递5条消息  机油 消息的 Sequence Number 在写的时候进行判断，一次性写多条，只有前面的值写入成功才能写入
消息乱序的原因如下主要是由 max.in.flight.requests.per.connection 导致的 最多配置5 不支持超过 5

**kafka1.0之前**
 需要做以下配置才能够发送有序消息
max.in.flight.requests.per.connection=1 (不需要考虑开启幂等性)
**kafka1.0之后**
在1.x 版本之后，如果没有开启幂等性，则也需要以下配置max.in.flight.requests.per.connection=1 (不需要考虑开启幂等性)max.in.flight.requests.per.connection=1


##### 

#### 消费信息
##### 消费者和消费组
一个发布在Topic上消息被分发给此消费者组中的一个消费者，假如所有的消费者都在一个组中，那 么这就变成了队列模型， 假如所有的消费者都在不同的组中，那么就完全变成了发布-订阅模型。
###### 消费者组
kafka消费者组(Consumer Group)是kafka提供的可扩展且具有容错性的消费者机制。
它是一个组，所以内部有可以有多个消费者，这些消费者共用一个ID(Group ID)，一个组内的所 有消费者共同协作，完成对订阅的主题的所有分区进行消费，其中一个主题中的一个分区只能由一个消 费者消费。
特性：

1. 一个消费者组可以有多个消费者。
2. Group ID是一个字符串，在一个kafka集群中，它标识唯一的一个消费者组。
3. 每个消费者组订阅的所有主题中，每个主题的每个分区只能由一个消费者消费，消费者组之间不影
响。
###### 消费者分配
单消费者：一个消费者订阅一个主题就行消费
多消费者：在启动消费者的时候 会把不同的分区，分配给不同的消费者；一个分区最多给到消费组中的一个消费者
提高消费速率，需要同时提供 消费者数量 和分区 数量。 一般数量== tomcat 实例，也会预估值，也会剩余两个分区。
###### 消费分配策略
新起消费者之后分区的分配celve

- RangeAssignor

分段的概念：分区总数/消费线程数，如果有余，则表明有的消费线程之间分配的分区不均 匀，那么这个多出来的分区会给前几个消费线程处理。 
同一个消费者 消费的分区 相邻， 注意这块，如果消息投递的时候解析出来的 key hash 相近，会造成部分消费者压力大，部分消费者空闲
配置方式
```
KafkaConstant.CONSUMER_PROPERTIES.setProperty(ConsumerConfig.PARTITION_ASSIGNMEN
T_STRATEGY_CONFIG, "org.apache.kafka.clients.consumer.RangeAssignor");
```

- RoundRobinAssignor

将所有可用partitions和consumers展开(字典排序)，以轮询的方式将partitions依次分配给 consumers。
如果consuemrs订阅Topics都是相同的，那么partitions将会被均匀分配给每个consumer，最理想 的状态是partitions数是consumers数的整数倍，这样每个consumer都有相同数量的partitions数。
这个分区分配策略简单来说就是列出所有的分区，然后和消费线程之间进行循环的分配即可；
如果要使用该分配策略，你需要满足所有的消费线程都是消费相同的topic，且每个消费者之间 的消费线程数是一样的
```
KafkaConstant.CONSUMER_PROPERTIES.setProperty(ConsumerConfig.PARTITION_ASSIGNMEN
T_STRATEGY_CONFIG, "org.apache.kafka.clients.consumer.RoundRobinAssignor");
```

- StickyAssignor

StickyAssignor（粘性）分区分配算法，目的是在执行一次新的分配时，能在上一次分配的结果的基础上， 尽量少的调整分区分配的变动，节省因分区分配变化带来的开销。
分配规则：StickyAssignor 的分配规则很麻烦，主要的作用就是尽量的实现均衡，然后尽量减少分区变化
```
KafkaConstant.CONSUMER_PROPERTIES.setProperty(ConsumerConfig.PARTITION_ASSIGNMEN
T_STRATEGY_CONFIG, "org.apache.kafka.clients.consumer.StickyAssignor");
```
##### kafka 消费消息
###### 消费者线程安全
kafka 的 Java consumer是单线程的设计，准确来说是双线程，从 kafka 0.10.1.0 版本开 始kafkaConsumer变成了用户主线程和心跳线程的双线程设计。
所谓用户主线程，就是你启动Consumer应用程序的main方法的那个线程，而心跳线程 (Heartbeat Thread)只负责定期发送心跳给对应的Boroker,以标识消费者应用的存活性，引入心跳线 程的目的还有一个:解耦真实的消息处理逻辑与消费者组成员存活性管理。
尽管多了一个心跳线程，但是实际的消息处理还是由主线程完成。所以我们还是可以认为 KafkaConsumer是单线程设计的。
kafka不支持 多线程消费。可以拿到消息之后投递到线程池中进行处理，只是把业务性的操作进行异步处理
**设计原因**

1. counsumer 设计了单线程+轮训的机制，这种设计能够较好的视线非阻塞式的消息获取
2. 单线程的设计能简化 Counsumer 端的设计，将处理消息的逻辑是否使用多线程的选择交由业务决定
3. 不管哪种编程语言，单线程的设计都比较容易实现，并且，单线程设计的Consumer 更容易移植到其他语言上。
###### 死信队列& 重试队列

- 死信队列：遗言性质，mq消费失败，重试也失败，转移到特定的 topic
- 重试队列:  消费失败进行重试，消息回滚也不会一直进行重复消费

Kafka不支持重试机制也就不支持消息重试，也不支持死信队列，因此使用kafka做消息队列时，如 果遇到了消息在业务处理时出现异常的场景时，需要额外实现消息重试的功能。 
###### 消息丢失&消息重复

- 重复消费： 消费了数据没有提交offset， 

kafka 消息消费之后，数据还在。 会记录一个消费者偏移量，不会进行重复的消费，消费之后的数据是保留一定时间，以及一个量之后会删除 7天量级 
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677899870581-b22d33b3-2c15-48cc-9c30-1b59c4cc82f6.png#averageHue=%23f0f0f0&clientId=u2d8bde90-96fd-4&from=paste&height=485&id=u00e46a8e&originHeight=970&originWidth=1768&originalType=binary&ratio=2&rotation=0&showTitle=false&size=344639&status=done&style=none&taskId=u980a0f56-2d89-47c3-80b7-40116170c3e&title=&width=884)

- 消息丢失：提交了offset后消费，可能会造成数据漏消费

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677899958294-32109c2d-3958-4435-9077-88aed93a4d49.png#averageHue=%23f3f1e1&clientId=u2d8bde90-96fd-4&from=paste&height=351&id=u9e23ba7b&originHeight=702&originWidth=1652&originalType=binary&ratio=2&rotation=0&showTitle=false&size=217450&status=done&style=none&taskId=uea6195b7-bed6-42f5-a94c-8ddc8eb7516&title=&width=826)
如果将offset设置为手动提交，当offset被提交时，数据还在内存中未处理，刚好消费者宕机， offset已经提交，数据未处理，此时就算再启动consumer也消费不到之前的数据了，导致了数据漏消费
如果想要consumer精准一次消费，需要kafka消息的消费过程和提交offset变成原子操作，此时需 要我们将kafka的offset持久化到其他支持事务的中间件(比如MySQL)

- 消息堆积
1. 如果是kafka消费能力不足，考虑增加topic分区数，并且同时增加消费者组的消费者数量，因为一 个partition只能被CG中的一个consumer消费，所以partition和consumer必须同时增加
2. 如果是下游数据处理不及时，可以提高每次拉取的数量，批次拉取数据过少，使得处理数据小于生 产的数据
3. ![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677900024067-6c3ca09d-396a-4652-b57f-949d1400fc42.png#averageHue=%23eeeeee&clientId=u2d8bde90-96fd-4&from=paste&height=276&id=u048e4cba&originHeight=552&originWidth=1716&originalType=binary&ratio=2&rotation=0&showTitle=false&size=206574&status=done&style=none&taskId=u8e4cbe12-799b-403d-b5eb-327d2c187e2&title=&width=858)
###### 
##### 消费者位移管理
在 zookeeper 里边记录（新版是在Consumer Group 的主题中 默认是 50个 分区） 存储 当前消费者组的当前消费位移量，以组为单位。
###### 自动提交
位移的提交方式： 自动提交
kafka 每隔5s 自动提交一次位移，自动提交位移的动作
###### 手动提交
和自动提交相反, 手动提交需要设置 ，然后用户自己来管理分区位 移，手动提交又可以分为同步提交和异步提交。
手动位移提交就是用户自行确定消息何时被真正处理完并可以提交位移 ，手动提交位移 API 进一步 细分为同步手动提交和异步手动提交，即 commitSync 和 commitAsync 方法。
拉取消息后「先提交 Offset，后处理消息」，如果此时处理消息的时候异常宕机，由于 Offset 已 经提交了, 待 Consumer 重启后，会从之前已提交的 Offset 下一个位置重新开始消费， 之前未处 理完成的消息不会被再次处理，对于该 Consumer 来说消息就丢失了。 拉取消息后「先处理消息，在进行提交 Offset」， 如果此时在提交之前发生异常宕机，由于没有 提交成功 Offset， 待下次 Consumer 重启后还会从上次的 Offset 重新拉取消息，不会出现消息丢 失的情况， 但是会出现重复消费的情况，这里只能业务自己保证幂等性。
###### 同步位移提交
调用 commitSync() 时，Consumer 程序会处于阻塞状态，直到远端的 Broker 返回提交结果，这 个状态才会结束，这可能会影响整个应用程序的 TPS，虽然可以通过降低提交频率来提升吞吐量，但一 旦发生再均衡，会增加重复消息的数量，如果提交过程中出现异常，该方法会将异常信息抛出。
###### 异步提交
因为同步位移提交的阻塞特性，会降低TPS，我们可以采用异步位移提交
commitAsync() 是一个异步非阻塞调用，调用 commitAsync() 之后，它会立即返回，不会阻塞，因
此不会影响 Consumer 应用的 TPS，Kafka 给 commitAsync() 方法提供了回调函数。
consumer 在后续 poll 调用时轮询该位移提交的结果，特别注意的是，这里的异步提交位移不是指 consumer 使用单独的线程进行位移提交，实际上 consumer 依然会在用户主线程的 poll 方法中不断轮
询这次异步提交的结果，只是该提交发起时此方法是不会阻塞的，因而被称为异步提交
**混合位移提交**
###### 指定位移提交
kafka 消费之后的消息会存储在本地，可以指定位移位置，重新开始消费
###### 日志压缩
自动提交位移很省事，但是不够灵活，而且由于是自动提交位移，即使当前位移主题没有消息可以 消费了，位移主题中还是会不停地写入最新位移的消息
显然 Kafka 只需要保留这类消息中的最新一条就可以了，之前的消息都是可以删除的，这就要求 Kafka 必须要有针对位移主题消息特点的消息删除策略，否则这种消息会越来越多，最终撑爆整个磁 盘，Kafka 使用 Compact 策略来删除位移主题中的过期消息
日志压缩 Log Compaction 对于有相同 key 的不同 value 值，只保留最后一个版本，Kafka 提供了 专门的后台线程Log Cleaner定期地巡检待 Compact 的主题，看看是否存在满足条件的可删除数据。
###### 优雅退出
当需要退出poll循环时，我们可以使用另一个线程调用 consumer.wakeup() ，调用此方法会使得 poll() 抛出 WakeupException ，如果调用 wakup 时，主线程正在处理消息，那么在下一次主线程调用 poll 时会抛出异常。
主线程在抛出 WakeUpException 后，需要调用 consumer.close() ，此方法会提交位移，同时发 送一个退出消费组的消息到Kafka的组协调者，组协调者收到消息后会立即进行重平衡(而无需等待此消 费者会话过期)。
###### 分区再均衡
新的消费者加入的时候进行分区的重新分配。触发：消费者变化，分区变化
缺点： 再均衡期间，群组不能接收消息， 均衡协议需要相同
再均衡监听器
```
//此方法会在消费者停止消费消费后，在重平衡开始前调用。
public void onPartitionRevoked(Collection partitions) //此方法在分区分配给消费者后，在消费者开始读取消息前调用
public void onPartitionAssigned(Collection partitions)
```


#### kafka 存储结构
kafka 使用日志文件的方式来保存生产者和发送者的消息，每条消息都有一个 offset 值来表示它在
分区中的偏移量
###### 分区分配方式
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677913549966-2ff39c9b-23ab-4ffd-88e0-2d3f03bb5614.png#averageHue=%23f5f4f1&clientId=u2d8bde90-96fd-4&from=paste&height=713&id=u0e37b11f&originHeight=1426&originWidth=1772&originalType=binary&ratio=2&rotation=0&showTitle=false&size=427929&status=done&style=none&taskId=u2fc57ee7-487e-4caf-856f-517bf49bd51&title=&width=886)
**topic**
topic可以对应多个partition，而topic只是逻辑概念，不涉及到存储，partition才是物理概念，同 一topic的不同partition可能分布在不同机器上
**partition**
分区(partition)是有序的， 新的不可变的消息增加到尾部，一个分区不能扩多个boker，甚至不能 跨多个磁盘
partition是一个文件夹，其中包含多个segment，每个partition是一个有序的队列，partition中的 每条消息都会分配一个有序的id，即offset
**segment**
为了更加方便管理 Partation中的数据， 将一个 Partation 拆分成多个大小相等的数据段，这每 一小段数据被称为 segment 
每个 segment 文件的大小相等，但消息数量不一定相等，这种特性方便已经被消费的消息的清 理，提高磁盘的利用率
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677913933303-9f2db684-59d0-4e68-a2b0-161eb6c9bba1.png#averageHue=%23f1f1f1&clientId=u2d8bde90-96fd-4&from=paste&height=409&id=u5ef84d09&originHeight=818&originWidth=1640&originalType=binary&ratio=2&rotation=0&showTitle=false&size=311642&status=done&style=none&taskId=uaba46ca0-0419-4b3a-80fd-521b81647ea&title=&width=820)
每个分片目录中，kafka 通过分段的方式将 数据 分为多个 LogSegment ，一个 LogSegment 对应 磁盘上的一个日志文件(00000000000000000000.log)和一个索引文件(如上: 00000000000000000000.index)，其中日志文件是用来记录消息的，索引文件是用来保存消息的索引。
每个LogSegment 的大小可以在 server.properties 中 log.segment.bytes=1073741824 (设置 分段大小,默认是1GB)选项进行设置。
offset 的最大值为 Long.MAX_VALUE = 2的63次方-1，这也是一个 segment 能存储消息的最大条 数，超出就需要分多个 segment 存储，当然，由于 kafka 限制了 segment 的大小，所以基本不会出 现这样的情况
###### 稀疏索引
为消息数据建了两种稀疏索引，一种是方便 offset 查找的 .index 稀疏索引，
还有一种是方便时 间查找的 .timeindex 的 稀疏索引   基于 offset 查找 很慢，线上不建议使用
##### 日志文件清理机制
日志删除：按照一定的保留策略直接删除不符合条件的日志分段 (LogSegment)。
日志压缩：针对每个消息的key进行整合，对于有相同key的不同value值，只 保留最后一个版本。
日志删除：

- 基于时间删除： 

周期性的检查日志删除条件， 基于失效时间触发。删除的是过期的时间段，是基于最后的修改时间检索的。
原先是基于文件的修改时间进行删除的，手动会影响结果，后续使用时间戳的形式，记录在文件内部

- 基于日志大小策略：

默认不启用， 指的是所有的日志文件大小加起来达到阈值触发删除的操作（同时需要超过 日志文件的分段最大值，即默认1G）。

- 基于日志起始偏移量进行删除：

小于指定偏移量的log文件进行删除。需要自行计算偏移量，确保小于该偏移量的都消费过了。



#### 优化
##### 性能优化：
###### 批量发送消息
Kafka 采用了批量发送消息的方式，通过将多条消息按照分区进行分组，然后每次发送一个消息集 合，从而大大减少了网络传输的开销
###### 消息压缩
因为有了批量发送这个前期，从而使得 Kafka 的消息压缩机制能真正发挥出它的威力(压缩的本质 取决于多消息的重复性)，对比压缩单条消息，同时对多条消息进行压缩，能大幅减少数据量，从而更 大程度提高网络传输率。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678169509504-f4dd8026-a358-4d3e-a95b-846eb21ba85a.png#averageHue=%23f4f4f4&clientId=uf4e7d042-aef6-4&from=paste&height=257&id=u44065fae&originHeight=514&originWidth=1646&originalType=binary&ratio=2&rotation=0&showTitle=false&size=142752&status=done&style=none&taskId=uf2be3fb3-06ba-49a4-9e30-ec39c5ce1a5&title=&width=823)
###### 高效序列化
可以根据实际情况选用快速且紧凑的序列化方式(比如 ProtoBuf、Avro)来减少实际的网络传输量以及磁盘存储量，进一步提高吞吐量。
##### 存储优化
###### 磁盘顺序写
磁盘顺序读写未必比内存满，随机读写肯定慢
###### 零拷贝
Kafka 用到了零拷贝(Zero-Copy)技术来提升性能，所谓的零拷贝是指数据直接从磁盘文件复制 到网卡设备，而无需经过应用程序，减少了内核和用户模式之间的上下文切换。
不需要用户空间操作，直接从网络-》内核-》内核-》硬盘 即可。 mmap 和 send file 两种技术
简单说检索，数据拷贝的次数，用户线程直接操作内核空间的内的数据
**传统的复制方式**

1. 操作系统将数据从磁盘中加载到内核空间的Read Buffer(页缓存区)中。
2. 应用程序将Read Buffer中的数据拷贝到应用空间的应用缓冲区中。
3. 应用程序将应用缓冲区的数据拷贝到内核的Socket Buffer中。
4. 操作系统将数据从Socket Buffer中发送到网卡，通过网卡发送给数据接收方。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678169650425-bdb4f6a6-349c-4374-ad37-e3eacd25791a.png#averageHue=%23fafaf9&clientId=uf4e7d042-aef6-4&from=paste&height=598&id=u3a8aea9b&originHeight=1196&originWidth=1666&originalType=binary&ratio=2&rotation=0&showTitle=false&size=724954&status=done&style=none&taskId=u4b1c780b-4016-4ab7-b350-8c925493941&title=&width=833)
###### DMA技术
DMA，又称之为直接内存访问，是零拷贝技术的基石，DMA 传输将数据从一个地址空间复制到另 外一个地址空间，当CPU 初始化这个传输动作，传输动作本身是由 DMA 控制器来实行和完成，因此通 过DMA，硬件则可以绕过CPU，自己去直接访问系统主内存，很多硬件都支持DMA，其中就包括网卡、 声卡、磁盘驱动控制器等。
有了DMA技术的支持之后，网卡就可以直接区访问内核空间的内存，这样就可以实现内核空间和应用空间之间的零拷贝了，极大地提升传输性能
**零拷贝技术**
所谓的零拷贝是指将数据在内核空间直接从磁盘文件复制到网卡中，而不需要经由用户态的应用程 序之手，这样既可以提高数据读取的性能，也能减少核心态和用户态之间的上下文切换，提高数据 传输效率，下图展示了Kafka零拷贝的数据传输过程。数据传输的的过程就简化成了:

1. 操作系统将数据从磁盘中加载到内核空间的Read Buffer(页缓存区)中。
2. 操作系统之间将数据从内核空间的Read Buffer(页缓存区)传输到网卡中，并通过网卡将数据发
送给接收方。
3. 操作系统将数据的描述符拷贝到Socket Buffer中，Socket 缓存中仅仅会拷贝一个描述符过去，不
会拷贝数据到 Socket 缓存。

如果采用零拷贝技术(底层通过 sendfile 方法实现)，流程将变成下面这样，可以看到:只需 3次拷贝以及 2 次上下文切换，显然性能更高。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678169954449-e8486205-12d2-434e-b5bd-398452c5a17f.png#averageHue=%23f9f9f8&clientId=uf4e7d042-aef6-4&from=paste&height=608&id=u4b8e84c9&originHeight=1216&originWidth=1592&originalType=binary&ratio=2&rotation=0&showTitle=false&size=416183&status=done&style=none&taskId=u55969be0-8165-4603-acf5-d8208a5c4af&title=&width=796)
通过零拷贝技术，就不需要把 内核空间页缓存里的数据拷贝到应用层缓存，再从应用层缓存拷贝到 Socket 缓存了，两次拷贝都省略了，所以叫做零拷贝。这个过程大大的提升了数据消费时读取文件数据 的性能，Kafka 从磁盘读数据的时候，会先看看内核空间的页缓存中是否有，如果有的话，直接通过网 关发送出去。
**java实现**
java中的零拷贝是依靠 java.nio.channels.FileChannel 中的 transferTo(long position, long count , WritableByteChannel target)方法来实现的。
 
