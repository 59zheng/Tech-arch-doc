核心依然是 nginx  这个属于对nginx 的增强，包装。把 nginx 的11个 处理请求的阶段 模块进行可自定义扩展，提供较原先方便的扩展方式。 
在 nginx 里边写一些 业务代码   前端分担后端压力。
特别简单的业务，直接在nginx 层进行处理，不转发到 tomcat 处理。openResty 就是包装nginx 并且提供实现上述这些功能的组件/功能（借助 Lua 实现的），简单的几个场景：

- 布隆过滤器 redis ：nginx 直接访问redis；
- 日志报警，量太大的情况下，检索日志效率太慢，在nginx层返回报错日志的时候进行报警；在nginx 处理阶段的最后一个步骤 log
- 鉴权，也可以提到这里。kong 网关，基于openResty 开发
- 静态详情页也可以这样处理，不变动的资源。。。

OpenResty是一个基于 Nginx 与 Lua 的高性能 Web 平台，其内部集成了大量精良的 Lua 库、第 三方模块以及大多数的依赖项：用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关
OpenResty通过汇聚各种设计精良的 Nginx 模块(主要由 OpenResty 团队自主开发)，从而将 Nginx 有效地变成一个强大的通用 Web 应用平台，这样，Web 开发人员和系统工程师可以使用 Lua 脚 本语言调动 Nginx 支持的各种 C 以及 Lua 模块，快速构造出足以胜任 10K 乃至 1000K 以上单机并发连 接的高性能 Web 应用系统。
OpenResty的目标是让你的Web服务直接跑在Nginx服务内部，充分利用 Nginx 的非阻塞 I/O 模 型，不仅仅对 HTTP 客户端请求,甚至于对远程后端诸如 MySQL、PostgreSQL、Memcached 以及 Redis 等都进行一致的高性能响应。
##### nginx 处理请求的 11个阶段
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677477558535-dc17d9b5-5ff3-4748-85a3-4d2c6903bf1f.png#averageHue=%23529353&clientId=ud1ea3700-687c-4&from=paste&height=357&id=u2cb6541f&originHeight=714&originWidth=1592&originalType=binary&ratio=2&rotation=0&showTitle=false&size=520692&status=done&style=none&taskId=udf893b7b-3bdf-404d-8703-c694da50834&title=&width=796)

1. 当请求进入Nginx后先READ REQUEST HEADERS 读取头部 然后再分配由哪个指令操作 
2. Identity 寻找匹配哪个Location*
3. Apply Rate Limits 是否要对该请求限制
4. Preform Authertication 权限验证
5. Generate Content 生成给用户的响应内容
6. 如果配置了反向代理 那么将要和上游服务器通信 Upstream Services
7.  当返回给用户请求的时候要经过过滤模块 Response Filter
8. 发送给用户的同时 记录一个Log日志
| 阶段 | 描述 |
| --- | --- |
| post-read | 接收到完整的http头部后处理的阶段，在uri重写之前，一般跳过 |
| server-rewrite | location匹配前，修改uri的阶段，用于重定向，location块外的重写指令(多次执行) |
| find-config | uri寻找匹配的location块配置项(多次执行) |
| rewrite | 找到location块后再修改uri，location级别的uri重写阶段(多次执行 |
| post-rewrite | 防死循环，跳转到对应阶段 |
| preaccess | 权限预处理 |
| access | 判断是否允许这个请求进入 |
| post-access | 向用户发送拒绝服务的错误码，用来响应上一阶段的拒绝 |
| try-files | 访问静态文件资源 |
| content | 内容生成阶段，该阶段产生响应，并发送到客户端 |
| log | 记录访问日志 |

##### OpenResty 作用
在上述的处理阶段，提供一些组件功能
###### 常见命令
主要帮助对http请求取参、取header头、输出等

| 命令 | 描述 |
| --- | --- |
| ngx.arg | 指令参数，如跟在content_by_lua_file后面的参数 |
| ngx.var | request变量，ngx.var.VARIABLE引用某个变量,lua使用nginx内置 的绑定变量. ngx.var.remote_addr 为获取远程的地址， |
| ngx.ctx | 请求的lua上下文,每次请求的上下文，可以在ctx里记录，每次请 求上下文的一些信息，例如: request_id , access_key 等等 |
| ngx.header | 响应头，ngx.header.HEADER引用某个头 |
| ngx.status | 响应码 |
| ngx.log | 输出到error.log |
| ngx.send_headers | 发送响应头 |
| ngx.headers_sent | 响应头是否已发送 |
| ngx.resp.get_headers | 获取响应头 |
| ngx.is_subrequest | 当前请求是否是子请求 |
| ngx.location.capture | 发布一个子请求 |
| ngx.location.capture_multi | 发布多个子请求 |
| ngx.print | 输出响应 |
| ngx.say | 输出响应，自动添加‘\\n‘ |
| ngx.flush | 刷新响应 |
| ngx.exit | 结束请求 |

##### kong 网关
Kong是一款基于OpenResty(Nginx + Lua模块)编写的高可用、易扩展的，由Mashape公司开源的API Gateway项目
Kong是基于NGINX和Apache Cassandra或PostgreSQL构建的，能提供易于使用的RESTful API来 操作和配置API管理系统，所以它可以水平扩展多个Kong服务器，通过前置的负载均衡配置把请求均匀 地分发到各个Server，来应对大批量的网络请求。
云时代 api 网关，
###### API网关
API网关是一个服务器，是系统的唯一入口。从面向对象设计的角度看，它与外观模式类似。API 网关封装了系统内部架构，为每个客户端提供一个定制的API。它可能还具有其它职责，如身份验 证、监控、负载均衡、缓存、请求分片与管理、静态响应处理。API网关方式的核心要点是，所有 的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。通常，网关也 是提供REST/HTTP的访问API。
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677485439418-b250288c-b3f0-4118-85ac-d6776b150d81.png#averageHue=%23f8f8f7&clientId=ub0f6f2cb-2611-4&from=paste&height=540&id=uf0a6cb72&originHeight=1080&originWidth=1700&originalType=binary&ratio=2&rotation=0&showTitle=false&size=463468&status=done&style=none&taskId=u91847ac6-f31e-4b1d-8e48-c05d85bd261&title=&width=850)
在微服务架构之下，服务被拆分的非常零散，降低了耦合度的同时也给服务的统一管理增加了难度。如上图左所示，在旧的服务治理体系之下，鉴权，限流，日志，监控等通用功能需要再每个服务中单独实现，这使得系统维护者没有一个全局的视图来统一管理这些功能。API 网关致力于解决的问题便是为微服务管理这些通用的功能，在此基础上提供系统的可扩展性。如右图所示，微服务搭配上API 网关，可以使得服务本身更专注于自己的业务。
目前，比较流行的网关有:Nginx 、 Kong 、Orange等等，还有微服务网关Zuul 、Spring Cloud Gateway等等
对于 API Gateway，常见的选型有基于 Openresty 的 Kong、基于 Go 的 Tyk 和基于 Java 的 gateway，这三个选型本身没有什么明显的区别，主要还是看技术栈是否能满足快速应用和二次开发。
###### 和Spring Cloud Gateway区别
像Nginx这类网关，性能肯定是没得说，它适合做那种门户网关，是作为整个全局的网关，是对外 的，处于最外层的;而Gateway这种，更像是业务网关，主要用来对应不同的客户端提供服务的， 用于聚合业务的。各个微服务独立部署，职责单一，对外提供服务的时候需要有一个东西把业务聚 合起来。 像Nginx这类网关，都是用不同的语言编写的，不易于扩展;而Gateway就不同，它是用Java写 的，易于扩展和维护
Gateway这类网关可以实现熔断、重试等功能，这是Nginx不具备的
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677486185304-4002de9d-4595-4af2-aaff-ce670f89103f.png#averageHue=%23fce9c1&clientId=u78fdc4a9-a2f4-4&from=paste&height=512&id=u931342be&originHeight=1024&originWidth=1700&originalType=binary&ratio=2&rotation=0&showTitle=false&size=425446&status=done&style=none&taskId=u6aadbf4e-1dbe-4c8b-9e5a-ff5053a74c1&title=&width=850)
###### kong 的优势

- 插件市场丰富，很多插件可以降低开发成本; 
- 可扩展性，可以编写lua脚本来定制自己的参数验证权限验证等操作; 
- 基于openResty，openResty基于Nginx保障了强劲的性能; 
- 便捷性能扩容，只需要水平增加服务器资源性能就能提升 ; 
- 负载均衡健康检查

组成部分

- kong Server : 基于nginx 的服务器，用来接收API 请求
- Apache Cassandra/PostgreSql ： 用来存储操作数据。
- Kong dashboard：官方推荐的UI 管理工具

Kong采用插件机制进行功能定制，插件集(可以是0或N个)在API请求响应循环的生命周期中被执 行。插件使用Lua编写，目前已有几个基础功能:HTTP基本认证、密钥认证、CORS(Cross-Origin Resource Sharing，跨域资源共享)、TCP、UDP、文件日志、API请求限流、请求转发以及Nginx监 控。
特性

- 可扩展性：通过简单地添加更多的服务器，可以轻松的进行横向扩展，意味着平台可以在一个较低负载的情况下处理任何请求；
- 模块化：可以通过添加新的插件进行扩展，这些插件可以通过RestFul Admin Api 快速配置；
- 在任何基础架构上运行：kong 官网可以在任何地方都能运行，可以在云或者内部网络环境中部署kong，包括单个或多个数据中心配置，以及 public，private 或 invite-only APIS。

