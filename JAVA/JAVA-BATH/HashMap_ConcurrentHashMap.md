#### HashCode
| hashcode 不是内存地址。

##### 存储的位置

1. 无锁状态：存在在对象头 markword 中
2. 如果是偏向锁或者轻量级锁的时候，在获取hashcode 的时候会进行锁升级，升级到重量级锁再生成hashcode：存储在 该对象所对应的 Object monitor 对象中

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676686479987-07bc8675-594b-4c77-81d1-73dbde85fcc7.png#averageHue=%23f4e5d9&clientId=u629600f7-c02b-4&from=paste&height=412&id=ue37d226c&originHeight=824&originWidth=1648&originalType=binary&ratio=1&rotation=0&showTitle=false&size=437908&status=done&style=none&taskId=u2b1e60f9-f880-488a-af95-5b94821560d&title=&width=824)
markword  8字节   
类型指针占4个字节是因为开启了指针压缩，不开启的话是占用8个字节
markword 内存结构
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676944645448-105d0351-5325-4c05-9224-ca91fd087cdf.png#averageHue=%237b8f75&clientId=u5063bb86-c835-4&from=paste&height=319&id=u1ed367ba&originHeight=638&originWidth=1840&originalType=binary&ratio=2&rotation=0&showTitle=false&size=606237&status=done&style=none&taskId=u2fd88836-ca44-4499-bb0e-2c96d200b9e&title=&width=920)
##### HashCode 生成算法

hashcode 生成：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676945700234-26f181de-4ea7-49e2-ab8c-1c76f5b8a203.png#averageHue=%23efefed&clientId=u5063bb86-c835-4&from=paste&height=626&id=u9f1bc832&originHeight=1252&originWidth=2028&originalType=binary&ratio=2&rotation=0&showTitle=false&size=640089&status=done&style=none&taskId=u67070cad-d4c2-40d6-92fd-bec4c4cdbcb&title=&width=1014)
##### 哈希扰动
增加 hashCode 的离散型 具体有不同的实现、
Xorshift 异或随机算法
#### HashMap
##### jdk1.7
数据+链表

头插法
##### jdk1.8
数据+链表/红黑树
尾插法
#### ConcurrentHashMap
##### jdk1.7
##### jdk1.8
