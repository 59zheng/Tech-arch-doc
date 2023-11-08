#### juc 大纲

![java-thread-x-juc-overview-1-u.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1690272304245-f00c101a-72a7-406a-b1d8-8ca8d0b01eeb.png#averageHue=%23fbfbfb&clientId=u672ae752-5082-4&from=paste&height=422&id=u55442462&originHeight=844&originWidth=1729&originalType=binary&ratio=2&rotation=0&showTitle=false&size=158559&status=done&style=none&taskId=u98231202-2317-4b3d-b04a-031be0a7dba&title=&width=864.5)
五个部分

- Lock 框架 和 Tools 类（把图片的这两个放一起理解）
- Collections：并发集合
- Atomic ：原子类
- Executors：线程池

#### Lock 框架和 Tools 类

##### 类结构总览

![java-thread-x-juc-overview-lock.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1690272427326-16a67a8a-4f43-4956-8339-cf5ffd476bb0.png#averageHue=%23efeeee&clientId=u672ae752-5082-4&from=paste&height=226&id=ubfe51494&originHeight=451&originWidth=764&originalType=binary&ratio=2&rotation=0&showTitle=false&size=11703&status=done&style=none&taskId=ue8b41bab-e43e-4c1a-8a68-2fb7e78f6b7&title=&width=382)

##### 接口：Condition *

> Condition 为接口类型，它将 Object 监视器方法（wait，notify和notifyAll）分解为截然不同的对象，以便于通过这些对象与任意 Lock 实现组合使用，为每个对象提供多个等待 set （wait-set）。其中，Lock 替代了 synchronized 方法和语句的使用，Codintion 替代了 Object 监视器方法的使用。可以通过 await（），signal（） 来休眠/唤醒 线程。


##### 接口：Lock

> Lock 为接口类型，Lock 实现提供了比使用 synchronized 方法和语句可获得更广泛的锁定操作，此实现允许更加灵活的接口，可以具有差别很大的属性，可以支持多个相关的 Condition 对象。

##### 接口：ReadWriteLock *

> ReadWriteLock 为接口类型，维护了一对相关的锁，一个用于只读操作，另一个用于写入操作，只要没有writer ，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。


##### 抽象类 ：AbstractOwnableSynchonizer

> AbstractOwnableSynchonizer 为抽象类，可以由线程以独占方式拥有的同步器。此类为创建锁和相关同步器（伴随着所有权的概念）提供了基础。AbstractOwnableSynchonizer 类本身不管理或使用此信息。但是，子类和工具可以使用适当维护的值帮助控制和监视访问以及提供诊断。


##### 抽象类（long）：AbstractQueuedLongSynchronizer

> AbstractQueuedLongSynchronizer 为抽象类，以 long 形式维护同步状态的一个 AbstractQueueSynchronizer 版本，此类具有的结构，属性和方法和 AbstractQueuedSynchronizer 完全相同，但所有与状态相关的参数和结果都定义为 long 而不是 int 。当创建需要 64位状态的多级别锁和屏障等同步器时，此类很有用。

##### 核心抽象类（int）：AbstactQueuedSynchronizer  *

> AbstactQueuedSynchronizer 为抽象类，其为实现依赖于 先进先出 （FIFO）等待队列的阻塞锁和相关同步器（信号量，事件，等等）提供了一个框架。此类的设计目标是成为依靠 单个原子 int 值来表示状态的大多数同步器的一个有用基础。


##### 锁常用类 ： LockSupport *

> LockSupport 为常用类，用来创建锁和其他同步类的基本线程阻塞原语。LockSupport 的功能和 “Thread中的 Thread.suspend() 和 Thread.resume() 有点类似”，LockSupport 中的 park（）和 unpark（）的作用分别是阻塞线程和接触阻塞线程。但是 park（）和 unpark（）不会遇到“Thread.suspend 和 Thread.resusnme 所可能引发的死锁”问题。

##### 锁常用类： ReentrantLock *

> ReentrantLock 为常用类，它是一个可重入的互斥锁 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大

##### 锁常用类 ： ReentrantReadWriteLock *

> ReentrantReadWriteLock 是读写锁接口 ReadWriteLock 的实现类，它包括 Lock 子类的ReadLock 和WriteLock 。ReadLock 是共享锁。WriteLock 是独占锁


##### 锁常用类：StampedLock *

> 它是 java8 在 java.util.concurrent.locks 新增的一个 API，StamoedLock 控制锁有三种模式（写，读，乐观读），一个 StampedLock 状态是由版本和模式两个部分组成，锁获取方法返回一个数字作为票据 stamp，它用相应的锁状态表示并控制访问，数字0表示没有写锁呗授权访问。在读锁上分为悲观锁和乐观锁。

##### 工具常用类： CountDownLatch  *

> CountDownLatch 为常用类，它是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或者多个线程一直等待。

##### 工具常用类： CyclicBarrier *

> CyclicBarrier 为常用类，其实一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点（common barrier point）。在涉及一组固定大小的线程的程序中，这些线程必须不时地相互等待，此时 CycliBarrier 很有用。因为该 barrier 在释放等待线程之后可以重用，所以称它为循环的 barrier。

##### 工具常用类： Phaser  *

> Phaser 是 JDK 7 新增的一个同步辅助类，可以实现 CyclicBarrier 和 CountDownLatch 类似的功能，而且它支持对人物的动态调整，并支持分层结构来达到更高的吞吐量

##### 工具常用类：Semaphore

> Semaphore 为常用类，其是一个计数信号量，从概念上讲，信号量维护了一个许可集，如有必要，在许可可用

##### 