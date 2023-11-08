![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689059904693-30339dd5-d7e6-40b0-9932-d041c88248e9.png#averageHue=%23fdfdfd&clientId=uff8bb567-d894-4&from=paste&height=1210&id=ue32811a8&originHeight=2420&originWidth=2746&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1114948&status=done&style=none&taskId=udbe7a037-307f-414f-9ad0-e7f5ea1493a&title=&width=1373)
#### 乐观锁 VS 悲观锁
> 乐观锁和悲观锁是一种广义上的概念，体现了看待线程同步的不同角度。在Java和数据库中都有此概念对应的实际应用。

概念。对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。java 中，synchroized 关键字 和Lock 的实现类都是悲观锁。
乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入，如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（例如报错或者自动重试）。。
乐观锁在 Java 中是使用无锁编程实现的，最常采用的是 Cas 算法，java 原子类的自增操作就是通过 CAS 自旋实现的。
![java-lock-2.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689127179224-71f6c031-ba26-4df3-932e-df5220ad2d96.png#averageHue=%23f6f3ee&clientId=ucafec178-de12-4&from=paste&height=731&id=u1318b495&originHeight=1462&originWidth=1898&originalType=binary&ratio=2&rotation=0&showTitle=false&size=104259&status=done&style=none&taskId=u28043df0-2071-4c16-ba6a-a42e48028a6&title=&width=949)
悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。
乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅度提升。

#### 自旋锁 VS 适应性自旋锁
> 自旋锁前提知识

阻塞或唤醒一个 Java 线程 需要操作系统切换 CPU 状态来完成，这种状态切换需要耗费处理器时间。如果同步代码中的内容过于简单，状态切换消耗的时间有可能比用户代码执行的时间还要长。
很多场景中，同步资源的锁定时间很短，为了这一小段时间去切换线程，线程挂起和恢复现场的花费可能会让系统得不偿失，如果物理机有多个处理器，能够让两个或以上的线程同时并行执行，就可以让后面的那个请求锁不放弃cpu 的执行时间，看看持有锁的线程是否会很快释放锁。
![java-lock-4.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689127860699-99d2bba6-6c6c-468d-9978-c3fa6a9852ab.png#averageHue=%23fcfcfc&clientId=ucafec178-de12-4&from=paste&height=638&id=uf778cb16&originHeight=1276&originWidth=1320&originalType=binary&ratio=2&rotation=0&showTitle=false&size=46569&status=done&style=none&taskId=u61989225-ed0c-4a2a-bc29-84df24bce27&title=&width=660)
自旋锁本身是有缺点的，它不能代替阻塞。自旋等待虽然避免了线程切换的开销，但它要占用处理器时间。如果锁被占用的时间很短，自旋等待的效果就会非常好。反之，如果锁被占用的时间很长，那么自旋的线程只会白浪费处理器资源。所以，自旋等待的时间必须要有一定的限度，如果自旋超过了限定次数（默认是10次，可以使用-XX:PreBlockSpin来更改）没有成功获得锁，就应当挂起线程。
自旋锁的实现原理同样也是CAS，AtomicInteger中调用unsafe进行自增操作的源码中的do-while循环就是一个自旋操作，如果修改数值失败则通过循环来执行自旋，直至修改成功。
#### 无锁 VS 偏向锁 VS 轻量级锁 VS 重量级锁
偏向锁通过对比Mark Word解决加锁问题，避免执行CAS操作。而轻量级锁是通过用CAS操作和自旋来解决加锁问题，避免线程阻塞和唤醒而影响性能。重量级锁是将除了拥有锁的线程以外的线程都阻塞。

#### 公平锁 VS 非公平锁
公平锁是指多个线程按照申请锁的顺序来获取锁，线程直接进入队列中排队，队列中的第一个线程才能获得锁。公平锁的优点是等待锁的线程不会饿死。缺点是整体吞吐效率相对非公平锁要低，等待队列中除第一个线程以外的所有线程都会阻塞，CPU唤醒阻塞线程的开销比非公平锁大。
非公平锁是多个线程加锁时直接尝试获取锁，获取不到才会到等待队列的队尾等待。但如果此时锁刚好可用，那么这个线程可以无需阻塞直接获取到锁，所以非公平锁有可能出现后申请锁的线程先获取锁的场景。非公平锁的优点是可以减少唤起线程的开销，整体的吞吐效率高，因为线程有几率不阻塞直接获得锁，CPU不必唤醒所有线程。缺点是处于等待队列中的线程可能会饿死，或者等很久才会获得锁。


#### 可重入锁 VS 非可重入锁
可重入锁又名递归锁，是指在同一个线程的外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提对象是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。Java 中 ReentrantLock 和 synchroized 都是可重入锁，可重入锁的一个优点是可以一定程度的避免死锁。
#### 独享锁（排它锁） VS 共享锁
独享锁也叫排它锁，是指该锁一次只能被一个线程锁持有，如果线程T对A加上排它锁后，则其他线程不能在对A加任何类型的锁。获得排它锁的线程既能读数据又能修改数据。JDK 的synchroized 和 juc 中的 Lock 都是排它锁。
共享锁是指该锁可以被多个线程所持有。如果线程T对数据A加上共享锁，则其他线程只能对A再加共享锁，不能加排它锁，获取共享锁的线程只能读取数据，不能修改数据。
![java-lock-15.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689128987920-aca16380-08ef-448c-8841-3275fe0d8ffb.png#averageHue=%23fdfcfc&clientId=ucafec178-de12-4&from=paste&height=538&id=ue5d5827e&originHeight=1076&originWidth=1850&originalType=binary&ratio=2&rotation=0&showTitle=false&size=145556&status=done&style=none&taskId=u63af72f2-ba0a-4b48-bf93-f31ba54e356&title=&width=925)
ReentrantReadWriteLock有两把锁：ReadLock和WriteLock，由词知意，一个读锁一个写锁，合称“读写锁”。再进一步观察可以发现ReadLock和WriteLock是靠内部类Sync实现的锁。Sync是AQS的一个子类，这种结构在CountDownLatch、ReentrantLock、Semaphore里面也都存在。
在ReentrantReadWriteLock里面，读锁和写锁的锁主体都是Sync，但读锁和写锁的加锁方式不一样。读锁是共享锁，写锁是独享锁。读锁的共享锁可保证并发读非常高效，而读写、写读、写写的过程互斥，因为读锁和写锁是分离的。所以ReentrantReadWriteLock的并发性相比一般的互斥锁有了很大提升。

