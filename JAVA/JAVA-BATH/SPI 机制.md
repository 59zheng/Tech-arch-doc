> java 提供的一套用来被三方实现或者扩展的接口，它可以启用框架扩展和替换组件。SPI 的作用就是为了这些被扩展的API寻找服务实现 . 
解耦接口和实现的一种手段  ： 解耦接口和实现，接口开发规范，策略（动态选择）

### jdk spi 机制实现

1. 服务提供者提供了接口的一种具体实现后，在jar包的META-INF/services目录下创建一个以“接口全限定名”为命名的文件，内容为实现类的全限定名；
2. 接口实现类所在的jar包放在主程序的classpath中；
3. 主程序通过java.util.ServiceLoder动态装载实现模块，它通过扫描META-INF/services目录下的配置文件找到实现类的全限定名，把类加载到JVM；
   1. 从ServiceLoader.load(SpiDemo.class);一路进去会看到有个懒加载迭代器lookupIterator = new LazyIterator(service, loader); 用于清除当前jvm 中的默认实现，等待真正使用的时候调用 hasNext（）再查找服务，调用next（）才实例化服务类。
   2. 服务提供商安装约定,将具体的实现类名称放到/META-INF/services/xxx下, ServiceLoader就可以根据服务提供者的意愿, 加载不同的实现了, 避免硬编码写死逻辑, 从而达到解耦的目的.

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1672734288651-adf8c29f-7927-4154-aa2b-c1c6f161f1c1.png#averageHue=%23ededed&clientId=u3119c1ed-1a51-4&from=paste&height=111&id=u2ac8830c&originHeight=221&originWidth=969&originalType=binary&ratio=1&rotation=0&showTitle=false&size=50341&status=done&style=none&taskId=ud956196b-5a69-4341-9c07-a315aae4c8b&title=&width=484.5)

4. SPI的实现类必须携带一个不带参数的构造方法；

#### jdk spi 在jdbc 中的运用

1. jdk默认提供了了一个 java.sql.Driver的接口，具体的视线由不同的数据库厂商提供，以Mysql提供的JDBC实现包为例。在mysql-connector-java-*.jar 包中的 META-INF/services 目录下，有一个 java.sql.Driver 文件中只有一行内容，如下所示：

```sql
com.mysql.cj.jdbc.Driver
```

2. 在使用 mysql-connector-java-*.jar 包连接 MySQL 数据库的时候，我们会用到如下语句创建数据库连接：

```sql
javaString url = "jdbc:xxx://xxx:xxx/xxx"; 
Connection conn = DriverManager.getConnection(url, username, pwd); 
```

3. DriverManager 是 JDK 提供的数据库驱动管理器，在调用 getConnection() 方法的时候，DriverManager 类会被 Java 虚拟机加载、解析并触发 static 代码块的执行；在 loadInitialDrivers() 方法中通过 JDK SPI 扫描 Classpath 下 java.sql.Driver 接口实现类并实例化，核心实现如下所示：

```sql
private static void loadInitialDrivers() { 
    String drivers = System.getProperty("jdbc.drivers") 
    // 使用 JDK SPI机制加载所有 java.sql.Driver实现类 
    ServiceLoader<Driver> loadedDrivers =  
           ServiceLoader.load(Driver.class); 
    Iterator<Driver> driversIterator = loadedDrivers.iterator(); 
    while(driversIterator.hasNext()) { 
        driversIterator.next(); 
    } 
    String[] driversList = drivers.split(":"); 
    for (String aDriver : driversList) { // 初始化Driver实现类 
        Class.forName(aDriver, true, 
            ClassLoader.getSystemClassLoader()); 
    } 
} 
```

4. 在 MySQL 提供的 com.mysql.cj.jdbc.Driver 实现类中，同样有一段 static 静态代码块，这段代码会创建一个 com.mysql.cj.jdbc.Driver 对象并注册到 DriverManager.registeredDrivers 集合中（CopyOnWriteArrayList 类型），如下所示：

```sql
static { 
   java.sql.DriverManager.registerDriver(new Driver()); 
} 
```

5. 在 getConnection() 方法中，DriverManager 从该 registeredDrivers 集合中获取对应的 Driver 对象创建 Connection.

#### 不足之处

1. 遍历拿到所有的实现，不能按需拿具体的某个实现

### springBoot spi 机制实现

### dubbo spi 机制实现 

#### 与jdk优势

1. 解决jdk spi 按需加载的问题
2. 使用 warpeer 对原生扩展点实现进行增强。 
3. 在指定 dubbo 扩展点实现中引入 其他的扩展点 或者 ioc bean 

#### 微内核架构

 多流程，多个节点都提供接口，可以有多套实现，可以自由组合。插件的发现和组装机制，主要负责引导多个插件的运行，只进行流程的规定，不提供具体节点实现。

#### 配置规则

**与jdk不同之处**

1. 位置 /META-INF/dubbo 或者 /META-INF/dubbo/internal （dubbo 给自身留的一般不使用） 或者 /META-INF/service 也可
2. 配置文件格式 k=v 
3. 或者方式  ExtensionLoader.getExtensionLoader(interface.class) .getExtension("kname"); 可以获取指定接口的指定实现。
4. 标明 dubbo 扩展点 com.dubbo.common.extension.@SPI  注解标识 

#### 高级特性

- 方法增强
  1. Wrapper 对原生扩展点实现进行增强。（目前增强是无序的）需要提供一个接口的构造函数，将需要增加的扩展点实现进行引入，通过装饰着模式进行目标方法增强。
  2. 增强有序方法 ： 提供新的注解，修改目标 wrapper 的加载顺序。实现有序增强
- 依赖注入

需要提供目标接口的 setter（）方法，

      1. 扩展点注入
      2. spring ioc bean注入

- 自适应加载：基于指定实现类名称运行时动态选择目标类进行加载
  1. 自定义 adaptive 实现类 （使用 @Adaptive 注解标注）
     1. 采集用的的配置---config
     2. 根据配置中的信息，采用spi或者指定的扩展点实现
     3. 调用指定扩展点实现的相关方法
  2. dubbo自动生成adaptive实现类 （使用 @Adaptive 注解标注需要实现的接口）	
- 扩展点自动激活 @Activate（group="",value="'）
- @SPI（"kName') 默认扩展点

#### 源码

#### 

