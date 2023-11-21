code#### 什么是k8s ？
kubermetes 的简称，本质上是一个开源的容器编排系统，主要用于管理容器化的应用，目标是让部署容器化的应用简单并且高效（powerful），Kubernetes 提供了应用部署，规划，更新，维护的一种机制。
简单说：k8s 是一个编排容器的系统，可以管理容器应用全生命周期的工具，从创建应用，应用的部署，应用提供服务，扩容缩容应用，应用更新，都非常方便，并且可以做到故障自愈。

#### k8s 组件有哪些，作用分别是什么？
k8s 主要由 master 节点和 node 节点构成。
master 节点负责管理集群，node 节点是容器应用真正运行的地方。
master 节点包含的组件有：kube-api-server 、 kube-controller-manager、kube-scheduler、etcd。
node 几点包含的组件有：kubelet、kube-proxy、container-runtime。
##### kube-api-server
api-server 是 k8s 最核心的组件之一，它是k8s集群的统一访问入口，提供了 RESTful api 接口，实现了认证，授权和准入控制等安全功能；api-server 还是其他组件之间的数据交互和通信的枢纽，其他组件彼此之间并不会直接通信，其他组件对资源对象的着呢个，删，改，查和监听操作都是由 api-server 处理后，再提交给 etcd 数据库持久化存储，只有api-server 才能直接操作 etcd 数据库，其他组件都是api-server 间接的读取，写入数据到etcd。

##### kube-controller-manager：
controller-manager 是 k8s 中各种控制器的管理者，是k8s 集群内部的管理控制中，也是k8s 自动化功能的核心；controller-manager 内部包含 replication contaoller、node controller、deployment controller、endpoint controller 等各种资源对象的控制器，每种控制器都负责一种特定资源的控制流程，而 controller-manager 正是这些 controller 的核心管理者。
##### kube-scheduler：
scheduler 是负责集群资源调度，其作用是将待调度的pod 通过一系列负责的调度算法计算出最合适的node 节点，然后将pod 绑定到目标节点上。shceduler 会根据 pod 的信息，全部节点信息列表，过滤掉不符合要求的节点，过滤出一批获选节点，然后给候选节点打分，选分最高的就是最佳节点。

##### etcd：
etcd 是一个分布式键值对存储数据库，主要是用于保存k8s集群状态数据，比如：pod ，service 等资源对象的信息；etcd 可以是单个也可以是多个，多个就是 etcd 数据库集群、etcd 通常部署奇数个实例，在大规模集群中，etcd 有5个或者 7个节点就足够了；etcd 本质上可以不与 master 节点部署在一起，只要master 节点能通过网络连接 etcd 数据库即可。

##### kubelet：
每个node 节点上都有一个kubelet 服务进程，kubelet 作为连接 master 和node之间的桥梁，负责维护 pod 和容器的生命周期，当监听到 master 下发到本节点的任务时，比如创建，更新，终止pod等任务，kubelet 即通过控制docker 来创建、更新、销毁容器；

##### kube-proxy
kube-proxy 运行在node 节点上，在 Node 节点上实现 pod 网络代理，维护网络规则和四层负载均衡工作，kube-proxy 会监听 api-server 中从而获取service 和endpoint 的变化情况，创建并维护路由规则以提供服务IP和负载均衡功能。简单理解进程是 service 的透明代理兼负载均衡器，核心功能是将某个 service 的访问请求转发到后端的多个 pod 实例上。
##### container-runtime：
容器运行时环境，即运行容器所需要的一系列程序，目前 k8s 支持的容器运行时有很多，如 docker 、rkt 或其他 ，最新版的 k8s 已经弃用docker 。

#### kubernetes 的相关基本概念
##### master
k8s 集群的管理节点，负责管理集群，提供集群的资源数据访问入口，拥有 Etcd 存储服务（可选），运行api server 进程，Controller manager 服务进程 及 scheduler 服务进程；

##### node（worker）：
node（worker）是 kubernetes 集群架构中原型pod 的服务节点，是 Kubernetes 集群操作的单元，用来承载被分配 pod 的运行，是pod 运行的宿主机。运行 docker eninge 服务，守护进程 kunelet 以及负载均衡器 kube-proxy；
##### pod
运行于 Node 节点中，若干相关容器的组合。pod 内包含的容器运行在同一宿主机上，使用相同的网络命名空间，IP地址和端口，能够通过 localhost 进行通信，pod 是 kurbernetes 进行创建、调度和管理的最小单位，提供了比容器更高层次的抽象，使部署和管理更加灵活。一个pod 可以包含一个容器或多个容器。
##### label
kubernetes 中label 实质上是一系列 Key/Value 键值对，其中key 与value 可自定义。 Label 可以附加到各种资源对象上，如 node 、pod、Service、PC等。一个资源对象可以定义任意数量的Label，同一个 Lable 也可以被添加到任意数量的资源对象上去。Kubernetes 通过 Label Selector（标签选择器）查询和筛选资源对象；

##### Replication Controller
Replication Controller 用来管理 Pod 的副本，保证集群中存在指定数量的 pod 副本。
集群中副本的数量大于指定数量，则会停止指定数量之外的多余容器数量。反之，则会启动少于指定数量个数的容器，保证数量不变。
Replication controller 是实现弹性伸缩，动态扩容和滚动升级的核心；
##### Deployment
deployment 在内部使用了 RS 来实现目的，Deployment 相当于 RC的一次升级，最大的特点是可以随时获取当前pod 的部署进度；

##### HPA（Horizontal pod Autoscaler）
pod 的横向自动扩容，也是 kubernetes 的一种资源，通过追踪分析 RC 控制的所有pod 目标的负载情况，来确定是否需要针对性的调整 Pod 副本数量；

##### Service：
Service 定义了pod 的逻辑集合和访问该集合的策略，是真实服务的抽象。
Service 提供了一个同一的服务访问入口以及服务代理和发现机制，关联多个相同 Label 的Pod ，用户不需要了解后台 POd 是如何运行的；
##### Volume：
Volume 是pod 中能够被多个容器访问的共享目录，Kubernetes 是 Volume 是定义在 pod 上，可以被一个或多个 pod 中容器挂载到某个目录下；

##### Namespace
Nmaespace 用于实现多租户的资源隔离，可将集群内部的资源对象分配到不同的 Namespace 中，形成逻辑上的不同项目，小组或者用户组、便于不同的Namespace 在共享使用整个集群的资源的同时还能被分别管理。
#### 简述Kubernetes 和Docker 的关系？
Docker 是开源的容器引擎，一种更加轻量级的虚拟化技术；
k8s，容器管理工具，用来管理容器 pod 的集合，可以实现容器集群的自动化部署，自动化扩缩容，维护等功能；

#### 简述 kubernetes 如何实现集群管理？
在集群管理方面，kubernetes 将集群中的机器划分为一个Master 节点和 一群工作节点 Node。
其中，在Master节点运行着集群管理相关的 一组进程 kube-apiserver，kube-controller-manager和kube-scheduler，这些进程实现了整个集群的资源管理、pod 管理、弹性伸缩、安全控制、系统监控和纠错等管理能力，并且都是全自动完成的。
#### kubernetes 的优势，使用场景极其特点？
优势：容器编排、轻量级、开源、弹性伸缩、负载均衡；
场景：快速部署应用、快速扩展应用、无缝对接新的应用功能、节省资源，优化硬件资源的使用；
特点：

- 可移植: 支持公有云、私有云、混合云、多重云（multi-cloud）、
- 可扩展: 模块化,、插件化、可挂载、可组合、
- 自动化: 自动部署、自动重启、自动复制、自动伸缩/扩展
#### kubernetes 中什么是 Minikube、Kubectl、Kubelet？
Minkube 是一种可以在本地轻松运行运行一个单节点 kubernetes 集群的工具；
kubectl 是一个命令行工作，可以使用该工作控制 kubernetes 集群管理器，如检查集群资源，创建，删除和更新组件，查看应用程序；
kubelet 是一个代理服务，在每个节点上原型，并使从服务器与主服务器通信；

#### kubelet 的功能，作用是什么、
kubelet 部署在每个node 节点上，主要有4个功能；

- 节点管理

kubelet 启动时会向 api-server 进行注册，然后会定时的想 api-server 汇报本节点信息状态，资源使用状态等，这样master 就能知道 node 节点的资源剩余，节点是否失联等相关的信息了。master 知道了整个集群所有节点的资源情况，对于 pod 的调度和正常运行比较重要。

- pod管理

kubelet 负责维护node 节点上 pod 的生命周期，当 kubelet 监听到 master 的下发到自己节点的任务时，比如要创建、更新、删除一个pod，kubelet 就会通过 CRI（容器运行时接口）插件来调用不同的容器运行时来创建、更新、删除容器；常见的容器运行时有 docker 、 containerd、rkt 等这些容器运行时，最熟悉的就是docker了，但在新版本的k8s已经弃用docker了，k8s1.24版本中已经使用containerd作为容器运行时了。

- 容器健康检查

pod 可以自定义启动探针、存活探针、就绪探针等3种，常用的就是存活探针、就绪探针，kubelet 会定期调用容器中的探针来检测容器是否存活，是否就绪，如果是存活探针，则会根据探测结果对检查失败的容器进行相应的重启策略；

- Metrics Server 资源监控。

在node 节点上部署 Metrics Server 用于监控 node 节点、pod 的cpu、内存、文件系统、网络使用等资源使用情况，而 kubelet 则通过 Metrics Server 获取所在节点及容器上的数据。
#### kube-api-server 的端口是多少？各个 pod 是如何访问 kube-api-server 的？
kube-api-server 的端口是8080 和6443，前者是 http 的端口，后者是 https 的端口。
在命名空间的kube-system命名空间里，有一个名称为kube-api-master的pod，
这个pod就是运行着kube-api-server进程，它绑定了master主机的ip地址和6443端口，但是在default命名空间下，存在一个叫kubernetes的服务，该服务对外暴露端口为443，目标端口6443，
这个服务的ip地址是clusterip地址池里面的第一个地址，同时这个服务的yaml定义里面并没有指定标签选择器，
也就是说这个kubernetes服务所对应的endpoint是手动创建的，该endpoint也是名称叫做kubernetes，该endpoint的yaml定义里面代理到master节点的6443端口，也就是kube-api-server的IP和端口。
这样一来，其他pod访问kube-api-server的整个流程就是：pod创建后嵌入了环境变量，pod获取到了kubernetes这个服务的ip和443端口，请求到kubernetes这个服务其实就是转发到了master节点上的6443端口的kube-api-server这个pod里面。
#### k8s 中命名空间的作用是什么？
namespace 是 kubernetes 系统中的一种非常重要的资源，namespcae 的主要作用是用来实现多套环境的资源隔离，或者说是多租户的资源隔离。
k8s 通过将集群内部的资源分配到不同的namespace 中，可以形成逻辑上的隔离，以方便不同的资源进行隔离使用和管理。
不同的命名空间可以存在同名的资源，命名空间为资源提供了一个作用域。
可以通过k8s的授权机制，将不同的namespace 交给不同的租户进行管理，可以实现多租户的资源隔离，还可以结合k8s的资源配额机制，限定不同租户能占用的资源，例如 cpu使用量，内存使用量等等可以实现租户可用资源的管理。
#### k8s 提供了大量的 Rest 接口，其中一个是 kubernetes proxy api 接口，作用以及使用。
kubernetes proxy api接口，从名称中可以得知，proxy是代理的意思，其作用就是代理rest请求；
Kubernets API server 将接收到的rest请求转发到某个node上的kubelet守护进程的rest接口，由该kubelet进程负责响应。
我们可以使用这种Proxy接口来直接访问某个pod，这对于逐一排查pod异常问题很有帮助。
```java
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/pods  	#查看指定node的所有pod信息
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/stats  	#查看指定node的物理资源统计信息
http://<kube-api-server>:<api-sever-port>/api/v1/nodes/node名称/proxy/spec  	#查看指定node的概要信息

http://<kube-api-server>:<api-sever-port>/api/v1/namespace/命名名称/pods/pod名称/pod服务的url/  	#访问指定pod的程序页面
http://<kube-api-server>:<api-sever-port>/api/v1/namespace/命名名称/servers/svc名称/url/  	#访问指定server的url程序页面

```


#### pod 是什么？
在 kubernetes 中，k8s 不直接处理容器，而是使用多个容器共存的理念，这组容器叫做 pod 。pod 是 k8s 中可以创建和管理的最小单元，是资源对象模型中由用户创建和部署的最小资源对象模型，其他的资源对象都是用来支撑pod 对象功能的，比如，pod 控制器就是用来管理pod对象的，service 或者 imgress 资源对象是用力啊暴露pod 引用对象的，persistentvolume 资源是用来为pod 提供存储的。
k8s 不会直接处理容器，而是 pod ，pod 才是k8s可以创建和管理的最小单元，也是基本单元。
#### pod 的原理是什么？
在微服务的概念中，一般的，一个容器会被设计为一个进程，除非进程本身产生子进程。
这样，由于不能将多个进程聚集在同一个单独的容器中，需要一个更高级的结果将容器绑定在一起，并将它们作为一个单元进行管理，这就是 k8s 中 pod 的背后原理。

#### pod 特点

- 每个pod 就像一个独立的逻辑极其，k8s 会为每个 pod 分配一个集群内部唯一的 ip地址，所以每个pod 都拥有 自己的 ip 地址，主机名，进程等；
- 一个 pod 可以包含1个或多个容器，1个容器一般被设计为只运行一个进程，1个pod 只可能运行在单个节点上，即不可能1个 pod 跨节点运行，pod 的生命周期是短暂的，也就是说 pod 可能随时被消亡（如节点异常，pod 异常等情况）
- 每一个 pod 都有一个特殊的被成为 “根容器”的 pause 容器，也称为 info 容器，pause 容器对应的镜像属于 k8s 平台的一部分，除了 pause 容器，每个 pod 还包含一个或多个 跑业务相关组件的应用容器；
- 一个 pod 中的容器共享 network 命名空间；
- 一个pod 里的多个容器共享 pod IP，意味着1个 pod 里面的多个容器的进程所占用的端口不能相同，否则在这个pod 里面就会产生端口冲突；既然每个 pod 都有自己的 IP 和端口空间，name对不同的两个 pod 来说就不能存在端口冲突；
- pod 是k8s 扩容，缩容的基本单位，也就是说 k8s 扩容缩容是针对 pod 而言而非容器。
#### pod 重启策略有哪些？
pod 重启容器策略是指针对pod 内所有容器的重启策略，不是重启pod，其可以通过 restartPolicy 字段配置重启容器的策略。

- Always：当容器终止退出后，总是重启容器，默认策略就是Always。
- OnFailure：当容器异常退出，退出状态码非0时，才重启容器。
- Never：当容器终止退出，不管退出状态码是什么，从不重启容器。

#### pod 镜像拉取策略有哪几种？
pod 镜像拉取策略可以 通过 imagePullPolicy 字段配置镜像拉取策略。
主要有3种镜像拉取策略，如下：

- ifNotPresent：默认值，镜像在node节点宿主机上不存在才拉取。
- Always ： 总是重新拉取，即每次创建 pod 都会重新从镜像仓库拉取一次镜像。
- Never： 永远不会主动拉取镜像，仅使用本地镜像，需要手动拉取镜像到 node 节点，如果 node 节点不存在镜像 则 pod 启动失败。
#### kubenetes 针对 pod 资源对象的健康检测机制？
提供三类 probe（探针）来执行对pod 的健康检测

- livenessProbe 探针（存活探针）

可以根据胡勇自定义规则来判断pod 是否健康，用于判断容器是否处于Running 状态，如果不是，kubelet 就会杀死该容器，并根据重启策略做相应的处理，如果容器不包含该探针，namekubelet 就会返回默认值，都是success；

- ReadinessProbe 探针：

同样是可以根据用户自定义规则来判断pod 是否健康，容器服务是否可用（Ready）。如果探测失败，控制器会将此pod从对应的 service 的 endpoint 列表中移除，从此不再讲任何请求调度到此pod上，直到下次探测成功

- startupProbe探针:

启动检查机制，应用一些启动缓慢的业务，避免业务长时间启动而被上面两类探针kill掉，
这个问题也可以换另一种方式解决，就是定义上面两类探针机制时，初始化时间定义的长一些即可;
备注：每种探测方法能支持以下几个相同的检查参数，用于设置控制检查时间：

   - initialDelaySeconds：初始第一次探测间隔，用于应用启动的时间，防止应用还没启动而健康检查失败；
   - periodSeconds：检查间隔，多久执行probe检查，默认为10s；
   - timeoutSeconds：检查超时时长，探测应用timeout后为失败；
   - successThreshold：成功探测阈值，表示探测多少次为健康正常，默认探测1次。


#### 就绪探针（ReadinessProbe 探针）与存活探针（livenessProbe探针）区别是什么？
两者作用不一样，
存活探针是将检查失败的容器杀死，创建新的启动容器来保持 pod 正常工作；
就绪探针是，当就绪探针检测失败，并不重启容器，而是将 pod 移出 endpoint ，就绪探针确保了service 汇总的 pod 都是可用的，确保客户端只与正常的pod交互并且客户端永远不会知道系统存在问题。
#### 存活探针的属性参数有哪几个？
存活探针的附加属性参数有以下几个：

- initialDelaySeconds：表示在容器启动后延时多少秒才开始探测；
- periodSeconds：表示执行探测的频率，即间隔多少秒探测一次，默认间隔周期是10秒，最小1秒；
- timeoutSeconds：表示探测超时时间，默认1秒，最小1秒，表示容器必须在超时时间范围内做出响应，否则视为本次探测失败；
- successThreshold：表示最少连续探测成功多少次才会被认定为成功，默认是1，对于liveness必须是1，最小值是1；
- failureThreshold：表示连续探测失败多少次才能被认定为失败，默认是3，连续3次失败，k8s将根据pod 重启策略对容器做出决定；

注意：定义存活探针时，一定要设置 initalDelaySeconds 属性，该属性为初始延时，如果不设置，默认容器启动时探针就开始探测了，这样可能会存在应用程序还未启动就绪，就会导致探针检测失效，k8s 就会更具 pod 重启策略杀掉容器然后再重新创建容器的 问题。
#### pod 的就绪探针有哪几种？
pod 启动后，就会立即加入 service 的endpoint ip 列表中，并开始接受到客户端的链接请求，如果此时pod中容器的业务进程还没有初始化完毕，name这些客户端链接请求就会失败，为了解决这个问题，kubernetes 提供了就绪探针来解决这个问题。
在pod中的容器定义一个就绪探针，就绪探针周期性检查容器，如果就绪探针检查失败嘞，说明该pod还未准备就绪，不能接受客户端链接，则将该pod从endpoint列表中移除，pod 被剔除了，service 就不会把请求分发给该 pod，然后就绪探针继续检查，如果随后容器就绪，则再重新把 pod 加回 endpoint 列表。
kubernetes 提供了 3中探测容器的存活探针，如下：

- httpGet：通过容器的IP、端口、路径发送http请求，返回200-400范围内的状态码表示成功。
- exec ：在容器内执行 shell 命令，根据命令退出状态码是否为0进行判断，0表示健康，非0表示不健康。
- TCPSocket：与容器的IP、端口建立TCP Socket 链接，能建立则说明探测成功，不能建立则说明探测失败。
#### pod 的就绪探针的属性参数有哪些
就绪探针的附加属性参数有以下几个：

- initialDelaySeconds：延时秒数，即容器启动多少秒后才开始探测，不写默认容器启动就探测；
- periodSeconds ：执行探测的频率（秒），默认为10秒，最低值为1；
- timeoutSeconds ：超时时间，表示探测时在超时时间内必须得到响应，负责视为本次探测失败，默认为1秒，最小值为1；
- failureThreshold ：连续探测失败的次数，视为本次探测失败，默认为3次，最小值为1次；
- successThreshold ：连续探测成功的次数，视为本次探测成功，默认为1次，最小值为1次；

#### pod 的重启策略是什么
通过命令 kubectl explain pod.spec 查看pod 的重启策略；

- Always ：但凡 pod 对象终止就重启，此为默认策略；
- OnFialure：仅在pod 对象出现错误时才重启；
#### pod 的创建过程
##### 情况一、使用 kubectl run 命令创建的 pod
```java
注意：
kubectl run 在旧版本中创建的是deployment，
但在新的版本中创建的是pod则其创建过程不涉及deployment
```
如果是单独的创建一个 pod ，创建过程是这样的：

1. 首先，用户通过 kubectl 或其他 api 客户端工具提交需要创建的 pod 信息给 apiserver；
2. apiserver 验证客户端的用户权限信息，验证通过开始处理创建请求生成 pod 对象信息，并将信息存入etcd ，然后返回确认信息给客户端；
3. apiserver 开始反馈 etcd 中 pod 对象的变化，其他组件使用 watch 机制跟踪 apiserver 上的变动；
4. scheduler 发现有新的 pod 对象要创建，开始调用内部算法机制为 pod 分配最佳的主机，并将结果信息更新至apiserver；
5. node 节点上的 kubelet 通过 watch 机制 跟踪 apiserver 发现有 pod 调度到本节点，尝试调用docker 启动容器，并将结果反馈 apiserver
6. apiserver 将收到的pod 状态存入 etcd 中。
##### 情况二、使用 deployment 来创建 pod；

1. 首先、用户使用 kubectl create 命令或者 kubectl apply 命令提交了要创建一个 deployment 资源请求；
2. api-server收到创建资源的请求后，会对客户端操作进行身份认证，在客户端的~/.kube 文件夹下，已经设置好了相关的用户认证信息，api-server 鉴权，请求合法就会接受本次操作，并把相关的信息保存到etcd中，然后返回确认信息给客户端。
3. apiserver 开发反馈 etcd 中过程创建的对象的变化，其他组件使用 watch 机制跟踪 apiserver 上的变动。
4. controller-manager 组件会监听 api-server 的信息，controller-manager是有多个类型的，比如Depolyment controller，它的作用就是负责监听Deployment，此时 Deployment Controller 发现有新的 deployment 要创建，那么它就会去创建一个 Replica Set ，一个 ReplicaSet 的产生，又被另一个叫做 ReplicaSet Controller 监听到了，紧接着它就会去分析 ReplicaSet 的语义，它了解到时要依照 ReplicaSet 的template 去创建 pod，这个pod并不存在 ，那么久创建此 pod，当pod 刚被创建时，它的nodeName 属性值为空，代表此 pod 未被调度。
5. 调度器 scheduler 组件开始介入工作， Scheduler 也是通过 watch 机制跟踪 apiserver 上的变动，发现有未调度的pod，则根绝内部算法，节点资源情况，pod 定义的亲和性 反亲和性 等等，调度器会综合的选出一批候选节点，在候选节点中选择一个最优的节点，然后将pod 绑定该节点，将信息反馈给api-server。
6. kubelet 组件部署与 Node 之上，它也是通过watch 机制跟踪apiserver 上的变动，监听到有一个pod要被调度到自身所在的node 上来，kubelet 首先判断本地是否在此 pod ，如果不存在，则会进入创建 pod 流程，创建pod 有分为几种情况，第一种是容器不需要挂载外部存储，则相当于直接 docker run 把容器启动，但不会直接挂载docler 网络，而是通过 CNI 调用网络插件配置容器网络，如果需要挂载外部存储，则还要调用 CSI 来挂载存储。kubelet 创建完 pod，将信息反馈给 api-server ，api-server 将pod 信息写入etcd。
7. pod 建立成功后，ReolicaSet Controller 会对其持续进行关注，如果Pod 因意外或被手动退出，RaplicaSet Controller 会知道，并创建新的pod ，以保持 replicas 数量期望值。
#### k8s 创建一个pod 的详细流程，涉及的组件怎么通信？

1. 客户创建提交创建请求，可以通过 api-server 提供的 restful 接口，或者是通过 kubectl 命令行工具，支持的数据类型包括 JSON 和 Yaml；
2. api-server 处理用户请求，将pod 信息存储在 etcd 中；
3. kube-scheduler 通过 api-server 提供的接口监控到未绑定的 pod ，尝试未 pod 分配 node节点，主要分为两个阶段，预选阶段和优选阶段，其中预选阶段是遍历所有的 node 节点，根据策略筛选出候选节点，而优选阶段是在第一步的基础上，未每一个候选节点进行打分，分数最高者胜出；
4. 选择分数最高的节点，进行pod binding 操作，并将结果 存储到 etcd 中；
5. 随后目标节点的 kubelet 进程通过 api-server 提供的接口监测到 kube-scheduler 产生的 pod 绑定事件，然后从etcd 获取pod 清淡，下载镜像并启动容器；
#### 简单描述一下pod 的终止过程
1、用户向apiserver发送删除pod对象的命令；
2、apiserver中的pod对象信息会随着时间的推移而更新，在宽限期内（默认30s），pod被视为dead；
3、将pod标记为terminating状态；
4、kubectl在监控到pod对象为terminating状态了就会启动pod关闭过程；
5、endpoint控制器监控到pod对象的关闭行为时将其从所有匹配到此endpoint的server资源endpoint列表中删除；
6、如果当前pod对象定义了preStop钩子处理器，则在其被标记为terminating后会意同步的方式启动执行；
7、pod对象中的容器进程收到停止信息；
8、宽限期结束后，若pod中还存在运行的进程，那么pod对象会收到立即终止的信息；
9、kubelet请求apiserver将此pod资源的宽限期设置为0从而完成删除操作，此时pod对用户已不可见。
#### pod的生命周期有哪几种？
pod 生命周期有 5种状态（也称5种相位），如下；

- pending（挂起）：API server 已经创建 pod ，但是该 pod 还有一个 或多个容器的镜像没有创建，包括正在下载镜像的过程；
- Running（运行中）：pod内所有的容器已经创建，且至少有一个容器处于运行状态，正在启动（正字啊重启状态）
- Succeed（成功）：pod 内所有容器均已退出，且不会再重启
- Failled（失败）：pod 内所有容器均已退出，且至少有一个容器为退出失败状态
- unknown（未知）：由于某种原因 api-server 无法获取 pod的状态，可能是由于网络通信问题导致的。
#### pod 一直处于 pending 状态一般有哪些情况，怎么排查？
一个 pod 一开始创建的时候，它本身会处于pending 状态，这时可能正在拉取镜像，正在创建容器的过程。
如果等了一会发现 pod 一直处于 pending 状态，可以使用kubectl describe 命令查看一下 pod 的 events 详细信息。一般可能会有这么几种情况导致 pod 一直处于 pending 状态；

1. 调度器调度失败。

Scheduer 调度器无法为pod 分配一个合适的node 节点。
而这又会有很多种情况，比如，node 节点处于 cpu、内存压力，导致无节点可调度；pod 定义了资源请求，没有node 节点满足资源请求；node 节点上有污点而 pod 没有定义容忍；pod 中定义了亲和性和反亲和性，而没有节点满足这些条件；

2. pvc 、pv 无法动态创建

如果因为 pvc 或 pv 无法动态创建，那么 pod 也会一直处于 pending 状态，比如要使用 StatefulSet 创建 redis 集群，定义的 storageClassName 名称写错了，那么会造成无法创建 pvc，这种情况 pod 也会一直处于 pending 状态，或者，即使 pvc 是正常创建了，但是由于 某些异常原因导致动态供应存储无法正常创建 pv ，那么这种情况 pod 也会一直处于 pending 状态。
#### DaenonSet 资源对象的特性？
DaemonSet 这种资源对象会在每个 K8S 集群中的节点上运行，并且每个节点只能运行一个 pod，这是它和 deployment 资源对象最大也是唯一的区别。所以在其 yaml 文件中，不支持定义 replicas，除此之外，与 Deployment，RS等资源对象的写法相同。
DaemonSet 一般使用的场景有

- 去做每个节点的日志收集工作；
- 监控每个节点的运行状态；
#### 删除一个 pod 会发生什么事情？
kube-api-server 会接收到用户的删除指令，默认有30秒时间等待优雅退出，超过30秒会被标记为死亡状态，此时pod的状态 Terminating，kubelet 看到 pod 标记为Terminating 就开始了关闭 pod 的工作；；
关闭流程如下：

1. pod 从service 的endpoint 列表中被移除；
2. 如果该 pod 定义了一个停止前的钩子，其会在 pod 内部被调用，停止钩子一般定义了如何优雅的结束进程；
3. 进程被发送 TERM 信号（kill -14）
4. 当超过优雅退出的时间后，pod 中的所有进程都会被发送 SIGKILL 信号 （kill -9）
#### pod 的共享资源？

1. pid 命名空间：pod 中不同应用程序可以看到其他应用程序的进程ID;
2. 网络命名空间：pod 中的多个容器能够访问同一个 ip 和端口范围；
3. ipc 命名空间：pod 中的多个容器能够使用 SystemV IPC 或 POSIX 消息队列进行通信；
4. UTS 命名空间：pod 中的多个容器共享一个主机名；
5. Volumes（共享存储卷）：pod 的各个容器可以访问在 pod 级别定义的 Volunmes；
#### pod 的初始化容器是干什么的？
init container，初始化容器用于在启动应用容器之前完成应用容器所需要的前置条件，
初始化容器本质上和应用容器是一样的，但是初始化容器是仅允许一次就结束的任务，初始化容器具有两大特征：
1、初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成；
2、初始化容器必须按照定义的顺序执行，当且仅当前一个初始化容器成功之后，后面的一个初始化容器才能运行；
#### pod 的资源请求、限制如何定义？
pod 的资源请求、资源限制可以直接在pod 中定义
主要包括两块内容，

- limits，限制 pod 能使用的最大 cpu 和内存。
- requests，pod 启动时申请的cpu 和内存。
```java
 resources:					#资源配额
      limits:					#限制最大资源，上限
        cpu: 2					#CPU限制，单位是code数
        memory: 2G				#内存最大限制
      requests:					#请求资源（最小，下限）
        cpu: 1					#CPU请求，单位是code数
        memory: 500G			#内存最小请求

```

#### pod 的定义中 有个command 和args 参数，这两个参数不会和docker镜像的 entrypointc 冲突吗
不会，在pod 中定义的 command 参数 是用于指定容器的启动命令列表，如果不指定，则默认使用DockerFile 打包时的启动命令，args 参数用于容器的启动命令需要的参数列表；
特别说明：
kubernetes中的command、args其实是实现覆盖dockerfile中的ENTRYPOINT的功能的。
```java
1、如果command和args均没有写，那么使用Dockerfile的配置；
2、如果command写了但args没写，那么Dockerfile默认的配置会被忽略，执行指定的command；
3、如果command没写但args写了，那么Dockerfile中的ENTRYPOINT的会被执行，使用当前args的参数；
4、如果command和args都写了，那么Dockerfile会被忽略，执行输入的command和args。

```

#### pause 容器作用
每个 pod 里运行着一个特殊的容器，被称为 pause的容器，也称 根容器，而其他容器都是业务容器；
创建pause 容器主要是为了 业务容器提供 Linux 命名空间，共享基础：包括 pid 、icp、net 等，以及启动 int 进程，并收割僵尸进程。
这些业务容器共享 pause 容器 的网络命名空间和 volume 挂载卷，当 pod 被创建时，pod 首先会创建 pause 容器，从而把其他业务容器加入 pause 容器，从而让所有业务容器都在同一个命名空间中，这样可以实现网络共享。
pod 还可以共享存储，在 pod 级别引入数据库 volume，业务容器都可以挂载到这个数据卷从而实现持久化存储。
#### 标签及标签选择器是什么？如何使用？
标签是键值对类型，标签可以附加到任何资源对象上，主要用于管理对象，查询和筛选。
标签常被用于标签选择器的匹配度检查，从而完成资源筛选；一个资源可以定义一个或者多个标签在上面。
标签选择器，标签要与标签选择器结合在一起，标签选择器允许我们选择标记有特定标签的资源对象子集，如pod，并对这些特定标签的pod进行查询，删除等操作。
标签和标签选择器最重要的使用之一在于，在deployment 中，在pod 模版中定义 pod 的标签，然后在 deployment 定义标签选择器，这样就通过标签选择器来选择那些pod是受其控制的，service也是通过标签选择器来关联哪些pod 最后其服务后端 pod。
#### service 是如何与 pod 关联的？
通过标签选择器，每一个由deployment 创建的pod 都带有标签，这样，service 就可以定义标签选择器来关联哪些pod 是作为其后端了，service 就与 pod 关联在一起了。
#### service 的域名解析格式、pod 的域名解析格式
service 的DNS 域名表示格式为 <servicenames>.<namespace>.svc.<clusterdomain>,
servicename 是 service 的名称，namespace 是service 所处的命名空间，clusterdomain 是 k8s 集群设置的域名后缀，一般默认为 cluster.lucal
对于 deployment 、daemonsets 等创建的 pod ，还可以通过 <pod-ip>.<deployment-name>.<namespace>.svc.<clusterdomain> 这样的域名访问。


#### service 的类型有几种

- clusterIP：表示 service 仅供集群内部使用，默认值就是 ClusterIP 类型
- NodePort：表示 service 可以对外访问应用，会在每个节点上暴露一个端口，这样外部浏览器访问地址为：任意节点的ip：NodeProt 就能连上 service 了
- LoadBalancer：表示service 对外访问应用，这种类型的 service 是共有云环境下的service ，此模式需要外部云厂商的支持，需要一个公网IP地址。
- ExternalName：这种类型的service 会把集群外部的服务引入集群内部，这样集群内直接访问service 就可以间接的使用集群外部服务了。

一般情况下，service 都是 clusterIP 类型的，通过 ingress 接入外部流量
#### pod 到service 的通信？

1. k8s 在创建服务时为服务分配一个虚拟IP，客户端通过该ip访问服务，服务则负责讲请求转发到后端pod 上；
2. service 是通过 kube-proxy 服务进程实现的，该进程在每个 Node 上均运行可以看作一个透明代理兼负载均衡器；
3. 对每个 TCP 类型 service，kube-proxy 都会在本地 Node 上建立一个 socketServer 来负责接收请求，然后均匀发送到后端pod，默认采用 Round Robin 负责均衡算法。
4. Service 的 Cluster IP 与 nodePort 等概念是 kube-proxy 通过 IPtables 的NAT转换实现，kube-proxy进程动态创建于 service 相关的 iptables 规则；
5. kube-proxy 通过查询和监听 API service 中的service 与Endpoints 的变化实现其主要功能，包括为新创建的 Service 打开一个本地代理对象，接受请求针对发生变化的 Service 列表，kube-proxy 会逐个处理。
#### 一个应用pod 是如何发现service 的，或者说，pod 里面的容器用于如何连接 service 的？
两种方式，一种是通过环境变量，另一种是通过 service 的dns 域名方式

- 环境变量：

当pod 被创建之后，k8s系统会自动为容器注入集群内有效的service 名称和端口号等信息为环境变量的形式，这样容器应用直接通过取环境变量值就能访问 service 了。
如curl http://${WEBAPP_SERVICE_HOST}:{WEBAPP_SERVICE_PORT}

- DNS方式：

使用dns 域名解析的前提是 k8s 集群内有 DNS 域名解析服务器，
默认k8s中有一个 CoreDNS 作为k8s集群的默认 DNS 服务器提供域名解析服务器；
service 的DNS 域名表示格式为<servicename>.<namespace>.svc.<clusterdomain>，servicename 是service 的名称，namespace 是service 所处的命名空间，clusterdomain 是 k8s 集群设置的域名后缀，一般默认为 cluster.local，这样容器应用直接通过 service 域名就能访问 service 了。 
如wget http://svc-deployment-nginx.default.svc.cluster.local:80，
另外，service的port端口如果定义了名称，那么port也可以通过DNS进行解析，格式为：_<portname>._<protocol>.<servicename>.<namespace>.svc.<clusterdomain>
#### 如何创建一个 service 代理外部的服务，或者换句话来说，在 k8s 集群内的应用如何访问外部的服务，如数据库服务，缓存服务等？
可以通过一个没有标签选择器的 service 来代理集群外部的服务。

1. 创建 service 时不指定 selector 标签选择器，但需要指定 service 的 port 端口、端口的 name、端口协议等，这样创建出来的 service 因为没有指定标签选择器就不会自动创建 endpoint；
2. 手动创建一个鱼 service 同名的 endpoint，endpoint 中定义外部服务的IP和端口，endpoint 的名称需要和 service 的 名称一样，端口协议也要一样，端口的name 也要与 service 的端口 的name 一样，不然endpoint 不能与service 进行关联。

完成以上两步，k8s 会自动将 service 和同名的 endpoint 进行关联，这样 k8s 集群内的应用程序直接访问这个service 相当于访问外部的服务了。
#### service 、 endpoint、kube-proxys 三者关系

- service

在 kubernetes 中，service 是一种为一组功能相同的 pod 提供单一不变的接入点的资源。当service 被建议时，service 的IP 和端口 不会改变，这样外部的客户端（也可以是集群内部的客户端）通过 service 的 IP 和端口来建立连接，这些链接会被路由到提供该服务的任意一个 pod 上。基于这种方式，客户端不需要知道每个单独提供服务的 pod 地址，这样 pod 就可以在集群中随时被创建或销毁。

- endpoint：

service 维护一个叫 endpoint 的资源列表，endpoint 资源对象保存着 service 关联的pod 的ip 和端口。从表面上看，当pod 消失，service 会在 endpoint 列表中剔除 pod ，当有新的 pod 加入，service 就会将pod ip 加入 endpoint 列表；但是正在底层的逻辑是，endpoint 的这种自动剔除、添加、更新pod 的地址其实底层是有 endpoint controller 控制的，endpoint controller 负责监听 service 和对应的 pod 副本的变化，如果监听到 service 被剔除，则删除和该 service 同名的 endpoint 对象，如果监听到新的 service 被创建或者修改，则根该service 信息获取相关的 pod 列表，然后创建或更新 service 对应的endpoint 对象，如果监听到 pod 事件，则更新它所对应的 servcie 的 endpoint 对象。

- kube-proxys

kube-proxy 运行在 node 节点上，在node 节点上实现 pod 网络dialing，维护网络规划和四层负载均衡工作，kube-proxy 会监听 api-server 中从而获取service 和endpoint 的变化情况，创建并维护路由规则以提供服务IP 和负载均衡功能。
简单理解次进程是 service 的透明代理兼负载均衡器、其核心功能是将某个 service 的访问请求转发到后端的多个 pod 实例上。
#### 无头service 和普通 service 区别，无头service 使用场景是什么？
无头 service 没有cluster ip ，在定义 service 时讲 service.spec.clusterIP:None，表示创建的是无头service。普通service 是用于为一组后端pod 提供请求连接的负载均衡，让客户端能通过固定service ip 地址来访问pod，这类的pod 没有状态的，同时service 还具有负载均衡和服务发现的功能。普通service 跟我们平时使用的nginx 反向代理很相似。
6个redis pod ，它们相互之间要通信并要组成一个redis集群，不需要所谓的service 负载均衡，这是无头service 派上用场了，无头service 由于没有 cluster ip，kube-proxy 就不会处理它不会对它生成规则负载均衡，无头service 直接绑定的是 pod 的ip 。无头 service 仍有标签选择器，有标签选择器就会有 endpoint 资源。
使用场景： 用于有状态的应用场景，如 kafka 集群、redis 集群等，这类 pod 之间需要相互通信相互组成集群，不需要所谓的service 负载均衡。
#### deployment 怎么扩容或缩容？
直接修改 pod 副本数即可，可以通过以下的方式来修改 pod 的副本数；

1. 直接修改 yaml 文件的replicas 字段数值，然后 kubectl apply -f xx.yml 来实现更新；
2. 使用 kubectl edit deployment xxx 修改 replicas 来实现在线更新；
3. 使用 Kubectl scale --replicas=5 deployment/deployment-nginx 命令来扩容缩容。
#### deployment 的更新升级策略有哪些？
deployment的升级策略主要有两种。

1. Recreate 重建更新：这种更新策略会杀掉正在运行的pod，然后再重新创建 pod；
2. rollingUpdate 滚动更新：这种更新策略，deployment 会以滚动更新的方式来逐个更新 pod ，同时通过设置滚动更新的两个参数 maxUnavailable、maxSurge 来控制更新的过程。

#### deployment 的滚动更新策略有两个特别主要的参数，解释含义？
deployment 的滚动更新策略，rollingUpdate 策略，主要有两个参数，maxUnavailable、maxSurge。

- maxUnavailable：最大可用数，maxUnavailable 用于指定 deployment 在更新的过程中不可用状态的 pod 的最大数量。maxUnavailbale 的值可以是一个整数值，也可以是pod期望副本的百分比，如25%，计算时向下去整。
- maxSurge：最大激增数，maxSurge 指定 deployment 在更新的过程中pod的总数量最大能超过 pod 副本数多少个，maxUnavailable 的值可以是一个整数值，也可以是pod期望副本的百分比，如25%，计算向上取整。

#### deployment 更新的命令有哪些？
可以通过三种方式来实现更新 deployment 。

1. 直接修改 yaml 文件的镜像版本，然后 kubectl apply -f xx.yaml 来实现更新；
2. 使用 kubectl edit deployment xxx 实现在线更新；
3. 使用 kubectl set image deployment/nginx busybox=busybox nginx=nginx:1.9.1 命令来更新。 
#### deployment 的更新过程？
deployment 是通过控制 replicase 来实现，由reolicase 真正创建 pod 副本，每更新一次 deployment ，都会创建新的 replicaset。
#### 有哪些存储卷，作用分别是什么？
| 卷 | 作用 | 常见场景 |
| --- | --- | --- |
| emptyDir | 用于存储临时数据的简单空目录 | 一个pod中的多个容器需要共享彼此的数据，emptyDir 的数据随着容器的消亡也会销毁 |
| hostPath | 用于将目录从工作节点的文件系统挂载到pod 中， | 不常用，缺点是，pod的调度是不固定的，也就是当pod消失后 deployment 重新创建一个 pod，而这pod 如果不是被调度到之前的pod 的节点，那么该pod 就不能访问之前的数据 |
| configMap | 用于将非敏感的数据保存到键值对中，使用时可以作为环境变量、命令行参数 arg ，存储卷被 pods 挂载使用 | 将应用程序的不敏感配置文件创建为 configmap卷，可实现热更新 |
| secret  | 主要用于存储和管理一些敏感数据，然后通过在 pod 的容器里挂载 Volume 的方式或者环境变量的方式访问到这些 Secret 里 保存的信息了，pod 会自动解析 Secret 的信息 | 将应用程序的账号密码登敏感信息通过 secret 卷的形式挂载到 pod 中使用 |
| downwardApi | 主要用于暴露pod 的元数据，如pod 的名字 | pod 中的应用程序需要指定 pod 的name 等元数据，就可以通过 downwardApi 卷的形式挂载给 pod 使用 |
| projected  | 这是一种特殊的卷 ，用于将上面这些卷一次性的挂载给 pod 使用 | 将上面这些卷一次性挂载给pod 使用 |
| pvc | pvc 是存储卷声明 | 通常会创建pvc 表示对内存的申请，然后再 pod 中使用 pvc |
| 网络存储卷 | pod 挂载网络哦存储卷，这样就能够将数据持久化到后端的存储里 | 常见的网络存储卷有 nfs 存储，glusterfs 卷，ceph rbd 存储卷 |

#### pv 的访问模式有哪几种？
pv 的访问模式有3种，如下：

- ReadWriteOnce ， 简写 RWO 表示，只仅允许单个节点以读写方式挂载；
- ReadOnlyMany ，简写 ROX 表示，可以被许多节点以 只读方式挂载；
- ReadWriteMany，简写 RWX 表示， 可以被多个节点以读写方式挂载；

#### pv 的回收策略 有哪几种？
主要有3种回收策略，retain 保留，delete 删除，Recycle 回收。

- Retain： 保留，该策略允许手动回收资源，当删除 pvc 时，pvc 仍然存在，pv 被视为已释放，管理员可以手动回收卷。
- Delete：删除，如果 Volume 插件支持，删除 pvc 时会同时删除 pv，动态卷默认 Delete ，目前支持 delete 存储的后端包括 AWS EBS，GCE PD , Azure Disk ， OpenStack Cinder 等。
- Recyle：回收，如果Volume 插件支持，Recycle 策略会对卷执行 rm -rf 清理该 pv，可使其用于下一下新的 PVC ，但是本地策略将会被弃用，目前只有NFS 和HostPath 支持该策略。（弃用了）

#### pv 的生命周期中，一般有几种状态
pv 一共有 4种状态，分别是：
创建 pv 后，pv的状态有 以下 4 种 Available（可用），Bound（绑定），Released（已释放），Failed（失败）
```java
Available，表示pv已经创建正常，处于可用状态；
Bound，表示pv已经被某个pvc绑定，注意，一个pv一旦被某个pvc绑定，那么该pvc就独占该pv，其他pvc不能再与该pv绑定；
Released，表示pvc被删除了，pv状态就会变成已释放；
Failed，表示pv的自动回收失败；
```

#### pv 存储空间不足怎么扩容？
一般的，会使用动态分配资源，在创建 storageclass 时指定参数 allowVolumeExpansion：true ，表示允许用户通过修改 pvc 申请的存储空间自动完成pv 的扩容。
当增大pvc 的存储空间时，不会重新创建一个 pv ，而是扩容其绑定的后端 pv。这样就能完成扩容了，但是allowVolumeExpansion 这个特性只支持 扩容空间 不支持减少空间
#### 存储类的资源回收策略：
主要有两种 回收策略，delete 删除，默认就是delete 策略 ，retain 保留。
Retain：保留，该策略允许手动回收资源，当删除PVC时，PV仍然存在，PV被视为已释放，管理员可以手动回收卷。
Delete：删除，如果Volume插件支持，删除PVC时会同时删除PV，动态卷默认为Delete，目前支持Delete的存储后端包括AWS EBS，GCE PD，Azure Disk，OpenStack Cinder等。
注意：使用存储类动态创建的pv默认继承存储类的回收策略，当然当pv创建后你也可以手动修改pv的回收策略。


#### 怎么使一个node脱离集群调度，比如要停机维护单又不能影响业务应用
使用kubectl drain 命令
#### k8s 生产中遇到问题，解决思路
前端的lb 负载均衡器上的 keepalived 出现过脑裂情况。
#### 
