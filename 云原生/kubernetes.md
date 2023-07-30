# kubernetes

> kubernets 生产级别的容器编排平台和集群管理系统，应用的开发、部署、运维等一系列工作。提供了一个以容器为中心的基础架构，提供应用层的功能，负载均衡/日志/监控/检查检测/自动重启/自动复制/自动部署/自动伸缩

云原生就是kubernetes原生的意思

## 设计思想

一旦要追求普适性，那就一定要从顶层开始做好设计

声明式API: 一切都是资源, 自动化, 兼顾性能、API完备性、版本化、向后兼容等很多工程化指标
控制器模式:
自定义CRD: CRD(Custom Resource Definition), 自定义API资源
插件机制: 
调度: 把一个容器按照某种规则，放置到某个最佳节点上运行起来
编排: 按照用户的意愿以及整个系统的规则，完全自动化地处理好容器之间的各种关系

## 架构

- master： API、Schedule、Manager, 默认情况下master节点是不允许运行用户的pod， 但可以依靠kubernetesd 的Taint/Toleration机制 实现可以调度
- node：kubelet, CRI（Container Runtime interface）， Device Plugin， CNI（Container Networking Interface）和CSI（Container Storage Interface）

核心组件

- etcd保存了整个集群的状态
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制
- controller manager 负责维护集群的状态，比如故障检测，自动扩展，滚动更新等
- scheduler 负责资源的调度，按照预定的调度策略将pod调度到相应的机器上
- kubelet负责维护容器生命的声明周期，也负责Volume（CVI）和网络CNI的管理
- container runtime 负责镜像管理以及pod和容器的真正运行（CRI）
- kube-proxy负责为service提供cluster内部的服务发现和负责均衡
- federation 提供跨可用区的集群

## 基本组件

### kubelet

kubelet管理master和node之间的通信，管理机器上运行的pods和containers容器, 由systemd托管的， 是k8s 的init system。 可以通过指定目录的方式将k8s的核心组件全部拉起

- 从不同资源获取pod清单，并按照需求启停pod的核心组件
- 上报node的健康及相关信息
- 上报pod的健康及相关信息

### api-server

- API Server只保证更新数据到etcd中，以及通知监听的客户端有变更的动作
- 控制器执行循环，负责将实际状态调整为期望的状态，控制器之间也不会相互通信

api-server 准入，授权，认证，只有api-server可以直接访问etcd，并且watch了etcd的全部变化
api-server也是支持watch的  kubectl -w就是订阅了watch
control manager 是watch变化，然后不断重试，然后达到最终一致性

### scheduler

调度器只是给pod分配节点, schedule 的顺序

- prefilter 过滤
- predict 预测
- priority  优先级
- bind 绑定

- 节点污点: 可以自定义污点和容忍度 key:value  effect
  - NoSchedule
  - NoExecute: 在节点上运行着的pod，将会从这个节点去除
  - PreferNoSchedule
- 节点容忍度: Pod Tolerate
- 节点亲缘性: Pod Affinity,  将pod只调度到某几个节点上
- 节点反亲和性: Pod Anti Affinity

### kube-proxy

监控service的配置，并完成负载均衡的配置

### pod

Pod是一组紧密关联的容器集合， Pod的IP会随着Pod的重启而变化，共享网络地址和文件系统
每个API对象都有3大类属性： 元数据metadata、规范spec和状态status

Pod 是一个或多个容器组合起来的共享资源，例如存储/网络/容器运行的信息, pod中的容器共享ip和端口
在pod中的容器之间共享进程命名空间，从而使改pod中的所有其他容器都是可见的
Qos优先级  

- Guaranteed Pod， 必须同时指定内存和CPU的限制
- Bursting Pod，至少一个容器具有内存或CPU要求
- Best Effort, 必须没有设置内存和CPU限制或请求
资源回收策略
- 可压缩资源：CPU属于可压缩资源，当Pod使用超过其设置的limits值时，
- 不可压缩资源：内存是不可压缩资源，当资源不足时先kill低优先级的pod

pod结构

- annotation 与 label 都是string map
  - annotation：不支持过滤查询， 主要是注解，用于pod结构的定义扩展
  - label：是支持过滤查询的
- finalizer 类似于锁
- resourceVersion 多线程或多进程同时修改，主要用于乐观锁， mvcc
- selfLink

livenessProbe 判断pod中的应用容器是否健康，可以理解成健康检查
readnessProbe 判断pod是否已经就绪， 可以接收请求和访问。

Pod中reloadable打开后，可以更新pod中某个container的镜像更新
只有处于running状态， 且 readness Probe检查通过的pod， 才会出现service 的endpoints 列表中

### service

service服务的主要作用, 就是作为pod的代理入口, 从而代替Pod对外暴露一个固定的网络地址, 实现服务发现和负载均衡
service是一个抽象概念, 定义了pod的逻辑分组和一种访问的策略
service分为4中

- clusterIP， 在集群内部IP上暴露服务，此类型使service只能从集群访问
- nodePort， 通过每个Node上的ip和静态端口(node port) 暴露服务
- loadbalance，负责均衡，可以向外暴露服务
- externalName，通过返回CNAME和它的值，可以将服务映射到externalName字段

service是基于kube-proxy之上的四层负载均衡，在TCP-IP协议栈上进行路由转发

### deployment

deployment管理pod的的方法，叫做控制器模式。

### stateful

StatefulSet是一种特殊的Deployment，实现对应用存储状态的管理, 启动顺序、稳定域名以及存储模板，就是stateful的三个关键能力

- statefulSet直接管理的是Pod
- k8s通过headless service， 为这些有编号的pod，在DNS服务器中生成带有同样编号的DNS记录
- StatefulSet还为每一个pod分配并创建一个同样编号的PVC

statefulSet适合数据库服务，集群化管理服务等有状态服务。

### daemonset

daemonSet是后台支撑服务集， 存储，日志和监控在每个节点上支撑k8s集群运行的服务

### ingress

ingress是基于Http/Https之上的七层负载均衡，是一堆Http路由规则的集合

- ingress controller：交给社区实现，最著名的是nginx
- ingress class：解耦ingress 和 ingress controller
ingress controller的pod，由于网络隔离的原因，还需要对外通过service暴露网络

### 存储

Union File System联合文件系统，将多个不同位置的目录联合挂载到同一个目录下，增量rootfs层
只读层 + init层 + 可读写层， whiteout 覆盖只读层的文件
init层只对当前的docker生效，在提交时智慧提交可读写层， 不包含init层

k8s卷的内容

- Empty Dir: 声明周期与pod相同，pod删除时，empty Dir也会删除，主要用于某些应用程序无需多久保存的临时目录，多个容器的共享目录等， 是Pod分配到Node上时被自动创建的
- Host Path：为pod挂载宿主机上的目录或文件，使得容器可以使用宿主机的文件系统进行存储，缺点是动态调度在不同的node上，下次调度到其他节点时，无法使用之前节点上的存储文件。（支持如果文件或目录不存在，就创建一个的功能）

PVC: persistent volume claim，特殊的volume，需要与实现进一步绑定
PV: persistent vlume

storage class自动地为集群里存在的每一个pvc，调用存储插件创建对应的PV

pv 和pvc的绑定

- pv的存储大小必须满足pvc的要求
- pv和pvc的storage class name字段必须一样

Volume机制，允许将宿主机指定的目录或者文件，挂载到容器中进行读取和修改操作

### 网络插件

netfilter子系统的作用，就是在Linux内核里挡在网卡 和 用户态进程之间的一道 ”防火墙“
network policy其实只是宿主机上的一系列iptables的规则, 已经实现network policy的网络查件包括 calico、weave 和 kube-router
通过控制循环的方式对network policy对象的增删改查做出响应，然后在宿主机上完成iptables规则的配置工作
基于iptables的service实现，实际是一组随机模式的iptables链
基于ipvs的service实现， ipvs并不需要在宿主机为每个pod设置iptables规则

TUN设备是一种工作在三层（Network Layer）的虚拟网络设备给。 TUN设备的功能非常简单，即在操作系统内核和用户应用程序之间传递IP包
UDP: 使用flannel进程在机器之间进行转发，性能较低
VXLAN: 虚拟可扩展局域网，VTEP: VXLAN Tunnel End Poin(虚拟隧道端点）， VTEP的封装和解封装对象， 是二层的数据帧

### namespace

NameSpace是Kubernetes项目里的一个逻辑管理单位，并没有提供实际的隔离或者多租户能力

## 跨平台

容器使用的内核和宿主机使用的内核是一致的。 如果应用依赖内核版本，就无法跨平台。 Windows系统会给容器外面套一个vm，所以也能运行linux容器
Docker on Mac 以及windows Docker（Hyper -V实现） 实际上是基于虚拟化技术实现的。 而Linux容器则是真正的容器

## 高可用

1、kubeapi-server、etcd是多实例且同时运行的
2、controller manager和kube-scheduler是同时运行，但只有一个实例起作用，其他的实例处于待命状态
3、唯一能和etcd通信的，是kubeapi-server组件，其他所有组件都是通过kubeapi-server通信
4、api-server实现了乐观锁机制，保证集群的状态是一致的
5、kubectl 通过一个http post请求到api-server上