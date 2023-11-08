### redis 
#### 基本数据机构
##### dict 哈希表字典
Redis的 dict 实现最显著的一个特点，就在于它的重哈希。它采用了一种 称为增量式重哈希(incremental rehashing，也叫渐进式重哈希)的方法， 在需要扩展内存时避免一次性对所有key进行重哈希，而是将重哈希操作分散到 对于dict的各个增删改查的操作中去。这种方法能做到每次只对一小部分key进 行重哈希，而每次重哈希之间不影响dict的操作
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686623708399-1734b4e9-53c3-408b-b27f-0e23f973e88d.png#averageHue=%23f8f1ee&clientId=ue34496fd-1746-4&from=paste&height=235&id=uc972accc&originHeight=470&originWidth=1582&originalType=binary&ratio=2&rotation=0&showTitle=false&size=228448&status=done&style=none&taskId=u37951955-6bb0-4229-892f-81f81c6fb3c&title=&width=791)
两张 hash 表 一张正常存，一张进行 rehash 
##### redis object 
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686643942179-2709d638-c8a3-42a4-a7e2-61bceff6c604.png#averageHue=%23f7efea&clientId=ue34496fd-1746-4&from=paste&height=403&id=ud8107c4c&originHeight=806&originWidth=1418&originalType=binary&ratio=2&rotation=0&showTitle=false&size=242840&status=done&style=none&taskId=u867c6776-65ce-41df-b135-68916451dec&title=&width=709)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686643906333-39a1741f-61a6-4e13-9e90-53d3b4e6db59.png#averageHue=%23f7f3f3&clientId=ue34496fd-1746-4&from=paste&height=608&id=u6b0d2e30&originHeight=1216&originWidth=2764&originalType=binary&ratio=2&rotation=0&showTitle=false&size=572762&status=done&style=none&taskId=u4f8b1991-ebdc-4f84-bbbe-5cc4cf121e2&title=&width=1382)
这里特别需要仔细察看的是encoding字段。对于同一个type，还可能对应 不同的encoding，这说明同样的一个数据类型，可能存在不同的内部表示方 式。而不同的内部表示，在内存占用和查找性能上会有所不同，下图展示了 redis 5种常见数据类型在不同编码方式下的数据结构:
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686643962741-fc36d56c-c0ee-4014-89ab-2a1435659629.png#averageHue=%23f2eae9&clientId=ue34496fd-1746-4&from=paste&height=434&id=u765a6797&originHeight=868&originWidth=1454&originalType=binary&ratio=2&rotation=0&showTitle=false&size=348343&status=done&style=none&taskId=u67893102-4b9b-4c79-9911-ffdac295504&title=&width=727)
##### zset 跳表
skiplist 本质上是链表结构，多层链表。
随机层数概念。  maxLevle :32。每一个插入操作的level 是随机出来的。
zet 结构变更 ziplist-> skiplist->dict
一个 dict + skiplist 组成
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686641762831-6ee57665-fdfa-4b0a-877a-47cc07e716c8.png#averageHue=%23f5e8e6&clientId=ue34496fd-1746-4&from=paste&height=536&id=uc9f0c791&originHeight=1072&originWidth=1408&originalType=binary&ratio=2&rotation=0&showTitle=false&size=270235&status=done&style=none&taskId=u7eaa5fea-74a3-4a43-a7c0-5ebd60efc3d&title=&width=704)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686641782078-d71d599d-cfab-46d6-8c85-af3ee0c42bff.png#averageHue=%23f7f5f5&clientId=ue34496fd-1746-4&from=paste&height=543&id=ucb9c0949&originHeight=1086&originWidth=3196&originalType=binary&ratio=2&rotation=0&showTitle=false&size=634420&status=done&style=none&taskId=u56aa2d5c-f9a4-4528-922e-9a23babd789&title=&width=1598)

#### rdis持久化 
rdb bgsave 的时候存储的数据是拷贝给子进程的吗？
Redis 持久化机制，AOF 很大怎么办
AOF 和RDB
##### RDB 快照
bgsave 通过fork一个子进程，专门用于写入 RDB 文件，避免了主线程的 阻塞，这也是 Redis RDB 文件生成的默认配，fork子进程负责持久化过程，阻 塞只会发生在fork子进程的时候。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686642150827-b3c3b337-b822-49c5-8d6a-eaf64120fe76.png#averageHue=%23f8f4f0&clientId=ue34496fd-1746-4&from=paste&height=620&id=uc0a689bf&originHeight=1240&originWidth=1252&originalType=binary&ratio=2&rotation=0&showTitle=false&size=366005&status=done&style=none&taskId=u31b9b62b-4c63-48d0-9106-0b51c5faf54&title=&width=626)
不会直接拷贝数据，内存如果超50%的话服务就挂了 fork 的时候会阻塞主进程。不是真正的copy数据是基于 cow 写时复制 技术做的
fork一个子进程，只有在父进程发生写操作修改内存数据时，才会真正去分 配内存空间，并复制内存数据，而且也只是复制被修改的内存页中的数据，并 不是全部内存数据。
fork采用操作系统提供的写实复制(Copy On Write)机制，就是为了避 免一次性拷贝大量内存数据给子进程造成的长时间阻塞问题，但fork子进程 需要拷贝进程必要的数据结构，其中有一项就是拷贝内存页表(虚拟内存 和物理内存的映射索引表)，这个拷贝过程会消耗大量CPU资源，拷贝完 成之前整个进程是会阻塞的，阻塞时间取决于整个实例的内存大小，实例 越大，内存页表越大，fork阻塞时间越久。拷贝内存页表完成后，子进程与 父进程指向相同的内存地址空间，也就是说此时虽然产生了子进程，但是 并没有申请与父进程相同的内存大小。那什么时候父子进程才会真正内存 分离呢?“写实复制”顾名思义，就是在写发生时，才真正拷贝内存真正的数据。 数据还是一份，只是读取操作不会影响，修改的时候会先copy 一下，版本号升一下。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686642387446-887eff50-df39-4ad1-b116-9956bc69cf84.png#averageHue=%23f1f7ed&clientId=ue34496fd-1746-4&from=paste&height=899&id=u62f8c76d&originHeight=1798&originWidth=3192&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1166077&status=done&style=none&taskId=u576a7546-8760-4fe1-80a1-70e31bc17bd&title=&width=1596)

##### AOF 日志
写后日志，Redis 先执行命令，把数据写入内存，然后才记录日志，追加的方式，写入。日志会特别大。
文件瘦身机制：aof重写机制。基于单个key 的多条操作记录，基于最终结果生成唯一一条命令。减少恢复的时候需要执行的语句条数。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686642748473-390bcd45-9d4a-417b-a636-f074e850246a.png#averageHue=%23f8edd2&clientId=ue34496fd-1746-4&from=paste&height=255&id=uc5cf461b&originHeight=510&originWidth=1516&originalType=binary&ratio=2&rotation=0&showTitle=false&size=324804&status=done&style=none&taskId=ue5e49b70-efbb-4d61-a70d-021d83ad050&title=&width=758)
aof 文件重写，不是分析 aof 文件的，是根据当前最新的数据，生成命令。
一个拷贝，两处日志
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686642941168-2f067959-121f-499e-9083-750cbc4f7e67.png#averageHue=%23f4f4e5&clientId=ue34496fd-1746-4&from=paste&height=847&id=u23a8fc58&originHeight=1694&originWidth=3270&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1303763&status=done&style=none&taskId=ub95ea662-7464-4c51-a858-39898deb64f&title=&width=1635)
数据拷贝也是用的 cow 写时复制技术
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686643009385-480205be-ffd6-405f-8cf1-6958be4aed22.png#averageHue=%23f7f4f2&clientId=ue34496fd-1746-4&from=paste&height=552&id=u738894bd&originHeight=1104&originWidth=1268&originalType=binary&ratio=2&rotation=0&showTitle=false&size=379653&status=done&style=none&taskId=ufd69cbb8-14e2-412f-818f-8fcbe94ce1b&title=&width=634)

##### 最佳实践
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686643115427-ca628ba2-3ce9-4635-9c14-adb438d9a46b.png#averageHue=%23f4f4f4&clientId=ue34496fd-1746-4&from=paste&height=260&id=u6b5e18a2&originHeight=520&originWidth=1462&originalType=binary&ratio=2&rotation=0&showTitle=false&size=126653&status=done&style=none&taskId=u8088ed65-a6b7-4d01-839e-0f42df03e49&title=&width=731)
快照多久做一次？
时间太长，出现问题丢失的数据太多，时间太短，频繁写磁盘会带来压 力，频繁fork子进程在fork时会阻塞主进程。
  最好的方式:增量快照，一次全量快照后，记录增量的数据。
Redis 4.0 中提出了一个混合使用 AOF 日志和内存快照的方法。简单来 说，内存快照以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间 的所有命令操作。这样一来，快照不用很频繁地执行，这就避免了频繁 fork 对 主线程的影响。而且，AOF 日志也只用记录两次快照间的操作，也就是说，不 需要记录所有操作了，因此，就不会出现文件过大的情况了，也可以避免重写 开销。
#### redis 删除策略
##### 过期删除
把过期时间加key 放到过期字典中

- 定时删除，创建过期时间的时候，同时创建一个定时器，定时扫描，消耗太大，不建议。
- 惰性删除， 在使用的时候检查一下，是否过期。Redis的惰性删除策 略由 db.c/expireIfNeeded 函数实现，所有键读写命令执行之前都会调用 expireIfNeeded 函数对其进行检查。过期key，不使用造成脏数据
- 定期删除，每隔一段时间，我们就对一些key进行检查，删除里面过期的key。由 redis.c/activeExpireCycle 函数实现，函数以一定的频率运行，每次运行时，都 从一定数量的数据库中取出一定数量的随机键进行检查，并删除其中的过期 键。 不是检查所有的。定期删除函数的运行频率，在Redis2.6版本中，规定每秒运行10次， 大概100ms运行一次。在Redis2.8版本后，可以通过修改配置文件redis.conf 的 hz 选项来调整这个次数。
- 最佳实践，定期删除+惰性删除。
##### 内存淘汰
虽然定期+惰性，但是如果定期删除漏掉了很多过期 key，然后你也没及时 去查询，也就没走惰性删除，此时会怎么样?如果大量过期 key 堆积在内存 里，导致 redis 内存块耗尽了，
  所以此时需要走内存淘汰机制!

- volatile-lru :设置了过期时间的key使用LRU算法淘汰; allkeys-lru :所有key使用LRU算法淘汰;
- volatile-lfu :设置了过期时间的key使用LFU算法淘汰; allkeys-lfu :所有key使用LFU算法淘汰;
- volatile-random :设置了过期时间的key使用随机淘汰; allkeys-random :所有key使用随机淘汰;
- volatile-ttl :设置了过期时间的key根据过期时间淘汰，越早过期越早淘汰;
- noeviction :默认策略，当内存达到设置的最大值时，所有申请内存的操作都会报错(如set,lpush等)，只读操作如get命令可以正常执行;
###### LRU
最近最少使用，使用时间排序，实现方式为链表
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686643736285-890351ee-2d0a-4f14-9dd2-01edd7921b97.png#averageHue=%23f9f2e0&clientId=ue34496fd-1746-4&from=paste&height=205&id=u0c3dfa97&originHeight=410&originWidth=1482&originalType=binary&ratio=2&rotation=0&showTitle=false&size=124551&status=done&style=none&taskId=ubcb8354d-9263-4813-a11d-b621fe6e55b&title=&width=741)
redis中是近LRU 随机选取 5个 排序。Redis中是随机 采样5个(可以修改参数maxmemory-samples配置)key，然后从中选择访问 时间最早的key进行淘汰，因此当采样key的数量与Redis库中key的数量越接 近，淘汰的规则就越接近LRU算法。但官方推荐5个就足够了，最多不超过10 个，越大就越消耗CPU的资源。
但在LRU算法下，如果一个热点数据最近很少访问，而非热点数据近期访问 了，就会误把热点数据淘汰而留下了非热点数据，因此在Redis4.x中新增了LFU 算法。
###### LFU
LFU(Least Frequently Used)表示最不经常使用，它是根据数据的历史 访问频率来淘汰数据，其核心思想是“如果数据过去被访问多次，那么将来被访 问的频率也更高”
LFU算法反映了一个key的热度情况，不会因LRU算法的偶尔一次被访问被 误认为是热点数据。
LFU算法的常见实现方式为链表:新数据放在链表尾部 ，链表中的数据按 照被访问次数降序排列，访问次数相同的按最近访问时间降序排列，链表满的 时候从链表尾部移出数据。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686644136683-de913a31-bd50-4f78-8201-0065aba009ae.png#averageHue=%23f7f0dd&clientId=ue34496fd-1746-4&from=paste&height=202&id=ubaa8950f&originHeight=404&originWidth=1438&originalType=binary&ratio=2&rotation=0&showTitle=false&size=151188&status=done&style=none&taskId=u18cd00a5-aa51-4724-a8f1-300c9f0ff7a&title=&width=719)
#### redis 高可用
集群选主， 主从复制流程， master选择策略，数据流向
##### 主从复制
启动多个 Redis 实例的时候，它们相互之间就可以通过 replicaof(Redis 5.0 之前使用 slaveof)命令形成主库和从库的关系，之后会 按照三个阶段完成数据的第一次同步。 提高了读的能力，没有提高写的能力。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686644298648-66d14987-bef1-4698-bf8f-e9b53fa7ccf6.png#averageHue=%23eef5e7&clientId=ue34496fd-1746-4&from=paste&height=329&id=u56aa8b13&originHeight=658&originWidth=1446&originalType=binary&ratio=2&rotation=0&showTitle=false&size=418171&status=done&style=none&taskId=u8317b4b2-0e80-4519-a37e-007187465b8&title=&width=723)
一旦主从库完成了全量复制，它们之间就会一直维护一个网络连接，主库 会通过这个连接将后续陆续收到的命令操作再同步给从库，这个过程也称为基 于长连接的命令传播，可以避免频繁建立连接的开销。
 **主从之间网络断开问题**
如果主从库在命令传播时出现了网络闪断，那么，从库就会和主库重新进 行一次全量复制，开销非常大。从 Redis 2.8 开始，网络断了之后，主从库会采 用增量复制的方式继续同步。全量复制是同步所有数据，而增量复制只会把主 从库网络断连期间主库收到的命令，同步给从库。
replbacklog，环形缓存区，记录主库的写的偏移量，和从库的写命令的偏移量。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686645261355-c29bdc6c-f69f-4e8a-89ae-f0885bf12f44.png#averageHue=%23eef4e6&clientId=ue34496fd-1746-4&from=paste&height=386&id=u67a23cd4&originHeight=772&originWidth=1516&originalType=binary&ratio=2&rotation=0&showTitle=false&size=411001&status=done&style=none&taskId=u153965ef-fec6-4016-9535-33836db7695&title=&width=758)
##### cluster 分片 集群
每个节点都有独立的读写能力，提高写能力
###### 一致性哈希算法
16384个哈希槽。key 偏移运算后，找到指定槽。多个节点瓜分 哈希槽，找到目标的节点。
###### 集群故障转移
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686645665041-e58beb50-fbf9-4268-ab2b-da38ea338d35.png#averageHue=%23f9f9f9&clientId=ue34496fd-1746-4&from=paste&height=500&id=ubc4aba0e&originHeight=1000&originWidth=1454&originalType=binary&ratio=2&rotation=0&showTitle=false&size=318637&status=done&style=none&taskId=ucbd88670-5078-46de-b241-4ed801f6b0d&title=&width=727)

- **故障检测：**

感知主节点故障，ping 命令 pong 
集群中的每个节点都会保存一份集群里所有节点的状态表，集群中的每个 节点都会定期的向其他节点发送 PING 消息，以此来检测对方是否下线。如果 再规定的时间内没有收到 PONG 消息的响应，就会将该节点标记为 疑似下线 (PFAIL)，集群中的各个节点会通过消息，来 交换 自己维护的节点状态表。 比如某个节点是在线、下线还是疑似下线。
如果集群里半数以上的主节点，都将某个主节点标记为疑似下线，那么这 个主节点将被标记为下线(FAIL)状态，将主节点标记为下线状态的节点将会 向集群广播一条消息，所有收到消息的节点，都会将节点标记为下线状态。

- **leader 选举**

master 有选举权利。过半原则。如果票数相同，递增纪元，重新选举。 redis 优化，发选票的时候有一个随机的时间差，时间差是基于复制的偏移量来算的，与master 的数据的差量有关。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686646261894-e44dac9f-3fc3-4de6-bf85-ad15f065b2ad.png#averageHue=%23ececec&clientId=ue34496fd-1746-4&from=paste&height=231&id=u8814e583&originHeight=462&originWidth=1516&originalType=binary&ratio=2&rotation=0&showTitle=false&size=152862&status=done&style=none&taskId=ue2abc760-181e-4686-ba63-692a2f3481e&title=&width=758)

- **转移过程**

新选举出的主节点会首先会执行SLAVEOF no one，停止对旧主节点的复 制动作。然后会将旧主节点的槽位指派给自己。最后向集群发送广播消息，让 集群中的其他节点知道新的主节点已经接管所有的槽位，旧主节点已经完成下 线动作。
  这样就完成了整个的故障转移过程，新的主节点将开始接受与自己负责的槽位有关的数据
### MongoDB vs Mysql
MongoDB和MySQL的区别?MongoDB为什么读写快?
#### 区别，特点
MySQL与MongoDB都是开源的常用数据库，但是MySQL是传统的关系型 数据库，MongoDB则是非关系型数据库，也叫文档型数据库，是一种NoSQL的 数据库。它们各有各的优点，关键是看用在什么地方。所以我们所熟知的那些 SQL语句就不适用于MongoDB了，因为SQL语句是关系型数据库的标准语言。
##### MySQL特点

1. 在不同的引擎上有不同的存储方式。
2. 查询语句是使用传统的sql语句，拥有较为成熟的体系，成熟度很高。 3. 开源数据库的份额在不断增加，mysql的份额也在持续增长。
3. 缺点就是在海量数据处理的时候效率会显著变慢。
##### MongoDB
非关系型数据库(nosql ),属于文档型数据库。先解释一下文档的数据 库，即可以存放xml、json、bson类型系那个的数据。这些数据具备自述性， 呈现分层的树状数据结构。数据结构由键值(key=>value)对组成。

1. 存储方式:虚拟内存+持久化。
2. 查询语句:是独特的MongoDB的查询方式。
3. 适合场景:事件的记录，内容管理或者博客平台等等。
4. 架构特点:可以通过副本集，以及分片来实现高可用。
5. 数据处理:数据是存储在硬盘上的，只不过需要经常读取的数据会被加载到内存中，将数据存储在物理内存中，从而达到高速读写。
##### mongodb 会比mysql快

1. 首先是内存映射机制，数据不是持久化到存储设备中的，而是暂时存储 在内存中，这就提高了在IO上效率以及操作系统对存储介质之间的性能 损耗。(毕竟内存读取最快)
2.  其次，NoSQL并不是不使用sql，只是不使用关系。没有关系的存在， 就比如每个数据都好比是拥有一个单独的存储空间，然后一个聚集索引 来指向。搜索性能一定会提高的。
3. mongodb 天生支持分布式部署，如果数据量足够大可以使用 mongodb的分片集群，这样也是mongodb比较快的一个原因
##### 使用场景区别
MongoDB 是一种文档型数据库，由于它不限制数据量和数据类型， 它是高容量环境下最合适的解决方案。由于 MongoDB 具备云服务需要的水平 可伸缩性和灵活性，它非常适合云计算服务的开发。另外，它降低了负载，简 化了业务或项目内部的扩展，实现了高可用和数据的快速恢复。
### ES
es的倒排索引?
想理解倒排索引，首先先思考一个问题，获取某个文件夹下所有文件名中 包含Spring的文件

- 1)确定要搜索的文件夹 
- 2)遍历文件夹下所有文件 
- 3)判断文件名中是否包含Spring

这种思维可以理解为是一种正向思维的方式，从外往内，根据key找 value。这种方式可以理解为正向索引。
而ElasticSearch为了提升查询效率，采用反向思维方式，根据value找 key。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686646656323-bd4456bd-5fd0-41e7-b6c3-1991ee44804b.png#averageHue=%23c8b79b&clientId=ue34496fd-1746-4&from=paste&height=310&id=u51e21854&originHeight=620&originWidth=1438&originalType=binary&ratio=2&rotation=0&showTitle=false&size=222663&status=done&style=none&taskId=u93afb988-a004-4f98-9a41-3dc7f675ac7&title=&width=719)
