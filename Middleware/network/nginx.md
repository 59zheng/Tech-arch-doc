高性能的 http 和反向代理服务，也是一个 IMAP/POP3/SMTP服务

1. 处理响应请求很快
2. 高并发连接
3. 低的内存消耗
4. 具有很高的可靠性
5. 高扩展性
6. 热部署

master管理进程与 worker 工程进程的分离设计，使得 Nginx 有热部署的功能。
##### Nginx 模板
整理采用模块化设计师 Nginx 的一个重大特点，甚至 http 服务器核心功能也是一个模块
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677289011359-db81a098-0d33-424b-bca5-b86aaf0b15ea.png#averageHue=%23ecece9&clientId=uff7e5e3b-290d-4&from=paste&height=349&id=ue7b1b0f2&originHeight=697&originWidth=1000&originalType=binary&ratio=2&rotation=0&showTitle=false&size=248038&status=done&style=none&taskId=ub126db2c-ee11-41e6-a098-cda99a5c171&title=&width=500)
##### nginx 应用场景
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677289088966-6d86071c-bf7c-4274-be5b-ca80619371d6.png#averageHue=%23f4f4f4&clientId=uff7e5e3b-290d-4&from=paste&height=275&id=ua5730d3f&originHeight=549&originWidth=1277&originalType=binary&ratio=2&rotation=0&showTitle=false&size=213918&status=done&style=none&taskId=uba102194-ee83-48ec-be17-30ba7a6f2ad&title=&width=638.5)
##### 两种代理方式
**正向代理：**客户端非常明确要访问的服务器地址：正向代理模式屏蔽或者隐藏了真实的客户端信息
**反向代理：**客户端是无感知dialing的存在的，反向代理，代理的事服务器，对外是透明的，访问者并不知道自己访问的是一个dialing，因为客户端不需要任何配置就可以访问。  反向代理，”它代理的是服务端，代服务端接收请求“，主要用于服务器集群分布式热部署的情况下，反向代理隐藏了服务器的信息。
##### nginx 配置
master 进程管理 worker 进程。维护个数，或者监控。不做用户相关处理。 
http-> server (虚拟主机) 不限制server 个数   
  server_name  多个相同的server_name可以共存，可以代理多个端口。
  sendfile/mmap 支持零拷贝。默认是开启状态；
   keepalive_timeout   长连接超时时间；
   location ： 域名后的地址匹配拦截
   root ： 访问资源的根路径 后面跟的是目录   alias 是全路径指定，不是从根目录起找的
   index  ： 默认的文件。

###### nginx 匹配规则：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677290602408-ea2e8e87-737b-46c2-a51e-069b8effdf23.png#averageHue=%23d7cc72&clientId=uff7e5e3b-290d-4&from=paste&height=519&id=ud5a4865e&originHeight=1038&originWidth=1924&originalType=binary&ratio=2&rotation=0&showTitle=false&size=601310&status=done&style=none&taskId=ue9cc3455-bdc9-43e1-bad6-7f2eb638e9f&title=&width=962)
###### 常见的nginx 配置模板（常规配置）
```shell
########### 每个指令必须有分号结束。#################
user administrator administrators; 
#配置用户或者组，默认为nobody nobody。 
worker_processes 2; #允许生成的进程数，默认为1
pid /nginx/pid/nginx.pid; #指定nginx进程运行文件存放地址
error_log log/error.log debug; #制定日志路径，级别。这个设置可以放入全局块，http块， server块，级别以此为:debug|info|notice|warn|error|crit|alert|emerg

events {

	accept_mutex on; #设置网路连接序列化，防止惊群现象发生，默认为on 
	multi_accept on; #设置一个进程是否同时接受多个网络连接，默认为off 
	#use epoll; #事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport 
	worker_connections 1024; #最大连接数，默认为512

} 

http {
	include mime.types; #文件扩展名与文件类型映射表
	default_type application/octet-stream; #默认文件类型，默认为text/plain #access_log off; #取消服务日志
	log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; #自定义格式
	
	access_log log/access.log myFormat; #combined为日志格式的默认值
	
	sendfile on; #允许sendfile方式传输文件 零拷贝 ，默认为off，可以在http块，server块， location块。
	
	sendfile_max_chunk 100k; #每个进程每次调用传输数量不能大于设定的值，默认为0，即不设上 限。
	
	keepalive_timeout 65; #连接超时时间，默认为75s，可以在http，server，location块。
    	
			upstream mysvr {
      server 127.0.0.1:7878;
			server 192.168.10.121:3333 backup; #热备 
			}
	error_page 404 https://www.baidu.com; #错误页 
	server {
		keepalive_requests 120; #单连接请求上限次数。
		listen 4545; #监听端口
		server_name 127.0.0.1; #监听地址
			location ~*^.+$ { #请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小
			#root path; #根目录
			#index vv.txt; #设置默认页
			proxy_pass http://mysvr; #请求转向mysvr 定义的服务器列表 deny 127.0.0.1; #拒绝的ip
			allow 172.18.5.54; #允许的ip
			} 
		}
}

```

###### 全局配置

- user:user是个主模块指令，指定Nginx Worker进程运行用户以及用户组，默认由nobody账号运行。

#这个地方如果写错了就会出现获取不到用户的错误

- worker_processes:是个主模块指令，指定了Nginx要开启的进程数，每个Nginx进程平均耗费10M~12M内存，建议指 定和CPU的数量一致即可。
- error_log: 是个主模块指令，用来定义全局错误日志文件，日志输出级别有debug、info、notice、warn、 error、crit可供选择，其中，debug输出日志最为最详细，而crit输出日志最少。

      #日志文件路径一般在nginx安装目录的logs目录中

- pid:是个主模块指令，用来指定进程pid的存储文件位置。

#进行成和nginx的master的进程号是一致的，只有nginx运行时才存在，如果nginx停止了 pid也会被删除掉
###### events事件指令
#events事件指令是设定Nginx的工作模式及连接数上限:

- use：use是个事件模块指令，用来指定Nginx的工作模式

#Nginx支持的工作模式有select、poll、kqueue、epoll、rtsig和/dev/poll，其中select和poll都是标 准的工作模式，kqueue和epoll是高效的工作模式，不同的是epoll用在Linux平台上，而kqueue用在 BSD系统中，对于Linux系统，epoll工作模式是首选

- worker_connections：也是个事件模块指令，用于定义Nginx每个进程的最大连接数，默认是1024。

#最大客户端连接数由worker_processes和worker_connections决定，即Max_client=worker_processes*worker_connections 在作为反向代理时，max_clients变为:max_clients = worker_processes * worker_connections/4。 进程的最大连接数受Linux系统进程的最大打开文件数限制，在执行操作系统命令“ulimit -n 65536”后 worker_connections的设置才能生效
###### HTTP服务器配置
```shell
http {
# 引入文件类型映射文件
include mime.types;
# 如果没有找到指定的文件类型映射 使用默认配置
default_type application/octet-stream;
# 日志打印格式
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
'$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" "$http_x_forwarded_for"';
# 启动零拷贝提高性能
sendfile on;
# 设置keepalive长连接超时时间
keepalive_timeout 65;
# 引入子配置文件
include /usr/local/openresty/nginx/conf/conf.d/*.conf;
}
```

- include：include是个主模块指令，实现对配置文件所包含的文件的设定，可以减少主配置文件的复杂度，可 以将其他各个模块的具体配置分散在不同的文件夹中。
- default_type：default_type属于HTTP核心模块指令，这里设定默认类型为二进制流，也就是当文件类型未定义时 使用这种方式，例如在没有配置PHP环境时，Nginx是不予解析的，此时，用浏览器访问PHP文件就会出 现下载窗口。
- log_format：log_format是Nginx的HttpLog模块指令，用于指定Nginx日志的输出格式，main为此日志输出格式 的名称，可以在下面的access_log指令中引用。
##### Nginx 路由匹配
###### 虚拟主机
#所谓虚拟主机，在 Web 服务里就是一个独立的网站站点，这个站点对应独立的域名(也可能是IP 或端口)，具有独立的程序及资源，可以独立地对外提供服务供用户访问。在 Nginx 中，使用一个 server{} 标签来标识一个虚拟主机，一个 Web 服务里可以有多个虚拟主机 标签对，即可以同时支持多个虚拟主机站点。虚拟主机有两种类型:基于域名的虚拟主机、基于IP+端口的虚拟主机。
 优先级

1. 完全匹配

server.name  www.abc.com  完全匹配域名

2. 通配符在前

server.name  *.com

3. 通配符在后

server.name  www.abc.*
**默认主机匹配**
默认虚拟主机的作用就是:如果有多个访问的域名指向这台web服务器，但某个域名未添加到 nginx虚拟主机中，就会访问默认虚拟主机(泛解析)
listen       80 default; 可将下方的 server.name 设置为默认
##### localtion 作用
 # location有“定位”的意思，根据请求不同的URL来进行不同的处理，在虚拟主机中(server)，location 配置是必不可少的，可以把网站不同的部分定位到不同的处理方式上。
###### 规则
  # location区段，通过指定模式来与客户端请求的URI相匹配
允许根据用户请求的URI来匹配定义的各location，匹配到时，此请求将被响应的location配置快中 的配置所处理，例如做访问控制等功能
语法 ： location [修饰符] pattern {...}
###### 常用的修饰符说明
| 修饰符 | 功能 |
| --- | --- |
| 空 | 前缀匹配 能够匹配以需要匹配的路径为前缀的uri |
| = | 精确匹配 |
| ~ | 正则表达式模式匹配，区分大小写 |
| ~* | 正则表达式模式匹配，不区分大小写 |
| ^~ | 精确前缀匹配，类似于无修饰符的行为，也是以指定模块开始，不同的是，如果模式匹配， 那么就停止搜索其他模式了，不支持正则表达式 |
| / | 通用匹配，任何请求都会匹配到。 |

- 前缀匹配：没有修饰符表示必须以指定模式开始,指定模式前面没有任何修饰符，直接在location后写需要匹配 的uri，它的优先级次于正则匹配
```
server {
    server_name www.abc.com;
    charset   utf-8;
    location /abc {
        default_type text/html;
		echo "前缀匹配-abc...如 www.abc.com/abc www.abc.com/abc/  www.abc.com/abc.html"; }
}
```

- 精确匹配：精确匹配使用 = 表示，nginx进行路由匹配的时候，精确匹配具有最高的优先级，请求一旦精确 匹配成功nginx会停止搜索其他到匹配项
```
server {
    server_name www.abc.com;
    charset   utf-8;
    location = /abc {
        default_type text/html;
echo "精确匹配-abc-accurate"; }
# 那么如下内容可正确匹配:
#  www.abc.com/abc www.abc.com/abc?....
#  如下内容则无法匹配:
#  www.abc.com/abc/ www.abc.com/abc/adcde
}
```

- 精确前缀匹配：精确前缀匹配的优先级仅次于精确匹配，nginx对一个请求精确前缀匹配成功后，停止继续搜索其 他到匹配项
```
server {
    server_name www.itcast.com;
    charset   utf-8;
    location ^~ /abc {
        default_type text/html;
echo "精确前缀匹配-abc-prefix"; }

#那么如下内容可以就可以正确匹配:
# www.itcast.com/abc www.itcast.com/abc/ www.itcast.com/abc?....
}
```

- 正则表达式：正则匹配分为区分大小写和不区分大小写两种，分别用 ~ 和 ~* 表示;一个请求精确匹配和精确前 缀匹配都失败后，如果配置有相关的正则匹配location，nginx会尝试对该请求进行正则匹配。需 要说明的是正则匹配之间没有优先级一说，而是按照在配置文件中出现的顺序进行匹配，一旦匹配 上一个，就会停止向下继续搜索
   - 区分大小写： ~:表示指定的正则表达式要区分大小写
   - 不区分大小写： ~*:表示指定的正则表达式不区分大小写
- 通用匹配：通用匹配使用一个 / 表示，可以匹配所有请求，一般nginx配置文件最后都会有一个通用匹配规 则，当其他匹配规则均失效时，请求会被路由给通用匹配规则处理;如果没有配置通用匹配，并且 其他所有匹配规则均失效时，nginx会返回 404 错误
###### 匹配顺序：
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677470733343-f692a0ad-ed86-4ed0-a96f-b8b5b69425f7.png#averageHue=%23f6f6f6&clientId=u223418bb-3e12-4&from=paste&height=391&id=u4ab82e73&originHeight=782&originWidth=1932&originalType=binary&ratio=2&rotation=0&showTitle=false&size=286829&status=done&style=none&taskId=u40448eca-7d70-44f6-9567-7758faeac09&title=&width=966)
###### path 进入 匹配的过程
![image.png](https://cdn.nlark.com/yuque/0/2023/png/21492435/1677470812869-5bec7b1d-44da-4d2e-865d-8bf921913bdc.png#averageHue=%23f2f2ec&clientId=u223418bb-3e12-4&from=paste&height=885&id=ub5b8c5ac&originHeight=1770&originWidth=1914&originalType=binary&ratio=2&rotation=0&showTitle=false&size=639282&status=done&style=none&taskId=uf982479e-c6ee-4651-b751-cbaf7281e08&title=&width=957)

- 将整个url拆解为域名/端口/path/params
- 先由域名/端口，对应到目标server虚拟主机
- path部分参与location匹配，path = path1匹配部分 + path2剩余部分 进入location方法体内部流程。 
- 若是静态文件处理，则进入目标目录查找文件:root指令时找path1+path2对应的文件;alias指令 时找path2对应的文件 
- 若是proxy代理，则形如proxy_pass=ip:port时转发path1+path2路径到tomcat;形如 proxy_pass=ip:port/xxx时转发path2路径到tomcat。params始终跟随转发。
###### 使用建议
```
#直接匹配网站根，通过域名访问网站首页比较频繁，使用这个会加速处理，官网如是说。 #这里是直接转发给后端应用服务器了，也可以是一个静态首页
# 第一个必选规则
location = / {
    proxy_pass http://tomcat:8080/index
}
# 第二个必选规则是处理静态文件请求，这是nginx作为http服务器的强项
# 有两种配置模式，目录匹配或后缀匹配,任选其一或搭配使用 

location ^~ /static/ {
    alias /webroot/static/;
}

location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
    root /webroot/res/;
}
# 第三个规则就是通用规则，用来转发动态请求到后端应用服务器 # 非静态文件请求就默认是动态请求，自己根据实际把握
# 毕竟目前的一些框架的流行，带.php,.jsp后缀的情况很少了 

location / {
    proxy_pass http://tomcat:8080/
}
```
##### nginx 负载均衡
负载均衡用于从“upstream”模块定义的后端服务器列表中选取一台服务器接受用户的请求，一个最基本的upstream模块是这样的，模块内的server是服务器列表:
```
#动态服务器组
upstream dynamicserver {
  server 172.16.44.47:9001; #tomcat 1
  server 172.16.44.47:9002; #tomcat 2
  server 172.16.44.47:9003; #tomcat 3
  server 172.16.44.47:9004; #tomcat 4
}
```
在upstream模块配置完成后，要让指定的访问反向代理到服务器列表:
```
#其他页面反向代理到tomcat容器 
location ~.*$ {
  index index.jsp index.html;
  proxy_pass http://dynamicserver;
}
```
这就是最基本的负载均衡实例，但这不足以满足实际需求;目前Nginx服务器的upstream模块支持6种 方式的分配
###### 常用参数
| 参数 | 描述 |
| --- | --- |
| server | 反向服务地址 加端口 |
| weight | 权重 |
| fail_timeout | 与max_fails结合使用。 |
| max_fails | 设置在fail_timeout参数设置的时间内最大失败次数，如果在这个时间内，所有针 对该服务器的请求都失败了，那么认为该服务器会被认为是停机了 |
| max_conns | 允许最大连接数 |
| fail_time | 服务器会被认为停机的时间长度,默认为10s |
| backup | 标记该服务器为备用服务器，当主服务器停止时，请求会被发送到它这里 |
| down | 标记服务器永久停机了 |
| slow_start | 当节点恢复，不立即加入 |

###### 负载均衡策略（仅nginx 自带，三方不包括）
| 负载均衡 | 描述 |
| --- | --- |
| 轮训 | 默认方式 |
| weight | 权重方式 |
| ip_hash | 依据ip分配方式 |
| least_conn | 最少连接方式 |
| fair(第三方) | 响应时间方式 |
| url_hash(第三方) | 依据URL分配方式 |

- 轮训：最基本的配置方法，上面的例子就是轮询的方式，它是upstream模块默认的负载均衡默认策略， 每个请求会按时间顺序逐一分配到不同的后端服务器。

注意： 

   - 在轮询中，如果服务器down掉了，会自动剔除该服务器。
   -  缺省配置就是轮询策略。 
   - 此策略适合服务器配置相当，无状态且短平快的服务使用。
- 权重方式：在轮询策略的基础上指定轮询的几率

配置方式  server 192.168.64.1:9001  weight=2;
weight参数用于指定轮询几率，weight的默认值为1,;weight的数值与访问比率成正比，比如 Tomcat 7.0被访问的几率为其他服务器的两倍。
 注意

   - 权重越高分配到需要处理的请求越多。 
   - 此策略可以与least_conn和ip_hash结合使用。 
   - 此策略比较适合服务器的硬件配置差别比较大的情况。
- ip_hash：指定负载均衡器按照基于客户端IP的分配方式，这个方法确保了相同的客户端的请求一直发送到相 同的服务器，以保证session会话，这样每个访客都固定访问一个后端服务器，可以解决session不 能跨服务器的问题
```
upstream dynamicserver {
ip_hash; 
#保证每个访客固定访问一个后端服务器 
server 192.168.64.1:9001 weight=2; 
server 192.168.64.1:9002;
server 192.168.64.1:9003;
server 192.168.64.1:9004; #tomcat 4
}
```
注意：

   - 在nginx版本1.3.1之前，不能在ip_hash中使用权重(weight)。
   -  ip_hash不能与backup同时使用 
   - 此策略适合有状态服务，比如session。 
   - 当有服务器需要剔除，必须手动down掉。
- least_conn：把请求转发给连接数较少的后端服务器，轮询算法是把请求平均的转发给各个后端，使它们的负载 大致相同;但是，有些请求占用的时间很长，会导致其所在的后端负载较高，这种情况下， least_conn这种方式就可以达到更好的负载均衡效果。
```
upstream dynamicserver {
least_conn; #把请求转发给连接数较少的后端服务器
server 192.168.64.1:9001 weight=2;
server 192.168.64.1:9002;
server 192.168.64.1:9003;
server 192.168.64.1:9004; #tomcat 4
}
```
注意： 此负载均衡策略适合请求处理时间长短不一造成服务器过载的情况。
##### 重试策略
###### 基础的失败重试
```
upstream dynamicserver {
      server 192.168.64.1:9001 fail_timeout=60s max_fails=3; #Server A
      server 192.168.64.1:9002 fail_timeout=60s max_fails=3; #Server B
}
```
max_fails=3 fail_timeout=60s 代表在 60 秒内请求某一应用失败 3 次，认为该应用宕机，后等 待 60 秒，这期间内不会再把新请求发送到宕机应用，而是直接发到正常的那一台，时间到后再有请求进 来继续尝试连接宕机应用且仅尝试 1 次，如果还是失败，则继续等待 60 秒...以此循环，直到恢复。

###### 错误重试
在系统出现 500 等异常的情况下，希望nginx能够到其他的服务器进行重试，我们可以配 置那些错误码才进行重试
**配置说明**
在nginx的配置文件中，proxy_next_upstream项定义了什么情况下进行重试，官网文档中给出的 说明如下
```
Syntax: proxy_next_upstream error | timeout | invalid_header | http_500 |
http_502 | http_503 | http_504 | http_403 | http_404 | off ...;
Default:    proxy_next_upstream error timeout;
Context:    http, server, location
```
配置了 500 等错误的时候会进行重试 示例
```
upstream dynamicserver {
  server 192.168.64.1:9001 fail_timeout=60s max_fails=3; #tomcat 1
  server 192.168.64.1:9002 fail_timeout=60s max_fails=3; #tomcat 2
}
server {
        server_name www.itcast.com;
        default_type text/html;
        charset   utf-8;
location ~ .*$ {
index index.jsp index.html;
proxy_pass http://dynamicserver;
#下一节点重试的错误状态
proxy_next_upstream error timeout http_500 http_502 http_503
http_504; }
}
```
###### backup 服务器
Nginx 支持设置备用节点，当所有线上节点都异常时启用备用节点，同时备用节点也会影响到失败重试的逻辑，因此单独列出来介绍。
**backup 处理逻辑**

1. 正常情况下，请求不会转到到 backup 服务器，包括失败重试的场景
2. 当所有正常节点全部不可用时，backup 服务器生效，开始处理请求
3. 一旦有正常节点恢复，就使用已经恢复的正常节点
4. backup 服务器生效期间，不会存在所有正常节点一次性恢复的逻辑
5. 如果全部 backup 服务器也异常，则会将所有节点一次性恢复，加入存活列表 6. 如果全部节点(包括 backup)都异常了，则 Nginx 返回 502 错误
###### 限制重试方式
 # 默认配置是没有做重试机制进行限制的，也就是会尽可能去重试直至失败，Nginx 提供了以下两个 参数来控制重试次数以及重试超时时间

- proxy_next_upstream_tries :设置重试次数，默认 0 表示无限制，该参数包含所有请求 upstream server 的次数，包括第一次后之后所有重试之和;
- proxy_next_upstream_timeout :设置重试最大超时时间，默认 0 表示不限制，该参数指的是 第一次连接时间加上后续重试连接时间，不包含连接上节点之后的处理时间


##### 常见案例
###### 代理静态文件
```
server {
        listen       10086;
				server_name  www.abc.com;
 location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-NginX-Proxy true;
        proxy_redirect off;
        }
	location /data/ {
		alias '/usr/local/data/'; #这里是重点，就是代理这个文件夹 expires 7d;
		} 
}
```
###### 反向代理
```
server {
    listen       80;
    server_name  www.itcast.com;;
    location / {
        proxy_pass http://127.0.0.1:8080;
        index  index.html index.htm .jsp;
	} 
}
```
###### 跨域配置
```
server {
        listen       80;
        server_name  www.itcast.com;
	if ( $host ~ (.*).itcast.com){ 
				set $domain $1;##记录二级域名值
			}
	#是否允许请求带有验证信息
	add_header Access-Control-Allow-Credentials true; #允许跨域访问的域名,可以是一个域的列表，也可以是通配符*
	add_header Access-Control-Allow-Origin *;
	#允许脚本访问的返回头
	add_header Access-Control-Allow-Headers 'x-requested-with,content-
	type,Cache-Control,Pragma,Date,x-timestamp'; #允许使用的请求方法，以逗号隔开
	add_header Access-Control-Allow-Methods 'POST,GET,OPTIONS,PUT,DELETE'; #允许自定义的头部，以逗号隔开,大小写不敏感
	add_header Access-Control-Expose-Headers 'WWW-Authenticate,Server-
	Authorization';
	#P3P支持跨域cookie操作
	add_header P3P 'policyref="/w3c/p3p.xml", CP="NOI DSP PSAa OUR BUS IND ONL UNI COM NAV INT LOC"';
	if ($request_method = 'OPTIONS') {##OPTIONS类的请求，是跨域先验请求

	return 204;##204代表ok

	} 
}
```
##### 防盗链
通过Referer实现防盗链比较基础，仅可以简单实现方式资源被盗用，构造Referer的请求很容易实现
**场景**: 由于图片链接可以跨域访问，所以图片链接往往被其他网站盗用，从而增加服务器负担; 
**解决方案**:nginx可以通过valid_referers配置进行防盗链配置
###### valid_referers 指令
指定合法的来源'referer', 他决定了内置变量$invalid_referer的值，如果referer头部包含在这个合 法网址里面，这个变量被设置为0，否则设置为1. 需要注意的是:这里并不区分大小写的.
语法: valid_referers none | blocked | server_names | string ...;  
配置段: server, location
**配置说明**

- none : 允许没有http_refer的请求访问资源;
- blocked : 允许不是http://开头的，不带协议的请求访问资源; 
- 192.168.0.1 : 只允许指定ip来的请求访问资源; 
- *.google.com:允许 *.google.com 的域名请求访问资源

示例
```
# 需要防盗的后缀
location ~* \.(jpg|jpeg|png|gif|bmp|swf|rar|zip|doc|xls|pdf|gz|bz2|mp3|mp4|flv)$
#设置过期时间
expires 30d;
# valid_referers 就是白名单的意思
# 支持域名或ip
# 允许ip 192.168.0.1 的请求
# 允许域名 *.google.com 所有子域名
valid_referers none blocked 192.168.0.1 *.google.com; if ($invalid_referer) {
# return 403;
# 盗链返回的图片，替换盗链网站所有盗链的图片 rewrite ^/ https://site.com/403.jpg;
}
    root  /usr/share/nginx/img;
}
```
