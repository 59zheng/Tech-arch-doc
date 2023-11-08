#### Spring 如何设计容器的，BeanFactory 和 ApplicationContext的关系是什么?
两个接口
##### 功能上区别：
 BeanFactory 是spring 中最底层的接口，是IOC的核心，其功能包含了各种Bean的定义，加载，实例化，依赖注入和生命周期的管理。是IOC最基本的功能。 
提供配置框架及基本功能，但是无法支持spring的aop功能的web应用。而ApplicationContext 接口作为BeanFactory 的派生，因而提供 BeanFactory 所有的功能。。而且ApplicationContext 还在功能上做了扩展，相较于BeanFactory， ApplicationContext 还提供了以下功能：
1）messageSource 提供国际化的消息访问
2）资源访问，如URL和文件
3）时间传播特性，即支持aop特性
4）载入多个（有继承关系）上下文，使得每一个上下文都专注于一个特定的层次，比如应用的web层。
ApplicationContext： 是IOC 容器的另一个重要接口，它继承了BeanFactory 的基本功能，同时也继承了容器的高级功能，如： MessageSource （国际化资源接口），ResourceLoader（资源加载接口），ApplicationEventPublisher（应用事件发布接口等）
##### 加载方式区别
BeanFactory 是延时架子啊，懒加载。getBean的时候才会对bean进行加载实例化
Application 是在容器启动的时候，一次性创建所有的bean （单例非懒加载）运行速度相比 BeanFactory 比较快
##### 注册方式区别
BeanFactory 和 ApplicationContext 都支持 BeanPostPorcessor，BeanFactoryPostPorcessor ,
beanFactory 需要手动注册的
ApplicationContext 是自动注册的

#### FactoryBean 和 BeanFactory 区别
 BenaFactory 是bean 的工厂，ApplicationContext 的父类，IOC容器的核心，负责生产和管理bean 对象。
FactoryBean： 是Bean ，可以通过实现 FactoryBean 接口定制实例化 Bean 的逻辑，通过代理一个 Bean 对象，对方法前后进行操作。自定义 bean的创建方式
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686794091599-687cf28b-e332-4ad9-9133-1251c9754527.png#averageHue=%23f7f6f6&clientId=ub8aa06df-5366-4&from=paste&height=396&id=u97fc17ad&originHeight=792&originWidth=2592&originalType=binary&ratio=2&rotation=0&showTitle=false&size=439930&status=done&style=none&taskId=u84d8fd44-9047-4d8b-b948-a4e0d7d617f&title=&width=1296)

#### 事务传播行为，spring 事务中有几种事务传播行为？
解决业务层方法之间相互调用的事务问题，多个事务方法相互调用时，事务如何在这些方法间传播。事务方法之间相互调用。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686794607344-f1c70c2d-6d73-49f7-bed4-feef3ace425b.png#averageHue=%23eeebe6&clientId=ub8aa06df-5366-4&from=paste&height=247&id=u6cbbbac0&originHeight=494&originWidth=2664&originalType=binary&ratio=2&rotation=0&showTitle=false&size=733837&status=done&style=none&taskId=u09beab07-84f2-447b-85ed-99777fcea1d&title=&width=1332)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686794723352-537c2211-b48e-466c-acab-a3cd223b8161.png#averageHue=%23f9f8f8&clientId=ub8aa06df-5366-4&from=paste&height=555&id=uf7f27608&originHeight=1110&originWidth=2674&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1267261&status=done&style=none&taskId=u6c7d413e-95ee-49a5-b318-25dc6f779f6&title=&width=1337)
#### spring的单例 Bean 是否有并发安全问题？
spring 中单例bean 不是线程安全的，实际上上大部分的bean 是无状态的，如果有状态的话需要考虑并发安全。修改bean的属性 把 singleton 改为 prototype 单例变多例
有实例变量的bean，可以保存数据，是非线程安全的，有状态
没有实例变量的bean，不能保存数据，是线程安全的。
 
#### 如何解决有状态bean的线程安全问题
修改bean的作用域 singleton 改变为prototype 单例变多例
使用ThreadLocal 变量，为每一个线程设置为变量副本。每个线程独有的。
#### spring 有哪些扩展点（IOC流程）
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686812147238-f293afa8-9f40-49f2-864a-355a60e7153a.png#averageHue=%23faf7f4&clientId=ub8aa06df-5366-4&from=paste&height=927&id=u5a2f3f93&originHeight=1854&originWidth=3314&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2457098&status=done&style=none&taskId=ue79fd044-7030-4db2-8576-635996a89b7&title=&width=1657)

#### spring框架中的bean的生命周期  
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686812293275-cc2353dd-527b-4d63-9cda-a0685135ac3a.png#averageHue=%23eef3ee&clientId=ub8aa06df-5366-4&from=paste&height=162&id=u0622add3&originHeight=324&originWidth=1980&originalType=binary&ratio=2&rotation=0&showTitle=false&size=906243&status=done&style=none&taskId=u342b866b-3abc-4cda-9fec-dbf5d835ebc&title=&width=990)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686812449620-787c6b17-e4a4-4b70-a415-cbb0e163f058.png#averageHue=%23eff1f0&clientId=ub8aa06df-5366-4&from=paste&height=425&id=u0c725a4b&originHeight=850&originWidth=2190&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2763208&status=done&style=none&taskId=u2fe02f8f-f4e3-4d68-81b2-65b3be378f4&title=&width=1095)
![截屏2023-06-15 15.00.59.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686812693504-70086eb4-84f4-48ee-b0e9-4533bb0ec0f5.png#averageHue=%23f3efe8&clientId=ub8aa06df-5366-4&from=paste&height=979&id=u5241f3a3&originHeight=1958&originWidth=2270&originalType=binary&ratio=2&rotation=0&showTitle=false&size=3876514&status=done&style=none&taskId=ua4e281ef-fc92-4aca-bef0-de973b40d62&title=&width=1135)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686812506137-05756127-584c-4fbe-bdae-2b29c502ff43.png#averageHue=%23c0d8c2&clientId=ub8aa06df-5366-4&from=paste&height=465&id=ucd93876b&originHeight=930&originWidth=2300&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2453354&status=done&style=none&taskId=u55917a0d-51df-4610-8f91-6107e8f5067&title=&width=1150)
InstantiationAwareBeanPostProcesor 是 BeanPostProcessor 的子接口，作用在bean 实例化之前和之后阶段。
作用： 通常是用来控制特别目标类的默认实例化。 例： 特殊的bean的创建（连接池，懒加载的初始化类）注入对象的字段注入。
特殊用途：短路后置处理器，跳过 doCreateBean（） 方法，不走默认的生命周期那套。
#### 循环依赖
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686881276985-62081a77-b092-4c9c-929f-31b3fa59fc16.png#averageHue=%23f7f5f5&clientId=u2720f9bf-3b89-4&from=paste&height=879&id=ub1d5f17b&originHeight=1758&originWidth=2732&originalType=binary&ratio=2&rotation=0&showTitle=false&size=877337&status=done&style=none&taskId=u7a40d10c-ec0a-40dd-8ace-44b97d58c7b&title=&width=1366)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686881201161-2e9c7b2f-57b5-4a1a-8bd9-c0922ad2fb7c.png#averageHue=%23f6f4ee&clientId=u2720f9bf-3b89-4&from=paste&height=605&id=u429b11e8&originHeight=1210&originWidth=2770&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1709897&status=done&style=none&taskId=uc118529c-b38a-4b14-b33d-a6a3bafcbe1&title=&width=1385)


#### spring查询缓存的时候，是按照什么顺序来获取对象的
一二三 顺序查
SingletonObjects  一级缓存，存放完整的Bean
earlySingletonObjects   二级缓存  存放 创建出来但未赋值及初始化的 Bean
singletonFactories   三级缓存  存放 ObjectFactory  （aop代理会有问题）
#### spring一开始提前暴露的不是 ObjectBean 而是 ObjectFactory 为啥？
设计到 aop ，如果创建的 Bean 是有代理的，那么注入的就应该是代理 Bean，而不是原始的 Bean。但 是 Spring 一开始并不知道 Bean 是否会有循环依赖，通常情况下(没有循环依赖的情况下)，Spring 都会在完成 填充属性，并且执行完初始化方法之后再为其创建代理。但是，如果出现了循环依赖的话，Spring 就不得不为其提 前创建代理对象，否则注入的就是一个原始对象，而不是代理对象。因此，这里就涉及到应该在哪里提前创建代理 对象?
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1686882503381-b1b4ac80-7a1a-404a-b5bd-427af879905c.png#averageHue=%23faf6f6&clientId=u2720f9bf-3b89-4&from=paste&height=736&id=u487e9a61&originHeight=1472&originWidth=1182&originalType=binary&ratio=2&rotation=0&showTitle=false&size=691860&status=done&style=none&taskId=u87c478a2-1108-499b-af6a-16345c636f3&title=&width=591)
#### 循环依赖（三级缓存）
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687140966210-f10bef70-f8c7-42b2-b07d-21b73745a6e8.png#averageHue=%23fbf8f6&clientId=ue1a0c30d-f5aa-4&from=paste&height=1078&id=u590fa60d&originHeight=2156&originWidth=4560&originalType=binary&ratio=2&rotation=0&showTitle=false&size=4557257&status=done&style=none&taskId=u330bf366-8a6b-4893-b0f2-9ac73ad0d75&title=&width=2280)
总结： 三级缓存最主要的作用：没有循环依赖的情况下，能够保证 Bean 的代理 Bean 实在初始化的最后阶段生成代理对象，遵循 Spring 设计原则！

#### spring事务在哪几种情况下会生效？
##### 数据库引擎不支持事务
InnoDB  mysql 5.5 之后默认支持事务    MyISAM 不支持。
##### 没有被spring 管理
##### 方法不是 public 
spring 事务是基于动态代理的方式来实现的。在bean 初始化的过程中进行后置方法的执行，不是public，spring 默认不会支持事务
##### 方法用final /static 修饰
spring 事务底层使用了aop，也就是通过jdk 动态代理或者cglib ，生成了代理类，在代理类中实现的事务功能。如果方法用final 修饰的话，在代理类中无法重写方法，从而实现事务功能。
##### 方法内部调用
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687141550634-336ff780-9a40-4411-b272-047205080489.png#averageHue=%23f8f7f7&clientId=ue1a0c30d-f5aa-4&from=paste&height=748&id=uc1675686&originHeight=1496&originWidth=2710&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1421188&status=done&style=none&taskId=u32894726-de30-4e12-9cfb-2c5ec96c0bb&title=&width=1355)
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687141573371-b6e50c56-be79-45a4-99ce-1777fa74eb9b.png#averageHue=%23f8f8f7&clientId=ue1a0c30d-f5aa-4&from=paste&height=609&id=u000d1e46&originHeight=1218&originWidth=2666&originalType=binary&ratio=2&rotation=0&showTitle=false&size=566494&status=done&style=none&taskId=u1893e9cf-0a2c-47d8-996d-4c6ac8287cf&title=&width=1333)
##### 异常被吃了
手动 tray catch  自己处理
##### 错误的传播属性


#### Bean 注入容器有哪些方式？

- @Configuration + @Bean 

      @Configuration 声明一个配置类，然后使用@Bean 注解，用于声明Bean

- @ ComponentScan 扫包范围，指定包路径
- @Import 注解导入

Spring 扩展的时候经常会用到，经常搭配自定义注解使用，然后在容器中导入一个配置文件。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687141846994-eb05c818-83c5-40c1-8687-b5798fa61445.png#averageHue=%23f6f5f5&clientId=ue1a0c30d-f5aa-4&from=paste&height=320&id=u550dc320&originHeight=640&originWidth=2246&originalType=binary&ratio=2&rotation=0&showTitle=false&size=572598&status=done&style=none&taskId=u406f2313-5a02-46e9-b0eb-f56133dbdfd&title=&width=1123)

- 实现BeanDefinitionRegistryPostProcessor 进行后置处理

spring 提供的后置扩展点，在执行的时候动态添加一个bean

- 使用FactoryBean
#### SpringMvc 的controller 是不是单例模式
是，单例模式，在多线程访问的时候有线程安全问题，不要用可变桩体阿亮，可以使用ThreadLocal ，为每个线程单独生成一个变量副本，独立操作，互不影响
#### Spring MVC 的拦截器和 Filter 过滤器 区别
filter 优先于 拦截器 执行，
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687142214976-8d2f4e38-5844-47db-bd49-08bbe1a2909d.png#averageHue=%23fafaf9&clientId=ue1a0c30d-f5aa-4&from=paste&height=727&id=u4366dbb4&originHeight=1454&originWidth=2836&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2211008&status=done&style=none&taskId=u8249e35f-99ea-45f5-a8b4-2f2a9ab9cc5&title=&width=1418)
spring mvc 和 spring 存在父子容器 关系， 
#### AOP 实现原理
通过动态代理实现的，为某个bean 配置了 切面，会在创建这个bean 的时候，创建一个代理，在这个代理类中进行方法增强。 jdk 动态代理 和 cglib 动态代理，有限 jdk 如果实现 接口的话
##### spring Aop 中有哪些 Advice ？
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687142952284-b5ed4fa7-4388-4733-a8b8-b6a6b06d64b8.png#averageHue=%23fbfafa&clientId=ue1a0c30d-f5aa-4&from=paste&height=408&id=u27e22f04&originHeight=816&originWidth=2666&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1119050&status=done&style=none&taskId=u2caeb902-3400-4d1b-99bf-b02b3d0e4e9&title=&width=1333)
#### spring @ EnableAspectJAutoProxy 的原理
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687153342395-8feeac71-0973-4b05-88ac-e577b38fc3e7.png#averageHue=%23e3e0d8&clientId=ue1a0c30d-f5aa-4&from=paste&height=280&id=ud0f8fea0&originHeight=560&originWidth=2644&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1230815&status=done&style=none&taskId=uaef01e2c-f784-4916-8642-2610e6b1738&title=&width=1322)
##### spring aop 自动代理的实现

#### aop  中jdk 动态代理 和 cglib 动态代理区别
cglib 基于继承实现，jdk 动态代理 基于接口实现。优先使用jdk 动态代理实现 aop 。
区别： jdk 动态代理使用 jdk 中的类 proxy 来创建代理对象，它使用反射技术来实现，不需要导入其他依赖。cglib 需要引入相关依赖 asm.jar ，使用字节码增强技术来实现。
当目标类实现了接口的时候 spring Aop 默认使用 jdk 动态代理方式来增强方法，没有实现接口的时候使用 cglib 动态搭理方式增强方法。

#### Mybatis 有那些执行器，之前的区别是什么
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687253426807-3617a5d9-b964-44e1-8277-fbc0d24eb840.png#averageHue=%23fbfbfb&clientId=ufed652dc-062f-4&from=paste&height=686&id=uc95fe6d7&originHeight=1372&originWidth=2336&originalType=binary&ratio=2&rotation=0&showTitle=false&size=790806&status=done&style=none&taskId=u2ec8cf0b-e996-4d50-8954-452c871021d&title=&width=1168)
cachingExecutor  用于处理二级缓存，如果缓存中不存在要查询的数据，那么将查询请求委托给其他的Executor。如果是执行SQL的增伤，
#### mybaits 插件机制
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687748975293-85c99c3f-e22b-4544-99db-57fe2f49f1ee.png#averageHue=%23f1ece8&clientId=uf715a2f3-bf83-4&from=paste&height=931&id=uf96d11fe&originHeight=1862&originWidth=2966&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2024657&status=done&style=none&taskId=u820c8268-596c-47b1-a484-21bda82b3cb&title=&width=1483)

#### mybaits cachekey 如何保证唯一的
cachekey 是每次查询的特性抽象的类，map集合 cachekey（唯一标识）：value （查询结果） 
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687749401787-03e938cc-d0af-4987-864d-f795466226b9.png#averageHue=%23fbfbfa&clientId=uf715a2f3-bf83-4&from=paste&height=274&id=u17149dd7&originHeight=548&originWidth=2120&originalType=binary&ratio=2&rotation=0&showTitle=false&size=559586&status=done&style=none&taskId=u17c5e686-6872-4143-98bd-e4426ef9f5b&title=&width=1060)
比较元素是否相同，不是直接比对的地址值
#### mybaits 缓存机制
一级缓存是sqlsession 级别的缓存，默认支持，开启的， 与sqlsession 的生命周期相同
二级缓存是 mapper 级别的缓存，可以在各个sqlsession 之间共享，默认不开启，需要手动开启  ，会造成脏读，缓存命中特别低
二级缓存-》 一级缓存-》 数据库

#### springboot 自动配置›
约定大约配置  autoconfigure 中存在spring.factories 文件，这个文件中定义了可以剩下的所有配置类。 在项目启动的时候，会加载所有 spring.factories 文件，基于enableconfigure 的key 获取到所有的配置类。
首先会进行 去重，以及手动排除 的； 
fifter 过滤  在目标的配置类上边 基于 conditional 开头的注解，是否生效的条件
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687750567084-f0ae013d-6699-44ff-8a6d-95cbdefd921b.png#averageHue=%23f8f7f6&clientId=uf715a2f3-bf83-4&from=paste&height=247&id=uebd0fbdf&originHeight=494&originWidth=2232&originalType=binary&ratio=2&rotation=0&showTitle=false&size=574148&status=done&style=none&taskId=ue8e5d3b9-3a3d-4d9b-b902-45b40d274a0&title=&width=1116)

#### springboot starter 自定义实现
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687750695285-8ce25053-dede-4a1d-b243-bd22b1412e44.png#averageHue=%23ebebeb&clientId=uf715a2f3-bf83-4&from=paste&height=183&id=uc4e440ea&originHeight=366&originWidth=1710&originalType=binary&ratio=2&rotation=0&showTitle=false&size=116063&status=done&style=none&taskId=u88df2c10-56e7-478e-a90d-75fb0b93043&title=&width=855)
#### HashMap 面试相关
##### jdk1.7 和1.8区别
1.7 数据加链表  1.8 数组加链表加 红黑树。链表是为了解决哈希冲突而存在的。红黑树是为了链表过长的情况下，加快检索速率。  1.7头插  1.8 尾差
##### 链表转红黑树条件：

- 当链表⻓度大于等于 8 且数组⻓度大于等于 64 才会转红黑树 
- 将链表转换成红黑树前会判断，如果当前数组的⻓度小于 64，那么会选择先进行数组扩容，而不是转换为红 黑树，以减少搜索时间。

数组扩容动作： 数据长度二倍，会重新计算原数组的数组位置，可能存在两种情况 =index 或者 = index +oldtable.size   数组扩容可以有效减少链表的长度
##### 红黑树退化链表条件：

- 为6的时候会退化成链表，长度。防止频繁 树转链表，链表转树。
##### 链表转红黑树的阈值为8：
 泊松分布
##### 默认加载因子为什么是 0.75 
 时间和空间成本上提供了很好的折中，较高的值会降低空间开销，但提高查找成本(体现在大多数的HashMap 类的操作，包括get 和put)设置初始大小时，应该考虑预计的entry 数在map 及其负载系数，并且尽量减少 rehash 操作的次数。如果初始容量大于最大条目数除以负载因子，rehash 操作将不会发生。
##### HashMap 中 key 的存储索引是怎么计算的？
根据key的值计算出 hashcode 的值，根据 hashcode 计算出 hash值，  hash扰动次数 （jdk1.7   7次扰动） （jdk1.8    右移16为，进行异或操作）
jdk1.7 多次扰动，计算hash 值性能稍差， 通过hashcode 的高16位异或 低16位，增大hash 的离散型。
##### 为什么hash 值与 length -1 相与？

- 把 hash 值对数组⻓度取模运算，模运算的消耗很大，没有位运算快。
- 当 length 总是 2 的n次方时，h& (length-1) 运算等价于对length取模，也就是 h%length，但是 & 比 % 具有 更高的效率。
##### HashMap 扩容方式
java里边的数组无法自动扩容，创建一个新的数组，将原数组的元素导过来。 key可以为 null 会放在第一个位置。


##### 
##### 
#### ConcurrentHashMap 相关
##### 1.7 如何保证线程安全？
采用数组加分段锁实现的。 锁的粒度太大。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687843683712-b7acc9dd-bb1e-4b70-b21f-89c7c19bff29.png#averageHue=%23fafafa&clientId=ube0c3e14-dffd-4&from=paste&height=935&id=u547206c1&originHeight=1870&originWidth=2714&originalType=binary&ratio=2&rotation=0&showTitle=false&size=2001463&status=done&style=none&taskId=u11f4f669-e8bf-4541-8980-a995b70cc2c&title=&width=1357)

1.7 put 操作
先计算 在segment 数组中的index ，然后计算在指定 entry[] 的index 值
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1687844108087-63e2ae60-3b61-446a-aff8-4fa81212633b.png#averageHue=%23e1ddd5&clientId=ube0c3e14-dffd-4&from=paste&height=145&id=u3d3a4137&originHeight=290&originWidth=2698&originalType=binary&ratio=2&rotation=0&showTitle=false&size=684062&status=done&style=none&taskId=ue4c12042-4b4c-4b7e-b21a-c52629a483e&title=&width=1349)
如何实现扩容，仅仅和 entry数组的长度有关，segment 数组的长度不会改变，扩容不会改变这个长度。
并发度是多少，能够同时更新 ConcurrentHashMap 的最大线程数，即 segment 的 size 。如果自定义，会选取大于等于该值的最大2 的幂等数

##### 1.8 如何保证线程安全？
弃用分段锁，使用 HashMap 的数据结构+ synchronized 锁。只需要锁住这个链表的头节点。减少锁的粒度。减少并发冲突的概率。去除；额 Segmnets 分段锁。
put 操作：
判断是否初始化，当前 node 节点是否存储元素，通过cas 尝试添加。如果正在扩容，参与一起扩容，否则锁node头。
synchroized 锁进行了优化  锁粗化，锁升级，cas 等。

