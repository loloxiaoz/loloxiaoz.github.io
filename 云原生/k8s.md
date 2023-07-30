# k8s

kubernets 不仅仅是一个编排系统，它消除了编排。 提供了一个以容器为中心的基础架构，提供应用层的功能，负载均衡/日志/监控/检查检测
自动重启/自动复制/自动部署/自动伸缩
Pod 是一个或多个容器组合起来的共享资源，例如存储/网络/容器运行的信息, pod中的容器共享ip和端口
node是工作节点, 上面运行着kubelet和container runtime
Kubelet，管理这k8s master和node之间的通信，管理机器上运行的pods和containers容器

service是一个抽象概念，定义了pod的逻辑分组和一种访问的策略
service分为4中
1. clusterIP， 在集群内部IP上暴露服务，此类型使service只能从集群访问
2. nodePort， 通过每个Node上的ip和静态端口(node port) 暴露服务
3. loadbalance，负责均衡，可以向外暴露服务
4. externalName，通过返回CNAME和它的值，可以将服务映射到externalName字段

service的作用:
通过service， 可以方便的实现服务发现和负载均衡
kubectl使用滚动更新，持续集成和持续交付的0停机

minikube使用中轻量级的k8s集群，通过在本地计算机上创建虚拟机并只包含单个节点的简单集群

一切都是资源
Deployment 一次部署行为
Service 一个服务
Pod下可以有多个service

对 Docker 项目来说，它最核心的原理实际上就是为待创建的用户进程：
1、启用 Linux Namespace 配置
2、设置指定的 Cgroups 参数
3、切换进程的根目录（Change Root）。

Union File System联合文件系统，将多个不同位置的目录联合挂载到同一个目录下，增量rootfs层
只读层 + init层 + 可读写层， whiteout 覆盖只读层的文件
init层只对当前的docker生效，在提交时智慧提交可读写层， 不包含init层
强一致性，容器镜像将会成为未来软件的主流发布方式

容器使用的内核和宿主机使用的内核是一致的。 如果应用依赖内核版本，就无法跨平台。 Windows系统会给容器外面套一个vm，所以也能运行linux容器

Docker on Mac 以及windows Docker（Hyper -V实现） 实际上是基于虚拟化技术实现的。 而Linux容器则是真正的容器
ENTRYPOINT 提供一个隐含的ENTRYPOINT， 即： /bin/sh -c

Volume机制，允许将宿主机指定的目录或者文件，挂载到容器中进行读取和修改操作
NameSpacce视图隔离, cgroups 资源隔离。 容器镜像本质就是容器的根文件系统rootfs

容器就从一个开发者手里的小工具，一跃成为云计算领域的绝对主角，能够定义容器组织和管理规范的容器编排技术，则当仁不让的坐上了容器化技术的头把交椅

kubernetes项目要解决的问题
1、

kubernetes的架构
* master： API、Schedule、Manager
* node：CRI（Container Runtime interface）， Device Plugin， CNI（Container Networking Interface）和CSI（Container Storage Interface）

一旦要追求普适性，那就一定要从顶层开始做好设计

应用之间需要非常频繁的交互和访问，或者通过本地文件进行信息交互，这些容器被划分为一个pod，pod里的容器共享一个Network Namespace， 同一组数据卷，从而达到高效率交换信息的目的

Service服务的主要作用，就是作为pod的代理入口，从而代替Pod对外暴露一个固定的网络地址
编排对象/服务对象，声明式API
调度：把一个容器按照某种规则，放置到某个最佳节点上运行起来
编排：按照用户的意愿以及整个系统的规则，完全自动化地处理好容器之间的各种关系

创建一个master节点， kubeadm
将一个node节点加入到当前集群中 kubeadm join

Kubernetes 默认 kube-proxy 和DNS这两个插件提供整个集群的服务发现和DNS功能
kubeadm目前最欠缺的是，一键部署一个高可用的kubernetes集群， 即Etcd， Master组件都应该是多节点集群， 而不是现在这样的单点

默认情况下master节点是不允许运行用户的pod， 但可以依靠kubernetesd 的Taint/Toleration机制 实现可以调度

Kubectl apply -f https:raw.github

很多时候所谓的云原生，就是kubernetes 原生的意思。 而像Rook、Istio这样的项目，正是贯彻了这个思路的典范， 在我们后面讲解声明式API，相信你对这些项目的设计思想有更深刻的体会，声明式带来的最大好处，正式自动化

deployment管理pod的的方法，叫做控制器模式。
Labels就是一组key-value格式的标签

Port 表示内部pod直接通信的port
NodePort 表示对外暴露的port
TargetPort表示最终的端口，也是pod的端口

ClusterIP 只对集群内部可见
NodePort 可对外部可见

PVC: persistent volume claim，特殊的volume，需要与实现进一步绑定
PV: persistent vlume
接口与实现的关系

StatefulSet实现对应用存储状态的管理
* StatefulSet直接管理的是Pod
* k8s通过headless service， 为这些有编号的pod，在DNS服务器中生成带有同样编号的DNS记录
* StatefulSet还为每一个pod分配并创建一个同样编号的PVC

storage class自动地为集群里存在的每一个pvc，调用存储插件创建对应的PV
StatefulSet 是一种特殊的Deployment， 只不过记录了很多状态

声明式API，兼顾性能、API完备性、版本化、向后兼容等很多工程化指标
CRD(Custom Resource Definition) 自定义API资源

Qos优先级
* Guaranteed Pod， 必须同时指定内存和CPU的限制
* Bursting Pod，至少一个容器具有内存或CPU要求
* Best Effort, 必须没有设置内存和CPU限制或请求
资源回收策略
* 可压缩资源：CPU属于可压缩资源，当Pod使用超过其设置的limits值时，
* 不可压缩资源：内存是不可压缩资源，当资源不足时先kill低优先级的pod

k8s卷的内容
* Empty Dir: 声明周期与pod相同，pod删除时，empty Dir也会删除，主要用于某些应用程序无需多久保存的临时目录，多个容器的共享目录等， 是Pod分配到Node上时被自动创建的
* Host Path：为pod挂载宿主机上的目录或文件，使得容器可以使用宿主机的文件系统进行存储，缺点是动态调度在不同的node上，下次调度到其他节点时，无法使用之前节点上的存储文件。（支持如果文件或目录不存在，就创建一个的功能）

livenessProbe 判断pod中的应用容器是否健康，可以理解成健康检查
readnessProbe 判断pod是否已经就绪， 可以接收请求和访问。

在pod中的容器之间共享进程命名空间，从而使改pod中的所有其他容器都是可见的

Device Plugin设计支持GPU的管理，但由于扩展性不好，处于可用但不好用的场景
Pod中reloadable打开后，可以更新pod中某个container的镜像更新

核心组件
* etcd保存了整个集群的状态
* apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制
* controller manager 负责维护集群的状态，比如故障检测，自动扩展，滚动更新等
* scheduler 负责资源的调度，按照预定的调度策略将pod调度到相应的机器上
* kubelet负责维护容器生命的声明周期，也负责Volume（CVI）和网络CNI的管理
* Container runtime 负责镜像管理以及pod和容器的真正运行（CRI）
* Kube-proxy负责为service提供cluster内部的服务发现和负责均衡
* Federation 提供跨可用区的集群


Pod是一组紧密关联的容器集合， Pod的IP会随着Pod的重启而变化，共享网络地址和文件系统
每个API对象都有3大类属性： 元数据metadata、规范spec和状态status
DaemonSet是后台支撑服务集， 存储，日志和监控在每个节点上支撑k8s集群运行的服务
StatefulSet适合数据库服务，集群化管理服务等有状态服务。 

节点污点
System pod 可以被调度到主节点上，因为toleration: noschedule  匹配污点 taint:noschedule
三种污点
* NoSchedule
* PreferNoSchedule
* NoExecute: 在节点上运行着的pod，将会从这个节点去除

节点容忍度
可以自定义污点和容忍度 key:value  effect

节点亲缘性
将pod只调度到某几个节点上

TUN设备是一种工作在三层（Network Layer）的虚拟网络设备给。 TUN设备的功能非常简单，即在操作系统内核和用户应用程序之间传递IP包

UDP: 使用flannel进程在机器之间进行转发，性能较低
VXLAN: 虚拟可扩展局域网，VTEP: VXLAN Tunnel End Poin(虚拟隧道端点）， VTEP的封装和解封装对象， 是二层的数据帧

RBAC
* Role: 角色， 它其实是一组规则，定义了一组对Kubernetes API对象的操作权限
* Subject： 被作用者， 既可以是”人“，也可以是”机器“， 也可以是你再kubernetes里定义的用户
* RoleBinding： 定义了被作用者和角色的绑定关系

NameSpace是Kubernetes项目里的一个逻辑管理单位，并没有提供实际的隔离或者多租户能力
Kubenetes中负责管理的”内置用户“，正是我们前面提到过的 ServiceAccount

Ingress 流入
egress 流出


已经实现network policy的网络查件包括 calico、weave 和 kube-router
通过控制循环的方式对network policy对象的增删改查做出响应，然后在宿主机上完成iptables规则的配置工作

netfilter子系统的作用，就是在Linux内核里挡在网卡 和 用户态进程之间的一道 ”防火墙“
network policy其实只是宿主机上的一系列iptables的规则
k8s在基础上提供了一种“弱多租户” soft multi-tenancy

只有处于running状态， 且 readness Probe检查通过的pod， 才会出现service 的endpoints 列表中

基于iptables的service实现，实际是一组随机模式的iptables链
基于ipvs的service实现， ipvs并不需要在宿主机为每个pod设置iptables规则


高可用
1、kubeapi-server、etcd是多实例且同时运行的
2、controller manager和kube-scheduler是同时运行，但只有一个实例起作用，其他的实例处于待命状态
3、唯一能和etcd通信的，是kubeapi-server组件，其他所有组件都是通过kubeapi-server通信
4、api-server实现了乐观锁机制，保证集群的状态是一致的
5、kubectl 通过一个http post请求到api-server上

分工
* API Server只保证更新数据到etcd中，以及通知监听的客户端有变更的动作
* 调度器只是给pod分配节点
* 控制器执行循环，负责将实际状态调整为期望的状态，控制器之间也不会相互通信

Namespace
* ipc
* Network
* UTS  : 主机与域名
* Mount
* pid
Cgroup ， 以层级的方式进行管理
cfs_cpu_quatos
cfs_cpu_period

UnionFS
* 将多个disk挂在在同一个目录下
* 将一个readonly branch 和 writeonly branch 联合在一起

Bootfs 启动的fs， rootfs就是正常的linux目录

docker history 看docker镜像的分层

Overlay  fs

api-server 准入，授权，认证，只有api-server可以直接访问etcd，并且watch了etcd的全部变化
api-server也是支持watch的  kubectl -w就是订阅了watch
control manager 是watch变化，然后不断重试，然后达到最终一致性

schedule 的顺序
1、predict  预测
2、priority  优先级
2、bind 绑定

kubelet， 是由systemd 托管的， kubelet是k8s 的init system。 可以将k8s的核心组件全部拉起， 通过指定目录的方式 
* 从不同资源获取pod清单，并按照需求启停pod的核心组件
* 上报node的健康及相关信息
* 上报pod的健康及相关信息

kubeproxy, 监控service的配置，并完成负载均衡的配置

k8s 也是自启动的

pod结构
1、annotation 与 label 都是string map
* annotation：不支持过滤查询， 主要是注解，用于pod结构的定义扩展
* Label：是支持过滤查询的
2、finalizer 类似于锁
3、resourceVersion 多线程或多进程同时修改，主要用于乐观锁， mvcc
4、selfLink



云计算核心
* 集群管理
* 作业调度
* 服务管理
prefilter
Predicate
priority
Pod affinity pod的反亲和性 ，一般的应用都会设置
Taint类型  
* noschedule
* noexecute
* prefer no schedule

prometheus 与 influxdb区别

TSDB   纵向写，横向读。  一次写一个时间点的全部数据，读一个时间区间的数据


prometheus 使用了mmap技术，大量数据放内存




pv 和pvc的绑定
* pv的存储大小必须满足pvc的要求
* pv和pvc的storage class name字段必须一样

Kubernetes 就是一个生产级别的容器编排平台和集群管理系统，应用的开发、部署、运维等一系列工作都要向 Kubernetes 看齐，使用容器、微服务、声明式 API 等技术，保证应用的整个生命周期都能够在 Kubernetes 环境里顺利实施，不需要附加额外的条件
组件实现了 Kubernetes 的核心功能特性，没有这些组件 Kubernetes 就无法启动，而插件则是 Kubernetes 的一些附加功能，属于“锦上添花”，不安装也不会影响 Kubernetes 的正常运行。

yaml是json的炒集

Kubectl api-resources
Kubectl get pods —v=9
Kubectl explain
Kubectl run —dry-run=client
Kubectl run -o yaml

单一型接口，复合型接口

service是基于kube-proxy之上的四层负载均衡，在TCP-IP协议栈上进行路由转发
Ingress是基于Http/Https之上的七层负载均衡，是一堆Http路由规则的集合
* ingress
* ingress controller：交给社区实现，最著名的是nginx
* ingress class：解耦ingress 和 ingress controller
ingress controller的pod，由于网络隔离的原因，还需要对外通过service暴露网络

启动顺序、稳定域名以及存储模板，就是stateful的三个关键能力