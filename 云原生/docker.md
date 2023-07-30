# docker

NameSpacce视图隔离, cgroups 资源隔离。 容器镜像本质就是容器的根文件系统rootfs

Namespace
* ipc
* Network
* UTS  : 主机与域名
* Mount
* pid

UnionFS
* 将多个disk挂在在同一个目录下
* 将一个readonly branch 和 writeonly branch 联合在一起

Bootfs 启动的fs， rootfs就是正常的linux目录

3、容器只是一个特殊的进程而已，容器是单进程的意思不是只能运行一个进程，而是只有一个进程是可控的
4、敏捷和高性能是容器相较于虚拟机最大的优势

对 Docker 项目来说，它最核心的原理实际上就是为待创建的用户进程：
1、启用 Linux Namespace 配置
2、设置指定的 Cgroups 参数
3、切换进程的根目录（Change Root）

docker通常用于如下场景：
1、web应用的自动化打包和发布
2、自动化测试与持续集成、发布
3、在服务型环境中部署和调整数据库或其他的后台应用
4、从头编译或者搭建自己的paas 环境

Docker服务端和客户端
docker服务端是一个服务进程，管理着所有容器docker Daemon运行于主机上，处理服务请求
docker客户端则扮演着服务端远程控制器，可以控制docker的服务端进程

docker可以理解为在沙盒中运行的进程，这个沙盒包含了该进程运行所必须的资源，包括文件系统，系统类库和shell环境等

当对docker修改后，运行docker commit将变更进行保存

docker run 镜像名、命令
docker只要求宿主机操作系统在内核3.10以上，系统未64位

Build once, configure once and run anywhere

Docker Container 负责应用程序的运行，包括操作系统、用户添加的文件以及元数据
Docker images是一个只读模板，用来运行docker容器
Docker File 是文件指令集，用来说明如何自动创建Docker镜像

Docker使用以下操作系统的功能来提高容器技术效率
1、Namespaces 确保一个容器中只运行一个进程而且不能看到或影响容器外的其他进程
2、Control Groups 是LXC的重要组成部分，具有资源核算与限制的关键功能
3、Union FS(文件系统） 作为容器的构建块，为了支持Docker的轻量级以及速度快的特性
运行任何应用程序，都需要1、构建一个镜像  2、运行容器

每个镜像都源于一个基本的镜像，然后根据Dockerfile中的指令创建模板，对于每个指令，在镜像上创建一个新的层面，一旦镜像创建完成，就可以将它们推送到中央的 registry   Docker Index
当容器被启动后，一个读写层会被添加到镜像的顶层，当分配到合适的网络和IP地址后，就可以运行

DockerFile 语法

CMD 提供容器默认的执行指令，只允许使用一次CMD指令
EXPOSE 指定容器在运行时监听的端口
ENTRYPOINT 配置给容器一个可执行的命令，使用镜像创建容器时一个特定的应用程序可以被设置为默认程序

ADD 复制文件指令
WORKDIR 指定 RUN  CMD 与ENTRYPOINT 命令的工作目录
ENV 设置环境便令。 使用键值对
VOLUME  授权访问从容器内到主机的目录
docker会通过iptables创建一条nat，把宿主机打到映射端口数据包通过转发到docker0的网关，docker0再通过广播找到对应ip的目标容器

docker容器连接
1、映射网络端口 NAT
2、容器连接，linking会创建一种父子级别的关系，父container可以看到子container提供的信息

overlay网络是最主流的容器跨节点传输和路由方案

Docker 通过 copy on write 机制实现容器的新增和修改， 通过白障机制实现删除 

