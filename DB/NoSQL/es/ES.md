开源搜索引擎  基于 lucene 分词器（作为 jar 包形式引入项目使用的 非中间件）
es 是对象 lucene 的包装，提供api 封装之前的繁琐的调用
基于 JSON 的分布式搜索和分析引擎  java 语言开发 快速，实时的存储，所有，分析的大规模的数据。（实时概念，插入的同时就能实时查出来了） 
特点：
近实时性，全文检索，分布式，集群规模，开箱即用，不支持事务
elk，异构数据同步，多数据源数据同步，指标监控收集，全文检索，事件数据和指标。
kibana 可视化工作（elk 工具链）
相关性算法（匹配的结果和提供的相关性排序），处理错误的拼写，给予自动提示，使用统计信息（统计分析）
restful http 协议暴露接口，无语言相关性了。

FST 算法 类似于树形的书法 词么分成首个字 然后依次找下个字这种  ， 敏感词过滤一般用于
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1683075626987-a80d7f35-25d4-44a1-98b2-75286f94ae09.png#averageHue=%23f0f0f0&clientId=u8ab87e34-8f42-4&from=paste&height=397&id=u67ae86b8&originHeight=794&originWidth=1636&originalType=binary&ratio=2&rotation=0&showTitle=false&size=200078&status=done&style=none&taskId=ue1f80272-fa3a-46b9-8497-ae61a25d33f&title=&width=818)
### 基本概念（逻辑概念）
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1683075726321-a7ce226c-d881-4d58-ae23-fd298db9d778.png#averageHue=%23f0f0f0&clientId=u8ab87e34-8f42-4&from=paste&height=492&id=ud5f85344&originHeight=984&originWidth=1652&originalType=binary&ratio=2&rotation=0&showTitle=false&size=320657&status=done&style=none&taskId=ucad1f0a7-75e9-40c7-b6ed-a9393b306db&title=&width=826)
### 物理概念
Elasticsearch是一个分布式系统，其数据会分散存储到不同的节点上，为了实现这一点，需要将 每个index中的数据划分到不同的块中，然后将这些数据块分配到不同的节点上存储
#### 集群
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1683076284958-ce93c93f-dc08-40c3-bb9b-3a18cb2272d1.png#averageHue=%23e6ead1&clientId=u8ab87e34-8f42-4&from=paste&height=270&id=u4007a80d&originHeight=540&originWidth=1558&originalType=binary&ratio=2&rotation=0&showTitle=false&size=295312&status=done&style=none&taskId=u8fca612c-938b-4a51-9b11-fa768453629&title=&width=779)
名字相同一个集群，默认elasticsearch 增加node 的时候基于这个标识添加到对应的集群
#### 节点(node)
一个节点是集群中的一个服务器，作为集群的一部分，它存储数据，参与集群的索引和搜索功能
#### 分片(Shards)
分片的存在是为了解决单个索引大量文档的存储问题、以及搜索是响应慢等问题。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1683076379071-aeafd87a-e2e6-419a-8f63-56c2e5e16700.png#averageHue=%23b97f40&clientId=u8ab87e34-8f42-4&from=paste&height=406&id=u6f782194&originHeight=812&originWidth=1566&originalType=binary&ratio=2&rotation=0&showTitle=false&size=262260&status=done&style=none&taskId=u07671ac4-fa34-408d-83ce-d41add15dad&title=&width=783)
会产生数据丢失的问题，解决这个问题增加 副本的结构  副本作用，保证主分片挂掉之后部分分片数据丢失，或者查询不可见的问题
两个副本不能放在 同一个 node es系统控制 同一个分片的副本只存在一份。
一般 分片确定就不会改变了，可以使用扩容缩容改变，倍数方式。
#### 段（segment）
基于lucene 里边的概念，segment 是最小的数据单元
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1683076739463-66ee0027-4841-4188-ab0e-ef4ffe1080bb.png#averageHue=%23bdeba2&clientId=u8ab87e34-8f42-4&from=paste&height=173&id=uc503c569&originHeight=345&originWidth=1629&originalType=binary&ratio=2&rotation=0&showTitle=false&size=215222&status=done&style=none&taskId=u61c5fe90-9ffc-4599-a445-fe6a9f765ee&title=&width=814.5)
只会增加，不会删除，删除操作只会把需要删除的数据标记一下，查询的时候过滤掉这部分数据
#### Master 节点
拥有成为master 节点资格的节点，即 master-eligible node 
主要职责是负责集群层面的相关操作，管理集群变更，创建删除索引，跟踪哪些节点是集群的一部分了，决定哪些分片分配给祥光的节点等
#### 仅协调节点
仅协调，凑数的，仅仅是路由的作用。
本质上，仅协调节点的行为就像智能负载均衡器，通过从数据和符合主节点的节点卸载协调节点角 色，仅协调节点可以使大型集群受益，他们加入集群并接收完整的集群状态，就像其他每个节点一样， 他们使用集群状态将请求直接路由到适当的地方。
#### 脑裂问题：
选票超半数成功当选。
### 初始化设计
数据库设计，必须慎重，因为变更数据结构只能基于扩容缩容来修改，需要进行数据迁移的问题，初期的设计非常重要，索引设计
索引设计，基于时间周期创建索引，每个索引又有分片。时间相关性大的话基于 别名进行查询。可以使用索引模版来周期性的创建索引，配置啥的形成一个模版，需要使用的时候调用一下就行。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1683078228890-0c9a8f4c-70ec-44d2-91da-d9278e3c07f8.png#averageHue=%23f6f6f6&clientId=u8ab87e34-8f42-4&from=paste&height=315&id=u8f2cd8db&originHeight=630&originWidth=1651&originalType=binary&ratio=2&rotation=0&showTitle=false&size=473994&status=done&style=none&taskId=u57c9187a-30ff-42c0-9475-761ea1c663f&title=&width=825.5) 
### 分词器

- character filter  过滤无意义的字符
- tokenizer    词拆分
- token filters  将切分的单词添加、删除或者改变

keyword 分词器，不分词。。。
触发场景： 创建或者更新文档的时候; 搜索文档的时候进行分词
