#### 为什么需要多线程
众所周知，CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

- CPU 增加了缓存，以均衡与内存的速度差异；// 导致 可见性问题
- 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；// 导致 原子性问题
- 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。// 导致 有序性问题
#### 线程不安全示例
如果多个线程对一个共享数据进行访问而不采取同步操作的话，那么操作的结果是不一致的
```java
public class ThreadUnsafeExample {

    private int cnt = 0;

    public void add() {
        cnt++;
    }

    public int get() {
        return cnt;
    }
}
```
```java
public static void main(String[] args) throws InterruptedException {
    final int threadSize = 1000;
    ThreadUnsafeExample example = new ThreadUnsafeExample();
    final CountDownLatch countDownLatch = new CountDownLatch(threadSize);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < threadSize; i++) {
        executorService.execute(() -> {
            example.add();
            countDownLatch.countDown();
        });
    }
    countDownLatch.await();
    executorService.shutdown();
    System.out.println(example.get());
}
```
```java
979 // 结果总是小于1000
```


#### 并发出现问题的根源：并发三要素
##### 可见性：cpu缓存引起
可见性：一个线程对共享变量的修改，另一个线程能够立刻看到
##### 原子性：分时复用引起
原子性：即一个操作或者多个操作，要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。
##### 有序性：重排序引起
有序性：即程序执行的顺序按照代码的先后顺序执行。
主要是指令重排
在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

- 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
- 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
- 内存系统的重排序。由于处理器使用缓存和读 / 写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

从 java 源代码到最终实际执行的指令序列，会分别经历下面三种重排序：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1688970742669-165cc72f-62bf-4bdf-853b-f85b8734231a.png#averageHue=%23efeded&clientId=u7019cd28-2e48-4&from=paste&height=84&id=u6063e740&originHeight=168&originWidth=1072&originalType=binary&ratio=2&rotation=0&showTitle=false&size=22469&status=done&style=none&taskId=u74befbf8-00f8-4743-9c57-8a929d2e142&title=&width=536)
上述的1 属于编译器重排序，2和3属于处理器重排序。这些重排序都可能会导致多线程程序出现内存可见性的问题。对于编译器，JMM的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器排序都要禁止）。对于处理器重排序，JMM的处理器重排规则会要求 java 编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers（障碍复数） ， intel 称之为 memory fence（围栏））指令，通过内存屏障指令来禁止特定类型的处理器重排序。


#### java是怎么解决并发问题的：JMM （java 内存模型）
##### 理解的第一个维度：核心知识点
JMM 本质上可以理解为，java 内存模型规范 了 jvm 如何提供按需禁用缓存和编译优化的方法。具体来说这些方法包括：

- volatile、synchronized 和 final 三个关键字
- Happens-Before 规则
##### 理解的第二维度：可见性，有序性，原子性

- 原子性

java中，基本数据类型的变量的读取和赋值操作都是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行：
```java
x = 10;        //语句1: 直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中
y = x;         //语句2: 包含2个操作，它先要去读取x的值，再将x的值写入工作内存，虽然读取x的值以及 将x的值写入工作内存 这2个操作都是原子性操作，但是合起来就不是原子性操作了。
x++;           //语句3： x++包括3个操作：读取x的值，进行加1操作，写入新的值。
x = x + 1;     //语句4： 同语句3
```
只有语句1是具备原子性的。只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相关赋值不是原子操作）才是原子操作
> java 内存模型只保证了基本读写和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过synchronized 和Lock 来实现。由于 synchronized 和Lock 能够保证在任一时刻只有一个线程执行该代码块，那么自然不存在原子性的问题。

- 可见性

java提供了 volatile 关键字来保证可见性。
当一个共享变量被 volatile 修饰时，它会保证修改的值会立刻更新到主存，当有其他线程需要读取时，它会去内存中读取新值。
而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。
> 通过 synchronized 和Lock 也能够保证可见性，synchronized 和 Lock 能保证同一个线程获取锁然后执行同步代码，并且在释放锁之前将对变量的修改刷新到主存当中，因此可以保证可见性。

- 有序性

在Java里面，可以通过volatile关键字来保证一定的“有序性”（具体原理在下一节讲述）。另外可以通过synchronized和Lock来保证有序性，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。当然JMM是通过Happens-Before 规则来保证有序性的。

##### Happens-Before 规则
上面提到了可以用 volatile 和 synchronized 来保证有序性。除此之外，JVM 还规定了先行发生原则，让一个操作无需控制就能先于另一个操作完成。

1. 单一线程原则  Single Thread rule

在一个线程内，在程序前面的操作先行发生于后面的操作。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1688976533344-524f4be3-d71b-49bf-affc-1d60f0cc12b7.png#averageHue=%23989998&clientId=u1f02f7a2-4007-4&from=paste&height=299&id=u45f67666&originHeight=598&originWidth=678&originalType=binary&ratio=2&rotation=0&showTitle=false&size=92460&status=done&style=none&taskId=ufc4128f8-7846-45a1-822d-3108cf893a0&title=&width=339)

2. 管程锁定规则  Monitor Lock Rule

一个 unlock 操作先行发生于后面对同一个锁的 lock 操作
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1688976518131-b8b3972c-ee94-49ef-821a-3d279eee7af4.png#averageHue=%23fdfdfc&clientId=u1f02f7a2-4007-4&from=paste&height=361&id=u0a8c70c9&originHeight=722&originWidth=1252&originalType=binary&ratio=2&rotation=0&showTitle=false&size=152425&status=done&style=none&taskId=u4269a8ad-daef-499d-b9c3-70ebe67b523&title=&width=626)

3.  Volatile 变量规则 Volatile Variable Rule

对一个 volatile 变量的写操作先行发生于后面对这个变量的读操作
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1688976656798-b77f8288-0da7-4368-a879-c89321f1b8ff.png#averageHue=%23575857&clientId=u1f02f7a2-4007-4&from=paste&height=380&id=u172f7d73&originHeight=760&originWidth=1474&originalType=binary&ratio=2&rotation=0&showTitle=false&size=192756&status=done&style=none&taskId=u59ca76d2-0d14-4cd1-9933-3ab55cce5e0&title=&width=737)

4. 线程启动规则 Thred Start Rule

Thread 对象的 start（） 方法调用咸鱼发生于此线程的每一个动作
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1688976719456-069512c7-af08-4b15-8429-a5e2a63c71a8.png#averageHue=%23e4e5e0&clientId=u1f02f7a2-4007-4&from=paste&height=330&id=u0197f8fb&originHeight=660&originWidth=1240&originalType=binary&ratio=2&rotation=0&showTitle=false&size=148321&status=done&style=none&taskId=u9a055207-7d2e-426a-9938-ef0ee634571&title=&width=620)

5. 线程加入规则 Thread Join Rule

Thread 对象的结束先行发生于 join（） 方法返回
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1688976775176-ed1266a4-4c3b-4ded-b29f-7530502409a8.png#averageHue=%23cecfcb&clientId=u1f02f7a2-4007-4&from=paste&height=321&id=u502d53a1&originHeight=642&originWidth=1254&originalType=binary&ratio=2&rotation=0&showTitle=false&size=163932&status=done&style=none&taskId=u893c028c-dbb8-46e5-8bf1-fe99a158457&title=&width=627)

6. 线程中断规则 Trhead Interruption Rule 

对线程 interrupt（）方法调用先行发生于被中断线程的代码检测到中断事件之前发生，可以通过 interrupted（）方法检测到是否有中断发生。

7. 对象终结原则 Finalizer Rule 

一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize（）方法的开始。

8. 传递性 Transitivity

如果操作A先行发生于操作B，操作 B 先行发生于操作C，那么操作 A 先行发生于操作 C。

#### 线程安全：不是一个非真既假的命题
一个类可以被多个线程安全调用时就是线程安全的。
线程安全不是一个非真既假的命题，可以讲共享数据按照安全程度的强弱规则分成一下五类：不可变，绝对线程安全，相对线程安全，线程兼容和线程对立
##### 不可变
不可变（Immutable）的对象一定是线程安全的。
类型：

- final 关键字修饰的基本数据类型
- String 
- 枚举类型
- Number 部分子类，如 Long 和 Double 等数值包装类型，BigInterger 和 BigDecimal 等大数据类型。但同为 Number 的原子类 AtomicInterger 和 AtomicLong 则是可变的。
- 对于集合类型，可以使用 Collection.unmodifiableXXX() 方法来获取一个不可变的集合
```java
    public static void main(String[] args) {
        Map<String, Integer> map = new HashMap<>();
        Map<String, Integer> unmodifiableMap = Collections.unmodifiableMap(map);
        unmodifiableMap.put("a", 1);
    }
```
   Collections.unmodifiableXXX() 先对原始的集合进行拷贝，需要对集合进行修改的方法都直接抛出异常。
```java
Exception in thread "main" java.lang.UnsupportedOperationException
	at java.util.Collections$UnmodifiableMap.put(Collections.java:1459)
	at org.example.concurrent.UnsafeDemo.main(UnsafeDemo.java:18)
```







##### 绝对线程安全
不管运行环境如何，调用者都不需要任何额外的同步措施。

##### 相对线程安全
相对线程安全需要保证对这个对象的单独操作是线程安全的，在调用的时候不需要做额外的保障措施。但是对一些特定顺序的连续调用，就可能需要在调用端使用额外的同步手段来保证调用的正确性。
在 java 语言中，大部分的线程安全类都属于这种类型，如 Vector、HashTable、Collections 的 synchronizedCollection（） 方法包装的集合等。
```java
public class VectorUnsafeExample {
    private static Vector<Integer> vector = new Vector<>();

    public static void main(String[] args) {
        while (true) {
            for (int i = 0; i < 100; i++) {
                vector.add(i);
            }
            ExecutorService executorService = Executors.newCachedThreadPool();
            executorService.execute(() -> {
                for (int i = 0; i < vector.size(); i++) {
                    vector.remove(i);
                }
            });
            executorService.execute(() -> {
                for (int i = 0; i < vector.size(); i++) {
                    vector.get(i);
                }
            });
            executorService.shutdown();
        }
    }
}
```
```java
Exception in thread "Thread-159738" java.lang.ArrayIndexOutOfBoundsException: Array index out of range: 3
    at java.util.Vector.remove(Vector.java:831)
    at VectorUnsafeExample.lambda$main$0(VectorUnsafeExample.java:14)
    at VectorUnsafeExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```
如果需要保证上面的代码能够正确的执行下去，需要对删除元素和获取元素的代码进行同步处理
```java
executorService.execute(() -> {
    synchronized (vector) {
        for (int i = 0; i < vector.size(); i++) {
            vector.remove(i);
        }
    }
});
executorService.execute(() -> {
    synchronized (vector) {
        for (int i = 0; i < vector.size(); i++) {
            vector.get(i);
        }
    }
});
```
##### 线程兼容
线程兼容是指对象本身并不是线程安全的，但是可以通过在调用端正确地使用同步手段来保证对象在并发环境中可以安全的使用，我们平常说一个类不是线程安全的，绝大多数时候指的是这一种情况。Java API 中大部分的类都是属于线程兼容的，如与前面的 Vector 和 HashTable 相对应的集合类 ArrayList 和 HashMap 等。

##### 线程对立
线程对比是指无论调用端是否采取了同步措施，都无法再多线程环境中并发使用的代码。由于java语言天生就具备多线程特性，线程对比这种排斥多线程的代码很少出现的，而且通常都是有害的，应该尽量避免
#### 线程安全的实现方法
##### 互斥同步
synchronized  和 ReentrantLock


##### 非阻塞同步
互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种不同也成为阻塞同步。
互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否会真的出现竞争，都要进行加锁操作（这里讨论的事概念模型，实际上虚拟机会优化掉很大一部分不必要的枷锁操作）、用户态核心态转换、维护锁计数器和检查是否被阻塞的线程需要唤醒等操作。
###### cas
硬件指令记得发展，可以使用基于冲突检测的乐观并发策略；先进行操作，如果没有其他线程共享数据，那操作九成宫了，否则采取补偿措施（不断地重试，直到成功为止）。这种乐观的并发策略的很多实现都不需要将线程阻塞，因此这种同步操作成为非阻塞同步。
乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成，硬件支持的原子性操作最典型的操作是：比较并交换（Compare-and-Swap cas）CAS指令需要3个操作数，分别是内存地址V、旧的期待值A和新值B。当执行操作是，只有V的值等于 A，才会讲V的值更新为B。
###### AtomicInterger
J.U.C 包里面的整数原子类 AtomicInteger，其中的 compareAndSet() 和 getAndIncrement() 等方法都使用了 Unsafe 类的 CAS 操作。
以下代码使用了 AtomicInteger 执行了自增的操作。
```java
 private AtomicInteger atomicInteger = new AtomicInteger();


    public void add() {
        // 递增返回增加之后的值
        atomicInteger.incrementAndGet();
    }
```
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689044345854-2ac3c090-cbd6-4a05-b8d0-ac46db2907e7.png#averageHue=%232e2d2c&clientId=u107408bc-0f12-4&from=paste&height=151&id=u01c09457&originHeight=302&originWidth=1344&originalType=binary&ratio=2&rotation=0&showTitle=false&size=52397&status=done&style=none&taskId=u231acf2f-d4d8-4e1d-b1f4-2f47c1eeb01&title=&width=672)

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1689045096225-caabd2ff-d3bc-4af7-b349-dcd58519b5e1.png#averageHue=%232d2d2c&clientId=u107408bc-0f12-4&from=paste&height=204&id=u084af67a&originHeight=408&originWidth=1504&originalType=binary&ratio=2&rotation=0&showTitle=false&size=61055&status=done&style=none&taskId=ucfd261ed-4c0d-4888-9965-e8a855431ff&title=&width=752)
o 对象内存地址 offset 字段相对的对象内存地址的偏移 delta 需要加的值。通过 getIntVolatile(o, offset) 得到旧的预期值，通过调用 compareAndSwapInt() 来进行 CAS 比较，如果该字段内存地址中的值等于 v，那么就更新内存地址为 o+offset 的变量为 offset+v。
可以看到 getAndAddInt() 在一个循环中进行，发生冲突的做法是不断的进行重试。
###### ABA
如果一个变量的初次读写的时候 A 值，它的值被改成了B，后来又被改回了A，那 CAS 操作就会误认为它从来没有被改变过。
J.U.C 包提供了一个带有标记的原子引用类 AtomicStampedReference 来解决这个问题，它通过控制变量值的版本来保证 CAS 的正确性。大部分情况下 ABA 问题不会影响程序并发的正确性，如果需要解决 ABA 问题，改用传统的互斥同步要比原子类更高效
##### 无同步方案
要保证线程安全，并不是一定就要进行同步，如果一个方法本身就不涉及共享数据，那它自然就无需任何同步措施去保证正确性。
###### 栈封闭
多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量时存储在虚拟机栈中，属于线程私有的。
```java
   public void add100() {
        int cnt = 0;
        for (int i = 0; i < 100; i++) {
            cnt++;
        }
        System.out.println(cnt);
    }

    public static void main(String[] args) {
        StackClosedExample example = new StackClosedExample();
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.execute(() -> example.add100());
        executorService.execute(() -> example.add100());
        executorService.shutdown();
    }
```


###### 线程本地存储（Thread Local Storage）
如果一段代码中所需要的数据必须和其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行，如果能保证，就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。
符合这种特点的应用并不少见，大部分使用消费队列的架构模式（如"生产者-消费者"模式）都会将产品的消费过程尽量在一个线程中消费完。其中最重要的一个应用实例就是经典 Web 交互模型中的"一个请求对应一个服务器线程"， "Thread-per-Request"的处理方式，这种处理方式的广泛应用可以使得很多 Web 服务端应用都可以使用线程本地存储来解决线程安全的问题。cookie 预处理，处理成 userId 存储在 ThreadLocal 里边供业务方使用
可以使用 java.lang.ThreadLocal 类来实现线程本地存储功能
```java
public static void main(String[] args) {
        ThreadLocal threadLocal = new ThreadLocal();
        
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(threadLocal.get());
            threadLocal.remove();
        });
        
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            threadLocal.remove();
        });
        
        thread1.start();
        thread2.start();
    }
```
每一个 ThreadLocal 都维护一个 ThreadLocalMap 对象，具体存储键值对。
ThreadLocal 从理论上来说并不是解决多线程并发问题的，因为根本不存在多线程竞争。
在一些场景 (尤其是使用线程池) 下，由于 ThreadLocal.ThreadLocalMap 的底层数据结构导致 ThreadLocal 有内存泄漏的情况，应该尽可能在每次使用 ThreadLocal 后手动调用 remove()，以避免出现 ThreadLocal 经典的内存泄漏甚至是造成自身业务混乱的风险。

###### 可重复代码（Reentrant code）
这种代码也叫做纯代码（Prue code），可以在代码执行的任何时刻中断它，转而去执行另外一段代码（包括递归调用它本身），而在控制权返回之后，原来的程序不会出现任何错误。
可重入代码有一些共同的特征，例如不依赖存储在堆上的数据和公用的系统资源，用到的状态量都由参数中传入、不调用非可重入的方法等。
