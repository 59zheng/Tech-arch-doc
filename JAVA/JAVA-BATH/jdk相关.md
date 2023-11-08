#### Object类
所有类的基类，默认继承Object类
##### Object植入时机
两个阶段   javac 进行编译  jvm 运行。 编译阶段和运行阶段
jdk6： 可以显示出来  extends Object 编译器在编译阶段进行了织入
无论jdk版本：都是编译器在编译阶段进行了织入
##### 7个 Native 方法（本地方法：java本身没有实现，具体实现由C/C++ 完成系统层面）
由于java是跨平台的，直接操作硬件部方便，需要适配多种系统，所以不直接提供此类方法，而是调用系统本地的方法。

- equals

Object 类中 ==运算符和equels方法是等价的。都是比较两个对象的地址值。
重写equals 方法为啥要重写 hashcode 方法、。。 如果两个对象 equals 方法相等，那么hashcode 一定相等。hashcode 相等，equals 不一样相等。可能会hash冲突。
基于 hashcode 集合 hashmap hashset hahstable 。。。。 之类的集合中。是先比较的hashcode 再比较equals，这样会增加集合的效率、
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676685486674-4cb5c5d1-3713-4cbf-be5f-793ea115c6e7.png#averageHue=%23f9f8f8&clientId=u629600f7-c02b-4&from=paste&height=214&id=u16663761&originHeight=427&originWidth=1609&originalType=binary&ratio=1&rotation=0&showTitle=false&size=416414&status=done&style=none&taskId=ub30d0241-3db3-4ec6-a4e4-4ba3e5bdab3&title=&width=804.5)

- hashCode

hashcode 不是内存地址。
##### hashcode 所处位置

##### Clone 方法
对象复制：浅拷贝 和深拷贝 主要指 赋值一个对象时拷贝的深度
浅拷贝： 复制对象的引用，之前对象的个别属性变更，新的对象也会随着变更
深拷贝： 拷贝所有的属性，如果属性是对象，可以对这些对象进行深拷贝
##### finalize 方法				 						 					
当 GC 确定不再有对该对象的引用时，GC 会调用对象的 finalize() 方法来清除回收。
Java VM 会确保一个对象的 finalize() 方法只被调用一次，而且程序中不能直接调用 finalize() 
方法。
finalize() 方法通常也不可预测，而且很危险，一般情况下，不必要覆盖 finalize() 方法。 
#### ArrayList类
数组实现-- 随机访问，元素有序可以重复--可变成数据，定长数组实现。支持扩容				
(1) ArrayList 是一种变长的集合类，基于定长数组实现。 
初始化长度： 初始化创建数组的长度：  

- jdk7 进行了初始化，长度为10 
- jdk 8的话创建了空数组。长度为空  在第一次add 的时候 会进行初始化 默认10 

(2)  ArrayList 允许空值和重复元素，当往 ArrayList 中添加的元素数量大于其底层数组容量时，其 会通过**扩容**机制重新生成一个更大的数组。 
(3)  ArrayList 底层基于数组实现，所以其可以保证在 O(1) 复杂度下完成随机查找操作。 
(4)  ArrayList 是非线程安全类，并发环境下，多个线程同时操作 ArrayList，会引发不可预知的异常 或错误。  

- arrayList可以存放null。
- arrayList本质上就是一个elementData数组。
- arrayList区别于数组的地方在于能够自动扩展大小，其中关键的方法就是gorw()方法。 
- arrayList由于本质是数组，所以它在数据的查询方面会很快，而在插入删除这些方面，性能下降很 多，有移动很多数据才能达到应有的效果

扩容机制
 创建一个长度的新数组 将旧数组 copy 到新数组
#### LinkedList类
定义：LinkedList是一种可以在任何位置进行高效地插入和移除操作的有序序列，它是基于双向链表实现的。
双向链表结构--查询慢，增删快
jdk 1.8中是尾插 二分查找 
#### HashMap 类
HashMap基于哈希表的Map接口实现，是以key-value存储形式存在，即主要用来存放键值对。 HashMap的实现不是同步的，这意味着它不是线程安全的。它的key、value都可以为null。此外， HashMap中的映射不是有序的。
JDK1.7 HashMap数据结构:数组 + 链表
JDK1.8 HashMap数据结构:数组 + 链表 / 红黑树
哈希表=数据+哈希实现。
##### 哈希表
Hash表也称为散列表，也有直接译作哈希表，Hash表是一种根据关键字值(key - value)而直接进行 访问的数据结构。也就是说它通过把关键码值映射到表中的一个位置来访问记录，以此来加快查找的速 度。在链表、数组等数据结构中，查找某个关键字，通常要遍历整个数据结构，也就是O(N)的时间级， 但是对于哈希表来说，只是O(1)的时间级
哈希表，它是通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度。这个映射函数叫做 散列函数，存放记录的数组叫做散列表，只需要O(1)的时间级
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676964678989-4b9de26e-f088-4dac-bcc2-549b0ac09229.png#averageHue=%23fefefd&clientId=u8c4c51c1-3e1f-4&from=paste&height=403&id=uc98e9da1&originHeight=806&originWidth=2380&originalType=binary&ratio=2&rotation=0&showTitle=false&size=359978&status=done&style=none&taskId=u0b02d1a0-75c6-4e9b-b9c0-be05c293661&title=&width=1190)
jdk 1.8 链表转红黑树的条件

- 当前链表的长度大于等于8
- 数组长度大于等于64

默认的初始化容量 16  1<<4 
加载因子 0.75
在put 的时候进行初始化操作。 寻址，存值
1.7头插法，1.8尾插法
##### 当前容量为啥是 2的N次幂
如果最大的容量不是 2的 n次幂的话，就会转换成比这个数大的2的N次幂 最近
算下标使用的事  与运算（位运算）。比取模运算速度快，
是 2的N次幂，是为了支持 与运算
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676968097007-5f6b395e-8399-4e89-b2fe-51e79cb85820.png#averageHue=%23f8f7f9&clientId=u6365b520-540a-4&from=paste&height=186&id=u1ebe8650&originHeight=372&originWidth=1734&originalType=binary&ratio=2&rotation=0&showTitle=false&size=263228&status=done&style=none&taskId=ub1b08874-3a92-4edf-8cdb-3b6cc957fef&title=&width=867)
##### 什么是加载因子，为啥加载因子是0.75
触发扩容的阈值，及容量到达总量*加载因子 个数的时候会触发扩容，不止是数组的长度，发生哈希冲突挂载在链表上的值也算。 对时间和空间利用的论证得出的理论最优值 0.75，均衡值，空间利用率和hash冲突中的平衡
为1（大） 的话 优点：空间利用率最大，缺点：更容易产生hash冲突，
##### hash为什么要进行扰动
9次扰动，
提高离散型，让hashCode 的高位也参与运算。
jdk1.7 的扰动： 不端右移 
jdk 1.8 的扰动： 高8位和低8位 进行异或运算。
hashMap key为null 的时候会固定挂载到 数组的 0位置 ，多个形成链表
put（） 如果 key 相同，会返回修改之前的值 
为什么 总使用 string 作为 hashMap 的key 保证修改之后hashcode 不同即可。如果object 类内部属性变更之后会导致 hash 变更。所以一般使用string 
##### 扩容逻辑：
阈值 8 链表转红黑树
阈值 6 红黑树退化为链表

#### Synchronized
#### ConcurrentHashMap
##### 为什么不推荐使用hashTable
原因锁的粒度太大了
hashTable 底层实现也是数组加链表，put get 都是一 synchronized 优势的 锁的粒度是整体的数组。单个线程写的时候会锁整个 hashTable 几乎不支持并发操作，效率太低 所有线程共用同一把锁
##### concurrentHashMap 1.7 分段锁思想
采用数组+分段锁实现的
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676981995713-8d0f90e8-734b-4705-a309-9e191625df89.png#averageHue=%23faf9f9&clientId=uc36047b0-5946-4&from=paste&height=514&id=uf78d9afc&originHeight=1027&originWidth=1679&originalType=binary&ratio=2&rotation=0&showTitle=false&size=849931&status=done&style=none&taskId=u279f069b-f34e-443d-9721-1fda402596f&title=&width=839.5)
 ![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676982574550-c35df916-fcd5-4b4b-843f-2490832c9eab.png#averageHue=%23fdfcfc&clientId=uc36047b0-5946-4&from=paste&height=547&id=u96e3da4f&originHeight=1094&originWidth=1938&originalType=binary&ratio=2&rotation=0&showTitle=false&size=667472&status=done&style=none&taskId=u9bf87532-ea48-4cdb-9fdd-0c1a1f00bf2&title=&width=969)
cmap：当前支持的最大并发是16  concurrnetHashMap 1.7 中 segment 数据不可扩容，初始化之后，默认容量为16
##### concurrnetHashMap 中key和value 都不能为null
防止歧义，二义性。 并发多线程的情况下，不允许产生歧义。在put 时候返回null。不能判断是存在null值还是为空返回的null、

##### concurrnetHashMap 1.8  数据+链表/红黑树  放弃 分段锁，重新使用 Synchronized +大量cas
1.7中使用分段锁：并发线程数有限制。分段锁的粒度还是大，降低锁的粒度
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1676987170057-e240b962-4b72-49e6-8f2e-c630dc188310.png#averageHue=%23f7f3f2&clientId=uc36047b0-5946-4&from=paste&height=465&id=ub57b8a5a&originHeight=930&originWidth=2061&originalType=binary&ratio=2&rotation=0&showTitle=false&size=663805&status=done&style=none&taskId=ue3b46771-c0c1-4162-b0e0-3976e5f4ac4&title=&width=1030.5)

