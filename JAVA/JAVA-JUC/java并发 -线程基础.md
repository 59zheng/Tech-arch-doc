#### QA&S
- 线程有哪几种状态? 
- 分别说明从一种状态到另一种状态转变有哪些方式?
- 通常线程有哪几种使用方式?
- 基础线程机制有哪些?线程的中断方式有哪些?
- 线程的互斥同步方式有哪些? 如何比较和选择?
- 线程之间有哪些协作方式?
#### 线程状态转换
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689047471808-a76b712f-7fb5-4e2c-9678-55a904ae75f9.png#averageHue=%23fdfcfc&clientId=u36660e4c-51ff-4&from=paste&height=497&id=u814472b5&originHeight=994&originWidth=1794&originalType=binary&ratio=2&rotation=0&showTitle=false&size=226810&status=done&style=none&taskId=u96c9cbed-3bbc-45dd-9610-5ca22d85892&title=&width=897)
##### 新建（NEW）
创建后尚未启动。
##### 可运行（Runnable）
可能正在运行，也有可能正在等待 CPU 时间片。
包含了操作系统线程状态中的Runnable 和 Ready 
##### 阻塞（Blocking）
等待获取一个排它锁，如果其他线程释放了锁就会结束此状态。
##### 无限阻塞等待（Waiting）
等待其他线程显式的唤醒，否则不会被分配CPU 时间片

| 进入方法 | 退出方法 |
| --- | --- |
| 没有设置 Timeout 参数的 Object.wait（） 方法 | Object.notity（）/ Object.notityAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完成 |
| LockSupport.park（）方法 |  |

##### 限期等待（Timed Waiting）
无需等待其他线程显式的唤醒，在一定时间之后被系统自动唤醒。
调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。
调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。
睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。
阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

| 进入方法 | 退出方法 |
| --- | --- |
| Thread.sleep()方法 | 时间结束 |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕 |
| LockSupport.parkNanos() 方法 | - |
| LockSupport.parkUntil() 方法	 | - |


##### 死亡（Terminated）
可以是线程结束任务之后自己结束，或者产生了异常而结束
#### 线程使用方式
有三种使用线程的方法:

- 实现 Runnable 接口；
- 实现 Callable 接口；（Callable 可以有返回值，返回值通过 FutureTask 进行封装。）
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。
实现接口 VS 继承 
Thread实现接口会更好一些，因为:Java 不支持多重继承，因此继承了 Thread 类就无法继承其它类，但是可以实现多个接口；类可能只要求可执行就行，继承整个 Thread 类开销过大。

#### 基础线程机制
##### Executor
Executor 管理多个异步任务的执行，而无需程序员显式的管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。
主要有是那种 Executor：

- CachedThreadPool ：一个任务创建一个线程；
- FixedThreadPool ：所有的任务只能使用固定大小的线程；
- SingleThreadExecutor ： 箱单与大小为 1 的 FixedThreadPool

##### Deamon
守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。
当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。
main（）属于非守护线程。
使用setDaemon（）方法讲一个线程设置为守护线程。
```java
public static void main(String[] args) {
    Thread thread = new Thread(new MyRunnable());
    thread.setDaemon(true);
}
```




##### sleep（）
Thread.sleep(millisec)方法会休眠当前正在执行的线程，millisec 单位为毫秒。
sleep（）可能会抛出 InterruptedExecption，因为异常不能扩线程传播回 main（）中，因此必须在本地进程处理。线程中抛出的其他异常也同样需要再本地进行处理。
```java
public void run() {
    try {
        Thread.sleep(3000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```


##### yield（）
对静态方法 Thread.yield()的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其他线程来执行。该方法只是对象线程调度器的一个建议，而且也只是建议同时具有相同优先级的其他线程可以进行cpu执行的抢占。 

#### 线程中断
一个线程执行完毕之后会自动结束，如果原型过程中发生异常也会提前结束
##### InterruptedExecption 
通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、期待等待或者无限期等待状态，那么就会抛出 InterruptedExecption 从而提前结束该线程。但是不能中I/O 阻塞和 synchronized 锁阻塞。
```java
 private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(20000);
                log.info("sssssssss");
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            super.run();
        }
    }

    public static void main(String[] args) {
        MyThread1 myThread1 = new MyThread1();
        myThread1.start();
        myThread1.interrupt();
        
    }
```
会提前中断结束，不会继续执行接下来的语句。

##### interrupted
如果一个线程的run（）方法执行一个无限循环，并且没有执行 sleep（）等会抛出 InterruptedExecption 的操作，那么调用线程的 interrupt（）方法就无法使线程提前结束。
但是调用 interrupted（） 会设置线程的中断标记，此时调用 interrupted（） 方法会返回true 。因此可以在循环体中使用 interruped（）方法来判断线程中是否处于中断状态，从而提前结束线程。
```java
    private static class MyThread2 extends Thread {
        @Override
        public void run() {
            while (!interrupted()){
                log.info("while true always");
            }
        }
    }
```

##### Executor 的中断操作
调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow（）方法，则相当于调用每个线程的 interrupt（） 方法
如果只是想中断 Executor 中的一个线程，可以通过使用 submit（）方法来提交一个线程，它会返回一个 Future<?>对象，通过调用该对象的cancel(true) 方法就可以中断线程。
```java
Future<?> future = executorService.submit(() -> {
    // ..
});
future.cancel(true);
```
##### 线程的互斥同步
java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized 而另一个是 JDK 实现的 ReentrantLock
##### synchronized 
###### 同步一个代码块
```java
  public void  func1(){
        synchronized (this){
            for (int i = 0; i < 5; i++) {
                System.out.println("sssss"+i);
            }
        }
    }
```
使用 ExecutorService 执行两个线程，由于调用的同一个对象的同步代码块，因此两个线程会同步进行，当一个线程进入同步语句块时，另一个线程必须等待。
如果调用的是两个不同的对象，这时候两个线程不需要再进行同步等待的操作了。两个线程交叉执行。

###### 同步一个方法
```java
   public synchronized void func2(){
        
    }
```
与同步代码块一样，作用于单个对象。
###### 同步一个类
```java
  public void func() {
        synchronized (SynchronizedExample.class) {
            // ...
        }
    }
```
此时的作用域是整个类，也就是说两个线程调用同一个类的两个对象也会同步等待执行，同步执行。

###### 同步一个静态方法
```java
public synchronized static void fun() {
    // ...
}
```
效果等同于同步整个类
##### ReentrantLock
RenntrantLock 是 JUC 包中的锁
```java
  private Lock lock = new ReentrantLock();

    public void func() {
        lock.lock();
        try {
            for (int i = 0; i < 10; i++) {
                System.out.println("sssss" + i);
            }
        } finally {
            // 释放锁
            lock.unlock();

        }
    }
```
##### 比较

1. 锁的实现

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的

2. 性能

新版 synchronized 进行了很多优化 ，例如自旋锁等。 两者性能大致相同

3. 等待可中断

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。
ReentrantLock 可以中断，而synchronized 不可中断

4. 公平锁

公平锁是指多个线程竞争执行权的时，必须按照申请锁的时间依次获得锁。
synchronized 中锁是非公平的，ReentrantLock 中的锁默认也是非公平的，但是可以设置为公平。
new ReentrantLock(true)  构造传入。

5. 解锁定多个条件

一个ReentrantLock 可以同时绑定多个 Condition 对象。
##### 使用选择
优先 synchronized 因为 synchronized 是 jvm 原生的实现。 ReentrantLock 是 jdk 的视线，除非使用高级功能，并且 ReentrantLock 需要手动释放锁，否则死锁。
#### 线程之间的协作
当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其他部分之前完成，那么需要对线程进行协调。
##### join（）
在线程中调用另一个线程的 join（） 方法，会将当前线程挂起，而不是继续等待，直到目标线程结束。

##### wait() notify() notifyall()
调用 wait（）使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其他线程会调用notify（） 或者 notifyall（）来唤醒挂起的线程。
它们都属于 Object 的一部分，而不属于 Thread。
只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateException  
使用 wait（）挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其他线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyall（）来唤醒挂起的线程，造成死锁。
```java
   public  synchronized  void  before(){
        System.out.println("before");
        notifyAll();
    }
    public synchronized  void after(){
        try {
            wait();
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println("after");
    }

    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        WaitNotifyExample waitNotifyExample = new WaitNotifyExample();
        executorService.execute(()->waitNotifyExample.after());
        executorService.execute(()->waitNotifyExample.before());
        executorService.shutdown();
    }
```
```java
before
after
```
wait（） 和 sleep（） 区别

- wait（） 是 Object 的方法，而sleep（）是 Thread 的静态方法
- wait（）会释放锁，而 sleep 不会释放锁
##### await() signal() signalAll()
java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。
使用 Lock 来获取一个 Condition 对象。
```java
 private Lock lock =new ReentrantLock();
    private Condition condition=lock.newCondition();

    public  void  beforeCondition(){
        lock.lock();
        try {
            System.out.println("before");
            condition.signalAll();
        } finally {
            lock.unlock();
        }
    }
public void after() {
        lock.lock();
        try {
            condition.await();
            System.out.println("after");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }

```

