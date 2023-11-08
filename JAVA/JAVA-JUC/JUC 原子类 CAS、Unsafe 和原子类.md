> JUC中的多数都是通过 volatile 和cas 实现的，CAS 本质上提供的是一种无锁方案， 而 Synchronized 和Lock 是互斥锁方案； java 原子类本质上使用的是 CAS ，而CAS底层是通过 Unsafe 类实现的，

#### QA&S

- 线程安全的实现方法有哪些?
- 什么是CAS?
- CAS使用示例，结合AtomicInteger给出示例?
- CAS会有哪些问题?
- 针对这这些问题，Java提供了哪几个解决的?
- AtomicInteger底层实现? CAS+volatile
- 请阐述你对Unsafe类的理解?
- 说说你对Java原子类的理解? 包含13个，4组分类，说说作用和使用场景。
- AtomicStampedReference是什么?
- AtomicStampedReference是怎么解决ABA的? 内部使用Pair来存储元素值及其版本号
- java中还有哪些类可以解决ABA的问题? AtomicMarkableReference

#### CAS
线程安全的实现方法包含

- 互斥同步： synchronzied 和 ReentrantLock 
- 非阻塞同步： CAS ， AtomciXXX
- 无同步方案，栈封闭，Thread Local ，可重入代码
##### 概念
CAS的全称为Compare-And-Swap，直译就是对比交换。是一条CPU的原子指令，其作用是让CPU先进行比较两个值是否相等，然后原子地更新某个位置的值，经过调查发现，其实现方式是基于硬件平台的汇编指令，就是说CAS是靠硬件实现的，JVM只是封装了汇编调用，那些AtomicInteger类便是使用了这些封装后的接口。 简单解释：CAS操作需要输入两个数值，一个旧值(期望操作前的值)和一个新值，在操作期间先比较下在旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。
CAS操作是原子性的，所以多线程并发使用CAS 更新数据时，可以不适用锁。JDK 中大量使用了 CAS 来更新数据防止加锁（Synchronized 重量级锁）来保持原子更新
##### CAS 问题
CAS 方式为乐观锁，synchronized 为悲观锁，因此使用CAS 解决并发问题性能更优，但会产生如下问题：
###### ABA问题
因为CAS 需要在操作值的时候，检查值有没有发生变化，比如没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS 进行检查时则会发现它们的值没有发生变化。
ABA 问题的解决方案就是使用版本号，在变量前面追加版本号，每次变量更新的时候把版本号+1，那么A-B-A 变成 1A-2B-3A
java 1.5 开始，JDK 的 Atomic 包中提供一个类 AtomicStampedReference 来解决 ABA 问题，这个类的 compareAndSet 方法的作用就是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。
###### 循环时间开销大
自旋CAS 如果长时间不成功，会给CPU 带来非常大的执行开销。如果JVM能支持处理器提供的 pause 指令，效率会有很大提升，pause 指令有两个作用：第一，它可以延迟流水线执行命令（de-pipeline），使cpu 不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理上延迟时间是零；第二，它可以避免在退出循环的时候因内存顺序冲突（Memory Order Violation ）而引起CPU 流水线被清空（CPU Pipeline Flush）从而提高CPU 的执行效率
###### 只能保证一个共享变量的原子操作
对同一个共享变量执行操作时，可以使用循环CAS 的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。
还有一个取巧的方法，就是把多个共享变量合并成一个共享变量来操作，比如，有两个共享变量i = 2，j = a，合并一下ij = 2a，然后用CAS来操作ij。
从Java 1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

##### 
#### UnSafe 类
sum.misc 包下的类，主要提供执行低级别，不安全操作的方法，如直接访问系统内存资源，自主管理内存资源等，这些方法在提高java 运行效率、增强java 语言底层资源操作能力方面起了很大的作用，但由于UnSafe 类使java 语言拥有了类似C语言指针一样操作内存空间的能力，增加程序发生相关指针的风险。
这个类尽管里面的方法都是 public 的，但是并没有办法使用它们，JDK API 文档也没有提供任何关于这个类的方法的解释。总而言之，对于 Unsafe 类的使用都是受限制的，只有授信的代码才能获得该类的实例，当然 JDK 库里面的类是可以随意使用的。
调用方式基于 反射调用
```java
public static final Unsafe unsafe = getUnsafe();

static sun.misc.Unsafe getUnsafe() {
    try {
        Field field = Unsafe.class.getDeclaredField("theUnsafe");
        field.setAccessible(true);
        return  (Unsafe) field.get(null);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```
![java-thread-x-atomicinteger-unsafe.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1699252980919-dde38832-fde2-4b1c-9df8-3e10afddedb4.png#averageHue=%23f8f8f7&clientId=u7d368697-5ba1-4&from=paste&height=516&id=u63456dee&originHeight=1032&originWidth=2306&originalType=binary&ratio=2&rotation=0&showTitle=false&size=66689&status=done&style=none&taskId=ua66e9d26-ca97-4073-865b-fcf20ff23fe&title=&width=1153)

#### AtomicInteger 
常见 api
```java
public final int get()：获取当前的值
public final int getAndSet(int newValue)：获取当前的值，并设置新的值
public final int getAndIncrement()：获取当前的值，并自增
public final int getAndDecrement()：获取当前的值，并自减
public final int getAndAdd(int delta)：获取当前的值，并加上预期的值
void lazySet(int newValue): 最终会设置成newValue,使用lazySet设置值后，可能导致其他线程在之后的一小段时间内还是可以读到旧的值。
```
