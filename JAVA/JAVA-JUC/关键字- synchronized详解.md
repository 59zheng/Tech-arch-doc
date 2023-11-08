> C程序代码中可以直接使用操作系统提供的互斥锁来实现同步块的互斥访问及线程的阻塞及唤醒等工作。 JAVA 除了提供 Lock API 外还在语言层面上提供了 synchronized 关键字来实现同步互斥原语。

#### QA & S

- Synchronized可以作用在哪里? 分别通过对象锁和类锁进行举例。
- Synchronized本质上是通过什么保证线程安全的? 分三个方面回答：加锁和释放锁的原理，可重入原理，保证可见性原理。
- Synchronized有什么样的缺陷? Java Lock是怎么弥补这些缺陷的。
- Synchronized和Lock的对比，和选择?Synchronized在使用时有何注意事项?
- Synchronized修饰的方法在抛出异常时,会释放锁吗?
- 多个线程等待同一个Synchronized锁的时候，JVM如何选择下一个获取锁的线程?Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?
- 我想更加灵活的控制锁的释放和获取(现在释放锁和获取锁的时机都被规定死了)，怎么办?
- 什么是锁的升级和降级? 什么是JVM里的偏斜锁、轻量级锁、重量级锁?
- 不同的JDK中对Synchronized有何优化?

#### synchronized 使用
需要注意的点

- 一把锁只能同时被一个线程获取，没有获取锁的线程只能等待
- 每个实例都对应有自己的一把锁（this），不同实例之间互不影响；例如；锁对象是*.class 以及 synchronized 修饰的是 static 方法的时候，所有对象公用同一把锁
- synchroized 修饰的方法，无论方法正常执行完毕还是抛出异常，都会释放锁
##### 对象锁
包括方法锁（默认锁的对象是 this，目前的实例对象）和同步代码块锁（自己指定锁对象）

- 代码块形式：手动指定锁对象，可以是 this ，也可以是自定义的锁
```java
        @Override
        public void run() {
            // 同步代码块形式——锁为this,两个线程使用的锁是一样的,线程1必须要等到线程0释放了该锁后，才能执行
            synchronized (this) {
                System.out.println("我是线程" + Thread.currentThread().getName());
                try {
                    Thread.sleep(3000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + "结束");
            }

        }
```

- 方法锁形式：synchroized 修饰普通方法，锁对象默认为this
```java
public synchronized void method() {
        System.out.println("我是线程" + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "结束");
    }
```
##### 类锁
synchronized 修饰静态的方法或指定锁对象为 class 对象
#### synchronized 原理分析
##### 加锁和释放锁的原理
javac 反编译出来的字节码
![java-thread-x-key-schronized-x1.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689140638995-240f6fb0-0638-4c2a-8fb2-c7285e5381a7.png#averageHue=%23f9f9f9&clientId=ucb0729d8-4d62-4&from=paste&height=749&id=u73050d33&originHeight=1498&originWidth=2340&originalType=binary&ratio=2&rotation=0&showTitle=false&size=55479&status=done&style=none&taskId=ud4ba862c-da8a-4c57-a2bc-4c8fa0d29f6&title=&width=1170)
Mointornter 和 Monitorexit 指令，会让对象在执行的时候，使其锁计数器加1 或者减1 。每一个对象在同一时间只与一个 monitor（锁）相关联，而一个monitor 在同一时间也只能被一个线程获得，一个对象在尝试获得与这个对象相关的Monitor 锁的所有权的时候，monitorenter 指令会发生以下情况之一：

- monitor 计数器为0 ，意味着目前没有被获得，那这个线程就会立刻获得然后把锁计数器加1，一旦+1，别的线程再想获取，就需要等待了。
- 如果这个 monitor 已经拿到了这个锁的所有权，又重入了这把锁，那锁计数器会累加，变成2，随着重入的次数，会一直增加。
- 这把锁已经被别的线程获取了，等待锁释放

monitorexit 指令： 释放对于monitor 的所有权，释放过程很简单，就是将 monitor 的计数器减1，如果减完之后，计数器不是0，则代表刚才是重入进来的，当前线程还继续持有这把锁的所有权，如果计数器变成0，则代表当前线程不再拥有该monitor 的所有权，即释放锁。
![java-thread-x-key-schronized-2.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689141129872-3fc963da-690f-4680-8f1d-5f0e543ebf5f.png#averageHue=%23f8f8f8&clientId=ucb0729d8-4d62-4&from=paste&height=131&id=u2944ee95&originHeight=261&originWidth=700&originalType=binary&ratio=2&rotation=0&showTitle=false&size=7030&status=done&style=none&taskId=ua8310151-416e-4738-af29-869c2ee7516&title=&width=350)
任意线程对 Object 的访问，首先要获得 Object 的监视器，如果获取失败，该线程就进入同步状态，线程状态变为 BLOCKED ，当 Object 的监视器占有者释放后，在同步队列中的线程就会有机会重新获取该监视器。

##### 可重入原理：加锁次数计数器
在同一个锁程中，每个对象拥有一个 monitor 计数器，当线程获取该对象锁喉，monitor 计数器就会加一，释放锁后就会讲 monitor 计数器减一，线程不需要再次获取同一把锁。

##### 保证可见性的原理：内存模型和 happens-before 规则
synchronized 的  happens-before 规则，即监视器锁规则：对同一个监视器的解锁，happens-before 在对该监视器的加锁
![java-thread-x-key-schronized-3.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689141514227-31a7e86f-b928-4f9e-90a6-f5ef03185ce6.png#averageHue=%23f9f8f4&clientId=ucb0729d8-4d62-4&from=paste&height=315&id=ubd7efa37&originHeight=629&originWidth=650&originalType=binary&ratio=2&rotation=0&showTitle=false&size=15316&status=done&style=none&taskId=ud12f91fb-f9dc-4988-9110-67fcdd1ac71&title=&width=325)
在图中每一个箭头连接的两个节点就代表之间的happens-before关系，黑色的是通过程序顺序规则推导出来，红色的为监视器锁规则推导而出：线程A释放锁happens-before线程B加锁，蓝色的则是通过程序顺序规则和监视器锁规则推测出来happens-befor关系，通过传递性规则进一步推导的happens-before关系。现在我们来重点关注2 happens-before 5，通过这个关系我们可以得出什么?
根据happens-before的定义中的一条:如果A happens-before B，则A的执行结果对B可见，并且A的执行顺序先于B。线程A先对共享变量A进行加一，由2 happens-before 5关系可知线程A的执行结果对线程B可见即线程B所读取到的a的值为1。
#### jvm 中所的优化
在jvm 中 monitorenter 和 monitorexit 字节码依赖于底层的操作系统 的 Mutex Lock 来实现的，但是由于使用的 Mutex Lock 需要讲当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的；然而在现实中的大部分情况下，同步方法是运行在单线程环境（无锁竞争环境）如果每次都调用 Mutex Lock 那么将严重的影响程序的性能。子啊 jdk 1.6 中对所进行大量优化，如锁粗化（Lock Coarsening），锁消除（Lock Elimination），轻量级锁（Lightweight Locking），偏向锁（Biased Locking），适应性自旋（Adaptive Spinning）等技术来减少锁操作的开销。

- 锁粗化：减少必要的紧连在一起的unlock，lock 操作，将多个连续的锁扩展成一个范围更大的锁。
- 锁消除：通过运行时JIT 编译器的逃逸分析来消除一些没有在当前同步块之后被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本的Stack 上进行对象空间的分配（同时还可以减少 Heap 上的垃圾收集开销）
- 轻量级锁：这种锁实现的背后基于一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态（即单线程执行环境），在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的事在 monitorenter 和 monitorexit 中只需要依赖一条CAS 原子指令就可以完成锁获取及释放。当存在锁竞争的情况下，执行cas指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒。
- 偏向锁：为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令，因为CAS原子指令虽然对于重量级锁来说开销比较小，但还是存在非常客观的本地延迟
- 适应性自旋：当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与montor 相光联的操作系统重量级锁（mutex semaphore）前会进入忙等待（Spinning）然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor 关联的 semaphore （即互斥锁）进入阻塞状态
##### 锁的类型
无锁，偏向锁，轻量级锁，重量级锁，随着竞争情况逐渐升级。锁可以升级但是不可以降级，目的是为了提供获取锁和释放锁的效率。
##### 自旋锁与自适应自旋锁

- 自旋锁

在等待锁的过程中并不直接进行线程阻塞等待的状态，而是自旋不断地尝试获取锁，自旋等待的次数默认为10次 ，可以通过 -XX：PreBlockSpin 来更改。

- 自适应自旋锁

自旋等待的次数不再固定，自旋的时间不固定，而是有上一次在同一个锁上的自旋时间及锁的拥有者的状态来决定。如果在同一个锁对象上，自旋等待刚刚成功获取过锁，并且持有锁的线程正在运行中，那么JVM会认为该锁自旋获取到锁的可能性很大，会自动增加等待时间。比如增加到100 此循环。相反，如果对于某个锁，自旋很少成功获取锁，那再以后要获取这个锁时将可能直接省略掉自旋过程，以避免浪费处理器资源。有了自适应自旋锁，JVM 对程序的状态预测会越来越准确，JVM 会更聪明
##### 锁消除
锁消除是指虚拟机即时编译器再运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行消除。锁消除的主要判定依据来源于逃逸分析的数据支持。意思就是：JVM会判断再一段程序中的同步明显不会逃逸出去从而被其他线程访问到，那JVM就把它们当作栈上数据对待，认为这些数据是线程独有的，不需要加同步。此时就会进行锁消除。
当然在实际开发中，我们很清楚的知道哪些是线程独有的，不需要加同步锁，但是在Java API中有很多方法都是加了同步的，那么此时JVM会判断这段代码是否需要加锁。如果数据并不会逃逸，则会进行锁消除。比如如下操作：在操作String类型数据时，由于String是一个不可变类，对字符串的连接操作总是通过生成的新的String对象来进行的。因此Javac编译器会对String连接做自动优化。在JDK 1.5之前会使用StringBuffer对象的连续append()操作，在JDK 1.5及以后的版本中，会转化为StringBuidler对象的连续append()操作。

##### 锁粗化
原则上，在加同步锁的时候，尽可能的将同步块的作用范围限制到尽量小的范围（只在共享数据的实际作用域中才进行同步，这样是为了使得需要操作数量尽可能变小。在存在锁同步竞争中，也可以使得等待锁的线程尽早的拿到锁）。
但是如果存在连串的一系列操作都对同一个对象反复加锁和解锁，甚至加锁操作出现在循环体中的，那即使没有线程竞争，频繁的进行互斥同步操作也会导致不必要的性能操作。
```java
public static String test04(String s1, String s2, String s3) {
    StringBuffer sb = new StringBuffer();
    sb.append(s1);
    sb.append(s2);
    sb.append(s3);
    return sb.toString();
}
```
在上述的连续append()操作中就属于这类情况。JVM会检测到这样一连串的操作都是对同一个对象加锁，那么JVM会将加锁同步的范围扩展(粗化)到整个一系列操作的 外部，使整个一连串的append()操作只需要加锁一次就可以了。
##### 轻量级锁
在jdk1.6中引入的轻量级锁，需要注意的是轻量级锁并不是替代重量级锁的，而是对在大多数情况下同步块并不会有竞争出现提出的一种优化。可以减少重量级锁对线程阻塞带来的线程开销。从而提高并发性能。
HotSpot 虚拟几种对象头的内存布局，对象头中（Object Header）中存在两部分，第一部分用于存储对象自身的运行时数据，HashCode 、GC Age 、锁标记位、是否位偏向锁。一般是32 位或者 64位（视操作系统的位数而定）。称为 Mark Word ，实现轻量级锁和偏向锁的关键。另外一部分存储是指向方法区对象类型的指针（Klass Point），如果对象是数组的话，还会有一个额外的部分存储数据的长度。

- 轻量级 加锁的过程

在线程执行同步块之前，JVM 会在当前线程的栈帧中创建一个名为锁记录（Lock Record）的空间，用于存储锁对象目前的 Mark Word 的拷贝（JVM 会将对象头中的 Mark Word 拷贝到锁记录中，Displaced Mark Word）此时线程堆栈和对象头的状态如下：
![java-thread-x-key-schronized-5.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689154228278-c143088d-8b58-4de9-abf2-443ea064e95a.png#averageHue=%23ecf0f6&clientId=u918c2f86-8bbc-4&from=paste&height=192&id=ucd7a39a3&originHeight=384&originWidth=729&originalType=binary&ratio=2&rotation=0&showTitle=false&size=4173&status=done&style=none&taskId=u58143915-53a9-4e9a-a4a3-25cbbeb7851&title=&width=364.5)
图示：如果当前对象没有被锁定，那么锁标记为01的状态，JVM 在执行当前线程的时候，首先会将当前线程栈帧中创建锁记录 Lock Record 的空间用于存储锁对象目前的 Mark Word 的拷贝。
然后，虚拟机使用CAS 操作将标记字段 Mark Word 拷贝到锁记录中，并且将 Mark Word 更新为指向 Lock Record 的指针。如果更新成功了，那么这个线程就拥有了该对象的锁，并且对象Mark Word 中最后的 2 Bit 00 ,表示次对象处于轻量级锁定状态.
![java-thread-x-key-schronized-6.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689154433778-70f216ed-da6b-45f8-9bfb-5657ce68b29a.png#averageHue=%23e9eef5&clientId=u918c2f86-8bbc-4&from=paste&height=175&id=ud09b5092&originHeight=349&originWidth=737&originalType=binary&ratio=2&rotation=0&showTitle=false&size=5487&status=done&style=none&taskId=u773e07a6-9cdd-41a3-a7c5-3fe6099fda3&title=&width=368.5)
but 这个更新操作失败的话，JVM 会检查当前的 Mark Word 中是否存在指向当前线程的栈帧的指针，如果有，说明该锁被其他线程抢占了，如果有两条以上的线程竞争同一个锁，那轻量级锁就不再有效，直接膨胀为重量级锁，没有获得锁的线程会被阻塞。此时，锁的标记位为 10.Mark Word 中存储的指向重量级锁的指针。
轻量级解锁时，会使用原子的CAS 操作将 Displaced Mark Word 替换回对象头中，如果成功，则表示没有发生竞争关系，如果失败，表示当前锁存在竞争关系。锁就会膨胀成重量级锁。两个线程同时争夺锁，导致锁膨胀的流程图如下：
![java-thread-x-key-schronized-7.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689154673494-90d40876-8d99-4ea2-8474-13b6c684a008.png#averageHue=%23d3d2d2&clientId=u918c2f86-8bbc-4&from=paste&height=263&id=u01580dd2&originHeight=525&originWidth=541&originalType=binary&ratio=2&rotation=0&showTitle=false&size=22741&status=done&style=none&taskId=uef888681-f30b-4e0e-923b-f85ba880520&title=&width=270.5)

##### 偏向锁
> 大多实际环境下，锁不仅不存在多线程竞争，而且总是由同一个线程多次获取，那么在同一个线程反复获取所释放锁的过程中，其中没有锁的竞争

当线程访问呢同步代码块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的id，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁。只需要测试一下对象头的 Mark Word 里是否存储着只想当前线程的偏向锁。如果成功，便是线程已经获取到了锁。
![java-thread-x-key-schronized-8.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689229959878-003f441e-58a4-48c7-bb69-62594d6ac37d.png#averageHue=%23f1f1f1&clientId=uebfbb29f-b498-4&from=paste&height=199&id=u135258eb&originHeight=398&originWidth=849&originalType=binary&ratio=2&rotation=0&showTitle=false&size=67761&status=done&style=none&taskId=u04fcb794-04da-4eff-90a2-b982d1210b4&title=&width=424.5)
###### 偏向锁的撤销
偏向锁使用了一种等待竞争出现才会释放锁的机制。所以当其他线程尝试获取偏向锁时，持有偏向锁的线程才会释放锁。但是偏向锁的撤销需要等待全局安全点（就是当前线程没有正在执行的字节码）。它会首先暂停拥有偏向锁的线程，让你后检查持有偏向锁的线程是否活着。如果线程不处于活动状态，直接将对象头设置为无锁状态。如果线程活着，JVM会遍历栈帧中的锁记录，栈帧中的锁记录和对象头要么偏向其他线程，要么恢复到无锁状态或者标记对象不适合作为偏向锁。
![java-thread-x-key-schronized-9.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689230173354-7b043519-5f11-4ad6-a247-949533a08c94.png#averageHue=%23e2e2e1&clientId=uebfbb29f-b498-4&from=paste&height=323&id=u0afd4bc6&originHeight=646&originWidth=582&originalType=binary&ratio=2&rotation=0&showTitle=false&size=22449&status=done&style=none&taskId=uccae949a-0c34-40e2-a498-044a7d6f413&title=&width=291)

##### 锁的优缺点对比
| 锁 | 优点 | 缺点 | 使用场景 |
| --- | --- | --- | --- |
| 偏向锁 | 加锁和解锁不需要CAS操作，没有额外的性能消耗，和执行非同步方法相比仅存在纳秒级的差距 | 如果线程间存在锁竞争，会带来额外的撤锁的消耗 | 适用于只有一个线程访问同步块的场景 |
| 轻量级锁 | 竞争的线程不会阻塞，提供了响应速度 | 如线程始终得不到锁竞争的线程，使用自旋会消耗cpu性能 | 追求响应时间，同步块执行速度非常快 |
| 重量级锁 | 现成竞争不适用自旋，不会消耗cpu | 线程阻塞，响应时间缓慢，在多线程下，频繁的获取释放锁，会带来巨大的性能消耗 | 追求吞吐量，同步块执行速度较长 |

#### synchronized 和 Lock
##### synchronized 的缺陷

- 效率低：锁的释放情况少，只有代码执行完毕或者异常结束才会释放锁；试图获取锁的时候不能设定超时，相对而言，Lock 可以中断和设置超时
- 不够灵活：加锁和释放的时机单一，每个锁仅有一个单一的条件（某个对象），相对而言，读写锁更加灵活
- 无法知道是否成功获得锁，相对而言，Lock可以拿到状态，成功获取锁的情况，获取失败的情况。

##### Lock 解决相应问题
Lock 类主要的4个方法

- lock(): 加锁
- unlock(): 解锁
- tryLock(): 尝试获取锁，返回一个boolean值tryLock(long,TimeUtil): 尝试获取锁，可以设置超时

Synchronized加锁只与一个条件(是否获取锁)相关联，不灵活，后来Condition与Lock的结合解决了这个问题。
多线程竞争一个锁时，其余未得到锁的线程只能不停的尝试获得锁，而不能中断。高并发的情况下会导致性能下降。ReentrantLock的lockInterruptibly()方法可以优先考虑响应中断。 一个线程等待时间过长，它可以中断自己，然后ReentrantLock响应这个中断，不再让这个线程继续等待。有了这个机制，使用ReentrantLock时就不会像synchronized那样产生死锁了
、
