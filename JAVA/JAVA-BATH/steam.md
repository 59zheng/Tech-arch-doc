常用词句说明

	语法糖： Syntactic sugar  这种语法对语言的功能并没有影响，但是更方便使用，通常来说使用语法糖可以增加程序的可读性。

## stream简介

```java
 A sequence of elements supporting sequential and parallel aggregate
```

译为：支持顺序和并行聚合的元素序列

Stream并不是I/O Stream，实际上，流不一定都是IO流？在 java 8 中得益于Lamdba带来的函数式编程，引入了一个全新的Stream概念，用于解决已有集合类库的弊端。

Stream可以理解为一个高级的Iterator（迭代器）。原始的Iterator，用户只能一个一个的遍历元素就行操作，高级版本的Stream，用户只需要指定需要对集合中的元素进行的操作，Stream会自动进行操作。

### Stream流的简单使用。

#### 1，创建stream流

##### 一.使用数组

```java
String arr = "1\n" + "115,64\n" + "118,67\n" + "119,68\n" + "120,69\n" + "121,70\n" + "122,71\n" + "2\n" + "3\n" + "4\n" + "5\n" + "6\n" + "64\n" +        "64,115\n" + "65\n" + "65,116\n" + "66\n" + "66,117\n" + "67\n" + "67,118\n" + "68\n" + "68,119\n" + "69\n" + "69,120\n" + "70\n" + "70,121\n" + "71\n" + "71,122\n" + "72";
String[] split = arr.replace("\n", ",").split(",");java.util.stream.Stream<String> split1 = java.util.stream.Stream.of(split);
```

#### 二。使用Collections

```java
List<String> strings = Arrays.asList(split);
Stream<String> stream = strings.stream();
```

#### 三，使用 Stream.generate() ；

```java
//        返回一个无穷序列无序流，其中每个元素是由提供Supplier生成即参数中的方法生成。这是适用于产生恒定的流，随机元素的流等。        
Stream<Integer> stream3 =Stream.generate(new Random()::nextInt).limit(10);        System.out.println(stream3.collect(Collectors.toList()));
```

#### 四，使用 Stream.iterate() ；

```java
/* 返回由迭代产生的无限顺序的参数* 参数 流元素类型 +起始元素*  参数   产生元素的方法  将起始元素传参进方法* */   
System.out.println(Stream.iterate(0,n->n+1).collect(Collectors.toList()));
```

#### 五， 使用流行的API，如Pattern.compile().splitAsStream() ；

```java
   String sentence = "ni hao wo hao da jia hao";
        Stream<String> wordStream = Pattern.compile("\\W").splitAsStream(sentence);
        String[] wordArr = wordStream.toArray(String[]::new);
        System.out.println(Arrays.toString(wordArr));
```

### 2，Stream常用方法

#### max和min

```java
        /*
         * Optional<T> max(Comparator<? super T> comparator);
         * 传入参数是一个 比较器 comparator
         *返回值是comparator中的T类型
         * */
//        比较器
        Comparator<String> comparator = Comparator.comparing(Integer::valueOf);
//      最大值
        System.out.println( strings.stream().max(comparator).get());
```

Collector 是一个可变的汇聚操作，它将输入元素累计到一个可变的结果容器中。在所有的元素处理完成后，Collector将累计的结果转换为一个最终的表示（是一个可选的操作）。collector支持串行和并行两种执行方式
链接地址：[https://blog.csdn.net/xiliunian/article/details/88773718](https://blog.csdn.net/xiliunian/article/details/88773718)

#### distinct（） 去重
  list 为 new arrayList（）的时候不是报错，为null 的时候会

```java
        /*
         * Stream<T> distinct();
         * 无参
         *返回值是comparator中的T类型
         * */
//        去重
        StringBuilder stringBuilder = new StringBuilder();
        Arrays.stream(arr.replace("\n", ",").split(",")).distinct().forEach(ite -> {
            stringBuilder.append(ite + ",");
        });
        System.out.println(stringBuilder.toString());
```

#### filter()    过滤

```java
 /**
       * Stream<T> filter(Predicate<? super T> predicate);
       * @Author: Zheng 
       * @Description:   过滤
       * @param predicate:
       * @return: Stream<T>
       **/
        System.out.println(Arrays.stream(arr.replace("\n", ",").split(",")).filter(a -> !a.equals("71")).collect(Collectors.toList()));
```

predicate是一个函数式接口：

提供一个抽象方法test，接受一个参数，根据这个参数进行判断，返回判断结果true ，false,同时提供几个默认的default方法，and ，or,negate组合判断

链接地址：[https://blog.csdn.net/love905661433/article/details/86425885](https://blog.csdn.net/love905661433/article/details/86425885)

#### collect（） 转换为集合

```java
/*  <R, A> R collect(Collector<? super T, A, R> collector); 
* @Description: collect() 
* @param Collector:收集器 
* @return: 收集器返回的类型 **/
List<String> collect = Arrays.stream(arr.replace("\n", ",").split(",")).collect(Collectors.toList());
```

#### map（）映射

```java
//      映射        
Stream.of("1", "2", "3", "4", "5").map(Integer::parseInt).forEach(ite-> System.out.println(ite));
```

#### limit（） 分页

```java
//        limit() 分页        
System.out.println(Stream.of("1", "2", "3", "4", "5").limit(3).collect(Collectors.toList()));
```

#### forEach（）  逐一处理

```java
//     forEach  逐一处理      
Stream.of("1", "2", "3", "4", "5").forEach(c-> System.out.println(c));
```

#### concat（） 组合

```java
//     concat组合流       
Stream.concat(Stream.of("1", "2", "3", "4", "5"),Stream.of("11", "10", "9", "8", "7")).forEach(c-> System.out.println(c));
```

#### skip （） 跳过指定元素

```java
//        skip  跳过几个元素      
System.out.println(Stream.of("1", "2", "3", "4", "5").skip(1).collect(Collectors.toList()));
```

测试类demo

```java
package com.ruoyi.fangyuanapi.utils;
import java.util.*;
import java.util.stream.Collectors;
import java.util.stream.Stream;
public class demo {    
    public static void main(String[] args) {        
        String arr = "1\n" + "115,64\n" + "118,67\n" + "119,68\n" + "120,69\n" + "121,70\n" + "122,71\n" + "2\n" + "3\n" + "4\n" + "5\n" + "6\n" + "64\n" +                "64,115\n" + "65\n" + "65,116\n" + "66\n" + "66,117\n" + "67\n" + "67,118\n" + "68\n" + "68,119\n" + "69\n" + "69,120\n" + "70\n" + "70,121\n" + "71\n" + "71,122\n" + "72";        
        String[] split = arr.replace("\n", ",").split(",");        java.util.stream.Stream<String> split1 = java.util.stream.Stream.of(split);        List<String> strings = Arrays.asList(split);
        ////        返回一个无穷序列无序流，其中每个元素是由提供Supplier生成即参数中的方法生成。这是适用于产生恒定的流，随机元素的流等。       
        Stream<Integer> stream3 = Stream.generate(new Random()::nextInt).limit(10);        System.out.println(stream3.collect(Collectors.toList()));       
        /* 返回由迭代产生的无限顺序的参数         
        * param 流元素类型 +起始元素         
        *  param   产生元素的方法  将起始元素传参进方法         * */        System.out.println(Stream.iterate(0, n -> n + 1).limit(10).collect(Collectors.toList()));       
        /*         * Optional<T> max(Comparator<? super T> comparator);         
        * 传入参数是一个 比较器 comparator         
        *返回值是comparator中的T类型         
        * */
        //        比较器        
        Comparator<String> comparator = Comparator.comparing(Integer::valueOf);
        //      最大值       
        System.out.println(strings.stream().max(comparator).get());        
        /*         * Stream<T> distinct();        
        * 无参         
        *返回值是comparator中的T类型        
        * */
        //        去重        
        StringBuilder stringBuilder = new StringBuilder();        Arrays.stream(arr.replace("\n", ",").split(",")).distinct().forEach(ite -> {            stringBuilder.append(ite + ",");        });        System.out.println(stringBuilder.toString());        
        /*         * Stream<T> filter(Predicate<? super T> predicate);        
        * @Author: Zheng        
        * @Description: 过滤         
        * @param predicate:        
        * @return: Stream<T>         **/        System.out.println(Arrays.stream(arr.replace("\n", ",").split(",")).filter(a -> !a.equals("71")).collect(Collectors.toList()));        
        /*          <R, A> R collect(Collector<? super T, A, R> collector);         * @Description: collect()         
        * @param Collector:收集器        
        * @return: 收集器返回的类型        
        **/        
        List<String> collect = Arrays.stream(arr.replace("\n", ",").split(",")).collect(Collectors.toList());
        //        分组        
        System.out.println(Arrays.stream(arr.replace("\n", ",").split(",")).collect(Collectors.groupingBy(Object::toString)));
        //      映射        
        Stream.of("1", "2", "3", "4", "5").map(Integer::parseInt).forEach(ite-> System.out.println(ite));
        //        limit() 分页       
        System.out.println(Stream.of("1", "2", "3", "4", "5").limit(3).collect(Collectors.toList()));
        //     forEach  逐一处理        
        Stream.of("1", "2", "3", "4", "5").forEach(c-> System.out.println(c));
        //     concat组合流       
        Stream.concat(Stream.of("1", "2", "3", "4", "5"),Stream.of("11", "10", "9", "8", "7")).forEach(c-> System.out.println(c));
        //        skip  跳过几个元素        
        System.out.println(Stream.of("1", "2", "3", "4", "5").skip(1).collect(Collectors.toList()));    }}
```
