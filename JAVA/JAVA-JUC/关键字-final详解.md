#### QA&S
- 所有的final修饰的字段都是编译期常量吗?
- 如何理解private所修饰的方法是隐式的final?
- 说说final类型的类如何拓展? 比如String是final类型，我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?
- final方法可以被重载吗? 可以
- 父类的final方法能不能够被子类重写? 不可以
- 说说final域重排序规则?
- 说说final的原理?
- 使用 final 的限制条件和局限性?
- 看本文最后的一个思考题

#### final 的基础使用
##### 修饰类
某个类的整体定义为 final 时，表明不能打算继承该类，而且也不允许别人继承，这个类不能有子类
final 类中的所有方法都隐式为 final ，所以在 final 类中给任何方法添加final 关键字无意义。
设计模式中最重要的两种关系，一种是继承/实现；另外一种是组合关系。所以当遇到不能用继承的（final修饰的类），应该考虑使用组成，一下是组合的实现
```java
class MyString{
   
    private String innerString;

    // ...init & other methods

    // 支持老的方法
    public int length(){
        return innerString.length(); // 通过innerString调用老的方法
    }

    // 添加新方法
    public String toMyString(){
        //...
    }
}
```

##### 修饰方法

- private 方法是隐式的final

类中所有的 private 方法都隐式地指定为 final ，由于无法取用 private 方法，所以也不能覆盖它。可以对 private 方法增加 final 关键字，
```java
public class Base {
    private void test() {
    }
}

public class Son extends Base{
    public void test() {
    }
    public static void main(String[] args) {
        Son son = new Son();
        Base father = son;
        //father.test();
    }
}
```
base 和 son 都有方法 test（），但是这并不是一种覆盖，因为 private 所修饰的方法都是隐式的final，也就是无法被继承，所以更不用说是覆盖了，在 Son 中的 test（）方法不过是属于 son 的新成员罢了，Son 进行向上转型得到 father ，但是 father.test（）是不可执行的，因为Base中的test（）方法是 private 的，无法被访问到

- final 方法是可以被重载的
```java
public class FinalExampleParent {
    public final void test() {
    }

    public final void test(String str) {
    }
}
```

##### 修饰参数
java 允许在参数列表中以声明的方式将参数指明为final，这意味着无法再方法中更改参数引用所指向的对象。主要是用来想匿名内部类传递数据。

##### 修饰变量
###### 所有的 final 修饰的字段都是编译期常量吗？
以为为编译器常量和非编译期常量
```java
public class Test{
    //编译期常量
    final int i =1;
    final static int J =1;
    final int[] a ={1,2,3,4};
    //非编译期常量
    Random r =new Random();
    final int k= r.nextInt();
    
}
```
k的值由随机数对象决定，所以不是所有的final修饰的字段都是编译期常量，只是k的值在被初始化后无法被变更。

###### static final
一个既是 static 又是 final 的字段只占据一段不能改变的存储空间，它必须在定义的时候进行赋值，否则编译期将不予通过。
```java
import java.util.Random;
public class Test {
    static Random r = new Random();
    final int k = r.nextInt(10);
    static final int k2 = r.nextInt(10); 
    public static void main(String[] args) {
        Test t1 = new Test();
        System.out.println("k="+t1.k+" k2="+t1.k2);
        Test t2 = new Test();
        System.out.println("k="+t2.k+" k2="+t2.k2);
    }
}
```
输出结果为  
```java
k=2 k2=7
k=8 k2=7
```
对于不同对象 k的值是不同的，但是 k2的值却是相同的，因为 static 关键字所修饰的字段并不属于一个对象，而是属于这个类的。简单的理解为 static final 所修饰的字段仅占据内存的一个一分空间，一旦被初始化之后就不会被更改了。
###### blank final
java 允许生成空白 final ，也就是说被声明为final 但又没有给出定值的字段，但是必须在该字段被使用之前赋值。

- 在定义进行赋值（这不叫空白 final）
- 在构造器中进行赋值，保证了该值在被使用前赋值。

增强了final 的灵活性。
```java
public class Test{
    final int i1=1;
    final int i2; //空白 fianl
    public Test(){
        i2=1;
    }
    public Test(int x){
        this.i2=x;
    }
}
```
i2的赋值更加灵活，注意，如果字段由 static 和 final 修饰，仅能在声明时候赋值或声明后再静态代码块中赋值，因为该字段不属于对象，属于这个类。
#### final 域重排序规则
上面聊得是 final 的使用，应该属于 java 基础层面的，final在多线程并发的情况？在java 内存模型中我们知道 java 内存模型为了能让处理器和编译器底层发挥他们最大的优势，对底层的约束就很少，也就是说针对底层来说java 内存模型就是一 弱内存模型。同时，处理器和编译器为了性能优化会对指令序列 由 编译器 和处理器进行重排序。那么，在多线程情况下，final 会进行怎样的重排序？会导致线程安全的问题吗？ 下边为final 的重排序。
##### final 域为基本类型
```java
public class FinalDemo {
    private int a; //普通域
    private final int b; //final 域

    private static FinalDemo finalDemo;


    public FinalDemo() {
        a = 1; // 1. 写普通域
        b = 2; // 2. 写final域
    }

    public static void writer() {
        finalDemo = new FinalDemo();
    }

    public static void reader() {
        FinalDemo demo = finalDemo; // 3.读对象引用
        int a = demo.a;    //4.读普通域
        int b = demo.b;    //5.读final域
    }

}
```
假设线程A在执行 writer（）方法，线程B执行 reader（） 方法
###### 写final 域重排序规则
写fianl 域的重排序规则禁止对 final 域的写重排序到 构造函数之外，这个规则的实现主要包含了两个方面：

- jvm 禁止编译期把 final 的写重排序到构造函数之外；
- 编译期会在 final 域写之后，构造函数 return 之前，插入一个 storestore 屏障，这个屏障可以禁止处理器把 fianl 域的写重排序到构造函数之外。

分析 writer 方法，虽然只有一行代码，但实际上做了两件事：

- 构造了一个FinalDemo 对象
- 把这个对象赋值给成员变量 finalDemo

![java-thread-x-key-final-1.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1690250801069-845b3561-16ab-446e-aa6f-3795a654dd4e.png#averageHue=%23f4f4f4&clientId=uf99d5579-aa60-4&from=paste&height=381&id=u67187e34&originHeight=762&originWidth=677&originalType=binary&ratio=2&rotation=0&showTitle=false&size=17157&status=done&style=none&taskId=u84d4b9d7-4c3f-46ed-948d-d30a96a4037&title=&width=338.5)
由于a，b之间没有数据依赖性，普通域（普通变量）a可能会被重排序到构造函数之外，线程B就有可能读的是普通变量a初始化之前的值（零值），这样就可能出现错误。而final域变量b，根据重排序规则，会禁止final修饰的变量b重排序到构造函数之外，从而b能够正确赋值，线程B就能够读到fianl变量初始化后的值。
因此，写final域的重排序规则可以确保，在对象引用为任意线程可见之前，对象的final 域已经被正确初始化过了，而普通域就不具有这个保障了。比如在上例，线程B可能就是一个未正确初始化的对象 finalDemo。

###### 读 final 域重排序规则
读 fianl 域重排序规则为：在一个线程中，初次读对象引用和初次读该对象包含的fianl域，JMM 会禁止这两个操作重排序（这个规则仅仅是针对处理器），处理器会在 读 final 域操作的前面插入一个 LoadLoad 屏障。实际上，读对象的引用和读该对象的final域存在间接依赖性，一般处理器不会重排序这两个操作。但是有一些处理器会重排序，因此，这条禁止重排序规则就是针对这些处理器而设定的。
read（）方法主要包含三个操作：

- 初次读引用变量 finalDemo；
- 初次读引用变量 finalDemo的普通域 a；
- 初次读引用变量 finalDemo 的final 域 b；

假设 线程A 写过程没有重排序，那么线程A和线程B有一种的可能执行时序为下图：
![java-thread-x-key-final-2.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1690251615562-97805efa-49d5-4290-b781-bcffdb421def.png#averageHue=%23f3f3f3&clientId=uf99d5579-aa60-4&from=paste&height=380&id=u387c19f4&originHeight=760&originWidth=629&originalType=binary&ratio=2&rotation=0&showTitle=false&size=17537&status=done&style=none&taskId=u3a460b1d-c7ed-412c-99bd-879e1a3cd5d&title=&width=314.5)
读对象的普通域被重排序到了读对象引用的前面就会出现线程B还未读到对象引用，就在读该对象的普通域变量，这显然是错误的操作。而final 域的读操作就“限定”了再读 final 域变量前已经读到了该对象的引用，从而就可以避免这种情况。
读final 域的重排序规则可以确保：在读一个对象的final 域之前，一定会先读这个包含这个final域的对象的引用。
##### final 域为 引用类型
###### 对final修饰的对象的成员域写操作
针对引用数据类型，final 域写针对编译器和处理器重排序增加了这样的约束：在构造函数内对一个final 修饰的对象的成员域的写入，与随后再构造函数之外把这个被构造的对象的引用赋给一个引用变量，这两个操作是不能被重排序的。这里的是“增加”也就是前面对 final 基本数据类型的重排序规则在这里还是使用。
```java
public class FinalReferenceDemo {
    final  int[] arrays;
    private  FinalReferenceDemo finalReferenceDemo;
    public FinalReferenceDemo(){
        arrays=new int[1];  // 1
        arrays[0]=1;   // 2
    }
    public void writerOne() {
        finalReferenceDemo = new FinalReferenceDemo(); //3
    }

    public void writerTwo() {
        arrays[0] = 2;  //4
    }

    public void reader() {
        if (finalReferenceDemo != null) {  //5
            int temp = finalReferenceDemo.arrays[0];  //6
        }
    }

}
```
针对上面的实例程序，线程A执行wirterOne方法，执行完线程B执行 writerTwo 方法，然后线程c 执行 reader 方法。
![java-thread-x-key-final-3.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1690252251648-c0652638-7f7d-4685-be0a-4a67fdd331da.png#averageHue=%23f4f4f4&clientId=uf99d5579-aa60-4&from=paste&height=377&id=u4620fa8f&originHeight=754&originWidth=779&originalType=binary&ratio=2&rotation=0&showTitle=false&size=18866&status=done&style=none&taskId=udf4cc597-b163-424f-82c2-187da3e9c42&title=&width=389.5)
由于对 final 域的写禁止 重排序到构造方法外，因此 1和3 不能被重排序。由于一个 final 域的引用对象的成员域写入不能与随后将这个被构造出来的对象赋给引用变量，重排序，因此 2和3 不能重排序。
###### 对 final 修饰的对象的成员域 读操作
JMM 可以确保线程C 至少能看到写线程 A 对 final 引用的对象的成员域的写入，即能看到 arrays[0]=1,而写线程B对数组元素的写入可能看不到，JMM 不保证线程B的写入对 线程C 可见，线程B和线程C之间存在数据竞争，此时的结果是不可预知的。如果可见的，可使用锁或者 volatile 关键字。
##### 关于final 重排序的总结
按照 final 修饰的数据类型分类：

- 基本数据类型：
   - final域写：禁止 final 域写与构造方法重排序，即禁止 final 域写重排序到构造方法之外，从而保证该对象对所有线程可见时，该对象的fianl域全部被初始化过。
   - final 域读：禁止初次读对象的引用与读该对象包含的fianl域的重排序。
- 引用数据类型：
   - 额外增加约束：禁止在构造函数对一个 final 修饰的对象的成员域的写入与随后将这个被构造的对象的引用赋值给 引用变量 重排序
#### final 深入理解
##### final 的实现原理
写final 域会要求编译器在final 域写之后，构造函数返回前插入一个 StoreStore 屏障。读 final 域的重排序规则会要求编译器在读 final 域的操作前插入一个 LoadLoad屏障。

##### 为什么 final 引用 不能从 构造函数中 “溢出”
比较有意思的问题：上面对 fianl 域写重排序规则可以确保在使用一个对象引用的时候该对象的final 域已经在构造函数中被初始化过了，但是这里其实有一个前提条件的，也就是：在构造函数，不能让这个被构造的对象被其他线程可见，可就是说该对象引用不能再构造函数中"溢出"。
```java
public class FinalReferenceEscapeDemo {
    private final int a;
    private FinalReferenceEscapeDemo referenceEscapeDemo;

    public FinalReferenceEscapeDemo() {
        a = 1; //1
        referenceEscapeDemo = this; //2
    }

    public void write() {
        new FinalReferenceEscapeDemo(); //3
    }

    public void reader() {
        if (referenceEscapeDemo != null) {  //3
            int temp = referenceEscapeDemo.a; //4
        }
    }
}
```
可能的执行时序
![java-thread-x-key-final-4.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1690253349847-ecc0177f-70e0-456d-8ba9-1c044ae0094f.png#averageHue=%23f4f4f4&clientId=uf99d5579-aa60-4&from=paste&height=313&id=u0949504e&originHeight=625&originWidth=676&originalType=binary&ratio=2&rotation=0&showTitle=false&size=12200&status=done&style=none&taskId=u55b276b6-0e13-4b94-951e-221176ff26d&title=&width=338)
假设一个线程A执行writer 方法 另一个线程执行 reader方法，因此构造函数中操作 1和 2 之间没有数据依赖性，1 和2 可以重排序，先执行了2，这个时候引用对象 referenceEscapeDemo 是一个完全没有初始化的对象，而当线程B 去读取 该对象时就会出错，尽管依然满足了 final 域 写重排序规则：在引用对象对所有线程可见时，其final 域已经被完全初始化成功，但是，引用对象 "this"逸出，该代码依然存在线程安全的问题。
##### 使用 final 的限制条件和 局限性
当声明一个fianl 成员时，必须在构造函数退出前设置它的值。
```java
public class MyClass{
    privater fianl int MyField=1;
    public MyClass() {
    ...
  }
}
```
或者
```java
public class MyClass {
  private final int myField;
  public MyClass() {
    ...
    myField = 1;
    ...
  }
}
```
将指向的成员变量 声明为 final 只能将该引用设为不可变的，而非执行的对象。
##### 好玩的一个问题
在Java中，对于byte类型的变量，其取值范围是从-128到127（8位有符号二进制补码表示）。当两个byte类型的变量进行加法运算时，Java虚拟机会将它们自动提升为int类型进行运算。
```java
byte b1=1;
byte b2=3;
byte b3=b1+b2;//当程序执行到这一行的时候会出错，因为b1、b2可以自动转换成int类型的变量，运算时java虚拟机对它进行了转换，结果导致把一个int赋值给byte-----出错

```
如果对 b1 b2 加上 final 就不会出错
```java
final byte b1=1;
final byte b2=3;
byte b3=b1+b2;//不会出错，
```
原因是 final 修饰的常量在编译期间会进行常量替换，而不会导致类型转换问题。但如果涉及到非常量的变量，仍然需要注意byte类型的取值范围和类型转换问题。
