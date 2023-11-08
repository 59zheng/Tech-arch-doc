#### 基本使用
消息投递方式 是推送的形式，比较特殊相比较kafka  rocket mq 之类都是消费者主动拉取的形式
RabbitMQ 与 AMQP 遵循相同的模型架构  
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678170574708-5ec7d5d2-d310-434c-a83a-53e09d862871.png#averageHue=%23f2ebe3&clientId=ud7ecbde6-35b2-4&from=paste&height=220&id=ucdb490cd&originHeight=440&originWidth=1612&originalType=binary&ratio=2&rotation=0&showTitle=false&size=268530&status=done&style=none&taskId=ucd3975ac-b556-440b-bef9-7fd95fa2ab5&title=&width=806)
##### 概念：

- publisher 生产者
- message 

消息组成有点特殊，和kafka 不同，需要有指定的header 和可变body，消息头存储的元数据： exchange_name 交换机的名字，路由键（RountingKey）和其他可选的一些配置。

- Exchange

交换机 负责接收来自生产者的消息，并将消息路由（推送）到一个或者多个的队列中，如果路由不到，则返回给生产者或者直接丢弃，取决于 exchange 的mandatory 属性 true 返回 false 丢弃。

- Binding key： 交换器与队列通过 BindingKey 建立绑定关系 （消费者基于队列与交互器 构建关系，并且注册上去，队列用于存积消息 ） 与 bindingKey 相等的话投递，
- RountingKey ： 路由规则（目标消费者的信息，或者匹配规则）
- Queue  ： 消息队列的载体 一个消息会投递到一个或者多个队列中。
- Consumer ：消费者
- connection 用于传递消息的 TCP 连接
- Channel 消息通道，客户端的每个连接里，可建立多个 channel，每个channel 代表一个会话任务。

RabbitMQ 采用类似NIO（非阻塞式IO）的设计，通过Channel 来复用TCP连接，并确保每个Channel的隔离性，就像是拥有独立的Connection 连接，当数据量不是很大时，采用连接复用技术可以避免创建过多的TCP连接造成昂贵的性能开销。

- Virtual Host 虚拟主机，一个broker 可以有多个vhost，用作不同用户的权限分离
- Broker ： 消息队列服务实体。

##### exchange 类型
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678173691444-6b8637f9-fd3b-4179-8bd3-41baf548345a.png#averageHue=%23efefef&clientId=ud7ecbde6-35b2-4&from=paste&height=289&id=uea58d386&originHeight=578&originWidth=1662&originalType=binary&ratio=2&rotation=0&showTitle=false&size=220241&status=done&style=none&taskId=u121499d8-6a7b-485a-8acf-13c5a4761ee&title=&width=831)

- 直连交换机 （一对一）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678173739157-4e036fd7-e8d7-4bc7-8073-eebde061e4d8.png#averageHue=%2342403f&clientId=ud7ecbde6-35b2-4&from=paste&height=441&id=ub623d84a&originHeight=882&originWidth=1668&originalType=binary&ratio=2&rotation=0&showTitle=false&size=397990&status=done&style=none&taskId=u7768e860-f5dd-4fad-a864-a7606eaa779&title=&width=834)
binding key== routing key 必须匹配

- 扇形交换机

消息路由绑定在它身上的所有队列进行发送，不理会没有绑定的路由
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678173953895-d928c42b-21d4-481c-9ca4-91581d7cd0a1.png#averageHue=%23787776&clientId=ud7ecbde6-35b2-4&from=paste&height=621&id=u8ecb2594&originHeight=1242&originWidth=1660&originalType=binary&ratio=2&rotation=0&showTitle=false&size=500002&status=done&style=none&taskId=ua7addbbf-afc2-4547-8dc4-e46b2aea44a&title=&width=830)

- 主题交换机 

基于消息的 routing key 与绑定到该交换器的队列的 pattern 进行匹配，路由消息到一个或多个队列中，常用于复杂的发布/订阅场景，当出现多消费者/应用的场景，消费者选择性地接收消息时，应该考 虑使用 topic exchange
**约束条件**

   1. binding key 中可以存在两种特殊字符 “_” 与“#”，用于做模糊匹配，其中 “_” 用于匹配一个单词， “#”用于匹配多个单词(可以是零个)
   2. routing key 为一个句点号 “.” 分隔的字符串(我们将被句点号 “. ” 分隔开的每一段独立的字符串称 为一个单词)，如“stock.usd.nyse”、“nyse.vmw”、“quick.orange.rabbit”
binding key 与 routing key 一样也是句点号 “.” 分隔的字符串

- 头 交换机(不常用)   不适用 bingkey 匹配使用 head 信息进行推送，与直连交换机相同

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678174134357-c19f0133-6526-4275-aca8-0d5ca8be28e6.png#averageHue=%23eeeeee&clientId=ud7ecbde6-35b2-4&from=paste&height=417&id=uefef03bb&originHeight=834&originWidth=1678&originalType=binary&ratio=2&rotation=0&showTitle=false&size=343776&status=done&style=none&taskId=u072270b5-f4ff-4fc4-ad62-2068a166bf4&title=&width=839)
##### 延迟消息
###### 概念：
###### 死信队列：
一般来说，producer将消息投递到broker或者直接到queue里了，consumer从queue取出消息进 行消费，但某些时候由于特定的原因导致queue中的某些消息无法被消费，这样的消息如果没有后续的 处理，就变成了死信，有死信，自然就有了死信队列;
只有rabbit mq 里边才用死信的概念。 rabbit mq 是消息推送的形式，所以会有指定时间内消费者无法确认的消息 ack ，消费者 拒接也会进行死信。只有拒接了，并且不允许重新进行当前队列。 可用队列最大长度，满了直接进入死信队列

使用场景：

   - 消费者对消息使用了 basicReject 或者 basicNack 回复，并且 requeue 参数设置为 false ,即不 再将该消息重新在消费者间进行投递 
   - 消息在队列中超时，RabbitMQ可以在单个消息或者队列中设置 TTL 属性
   -  队列中的消息已经超过其设置的最大消息个数

死信交换器不是默认的设置，这里是被投递消息被拒绝后的一个可选行为，是在创建队列的时进行声明的，往往用在对问题消息的诊断上。
死信交换器仍然只是一个普通的交换器，创建时并没有特别要求和操作，在创建队列的时候，声明该交换器将用作保存被拒绝的消息即可，相关的参数是 x-dead-letter-exchange 。
需要配置死信交换机，以及死信的队列，死信的路由key，正常队列过期时间（以队列为单位）

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1678351746978-e1d79fec-c1df-47a0-b87a-fb513ae7d195.png#averageHue=%23fcfaf7&clientId=uad46194b-f3aa-4&from=paste&height=403&id=u3079d2d1&originHeight=806&originWidth=1558&originalType=binary&ratio=2&rotation=0&showTitle=false&size=307242&status=done&style=none&taskId=u191fcd97-a49e-4728-98fe-fdcfd60ec1c&title=&width=779)

###### ttl：
TTL 是RabbitMQ中一个消息或者队列的属性，表明 ，单位是毫秒，换句话说，如果一条消息设置了TTL属性或者进入了设置TTL属性的队列，那么这条消息如果在TTL设置的时间内没有被消费，则会成为“死信”，如果不设置TTL，表示消息永远不会过期，如 果将TTL设置为0，则表示除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃。 java 配置方式：有被投递到该队列的消息都最多不会存活超过30s
```
@Bean
public Queue taxiOverQueue() {
    Map<String, Object> args = new HashMap<>(2);
    args.put("x-message-ttl", 30000);
    return QueueBuilder.durable(TAXI_OVER_QUEUE).withArguments(args).build();
}
```
消息消费失败被拒绝，重复消费，会到队列头，重试到指定次数后，到死信队列。
##### 消息可靠性保障

