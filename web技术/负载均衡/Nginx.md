# Nginx

nginx的平滑重启
nginx接收到hub信号后，首先重新加载配置文件，然后启动的新的worker进程，并向老的worker进程发送信号，告诉他们处理完当前请求后就退出，不再接受新的请求，新的worker启动后，就开始接受新的请求。

所有的worker进程在listenfd 会在新连接到来时变得可读。为了保证只有一个进程处理该连接，所有worker在注册listenfd之前抢accept_mutex，抢到的那个注册listenfd读事件

nginx采用异步非堵塞的方式处理请求，也就是说nginx可以同时处理上千万个请求
epoll是请求间的切换（循环处理多个准备好的事件），不需要上下文的切换，不需要创建线程，更多的并发数只是会占用更多的内存而已。

推荐设置的worker数量为cpu的核数，更多的worker数，只会导致进程来竞争cpu资源，带来不必要的上下文切换

所有定时事件时存放在红黑树中。

ulimit -n 每个进程最大的打开文件数，nginx的最大连接数为  每个进程最大能打开的文件描述符*进程数

nginx支持的event处理类型有
AIO(异步IO）
epoll(Linux特有）
eventport(solaris 10 特有）
kqueue(BSD 特有）
poll
select等

event模块的主要功能就是 监听accept后建立的连接，对读写事件进行添加删除。 事件处理模型和nginx的 非阻塞IO模型结合在一起使用。 当IO可读可写时，相应的读写事件就会被唤醒。 此时就会去处理事件的回调函数

nginx是多进程程序，80端口是各进程共享的。多进程同时listen 80端口，势必会产生竞争，也产生了所谓的惊群效应。 当内核accept一个连接时，会唤醒所有等待中的进程，但实际只有一个进程能获取连接，其他的进程都是无效唤醒。  nginx多进程的锁在底层默认是通过cpu自旋锁来实现，如果操作系统不支持自旋锁，就采用文件锁。

谁拿到锁，谁才将accept的fd加入到epoll队列，其他的子进程拿不到锁，就不会把fd加入到epoll中。接下来也不会有所有子进程的epol被唤醒返回

epoll是poll的升级版，它不会遍历所有的fd，只会遍历活跃的，加入到ready状态的fd
epoll 的io效率不会随fd数目增加而线性降低

epoll是非阻塞IO，但是是同步IO， 是多路复用IO
去掉了遍历文件描述符，而通过监听回调的方式
主要是当有大量的idle-connection时，效率会更高



1、nginx是一款高性能都http与反向代理服务器软件
2、nginx是异步非阻塞，支持并发好，6w+的并发，占内存少，但扩展少，性能高，对静态文件，索引文件以及自动索引效率高，模块化。支持好，nginx对fastcgi支持好。采用epoll与queue模型
3、apache是阻塞，并发支持不好，占内存多，支持通用的语言接口，但是主流，模块化。如果动态文件较多，rewrite频繁，则采用apache。
4、nginx分为核心模块、基础模块与第三方模块。
5、一个nginx进程占用20MB内存，一个fastcgi占用 15MB内存
6、同等硬件条件下， nginx性能比apache高5、6倍
7、能够根据域名、url不同，将HTTP请求分发到不同的后端服务器群组
8、kill -HUP ‘/user/local/webserver/nginx/logs/nginx.pd 平滑重启nginx
9、修改nginx后，可以使用 -t -c 配置文件。测试nginx配置文件是否正常。
10、Kill -TERM  ‘/user/local/webserver/nginx/logs/nginx.pid’ 快速结束nginx
11、kill -QUIT ‘/user/local/webserver/nginx/logs/nginx.pid’ 从容结束nginx
12、nginx中有1个master进程与多个worker进程
13、Kill -USER2 ‘nginx主进程pid
14、Kill -USER1 重新打开日志文件，在切割日志时作用较大
14、平滑升级nginx的过程中，新旧两个版本的nginx会同时工作，处理请求
15、kill -winch 旧版本nginx主进程pid
16、虚拟主机提供了在同一台nginx服务器，同一组nginx进程上运行多个网站的功能
17、虚拟主机分三种1、基于ip的虚拟主机2、基于域名的虚拟主机3、基于端口的虚拟主机
18、本地网卡设备 eth0与本地回环设备 lo。 本地回环设备的ip是127.0.0.1. 作用有两个，1、ping 本地回环设备，用来测试本机的ip协议与网卡设备都正常。2、用于作为当本地没有运行其他server时，绑定本地回环设备
19、修改  /etc/rc.local。 增加虚拟ip
20、DNS负载均衡成本很低，但是一旦某个服务器挂了，将出现部门用户无法访问。即使修改了DNS
，仍会又由于缓存的原因，不能及时生效。并且不能区分服务器性能与服务器运行状态的差距。
21负载均衡、常见的时 四/七层负载均衡交换机。 
软件四层常用的负载均衡是  lvs。 linux virtual server  采用ip负载均衡与基于内容简单分发技术
软件七层常用的负载均衡是 nginx与HA proxy。nginx负载均衡可以很方便的支持反向代理，包括基于ip,url与hash等对后端服务器进行负载均衡，并能够对后端服务器进行健康检查
22、根据ip_hash,保证每个ip能够hash到同一个服务器上，保存了session信息。 不会提示用户未登录。
23、当需要从负载均衡中摘除一段时间时，标记  down
24、pcre （perl compatible regular expression)
25、一些使用MVC框架的程序，只有一个入口，可以通过rewrite实现。 当目录结构发生变化，通过rewrite实现。一些动态的url要伪装成静态的url，也需要通过url实现。
26、~表示区分大小写的匹配， ~*表示不区分大小写的匹配
27、~!表示区分大小写的不匹配， ~*！表示不区分大小写的匹配
28、-f 表示是否文件存在， -d表示是否路径存在，-e表示是否文件与路径存在。 -x表示是否能够执行
29、302 redict 临时重定向，301 perminant 永久重定向。 
30、last表示完成rewrite，再次对server进行匹配，break表示完成rewrite，不再匹配
31、加上？能够去除url中的参数
32、nginx配置四大模块。  main、server、upstream、location
33、nginx模块开发，对于业务简单，但访问量大的服务，
nginx配置
1、root表示文件资源的根目录
2、index表示主页默认页面
3、一段server就是一个虚拟主机，通过server_name区分不同的虚拟主机
4、默认的日志格式，是combined格式。
5、Access_log设置log的访问日志存放的位置，缓存等
6、log_format设置日志的格式等，默认combined格式。
7、open_log_file_cache 日志文件描述符缓存，每次nginx写日志时，都默认先打开再关闭文件描述符，可以设置打开一段时间后再关闭
8、定时切割日志，还是需要借助crontab，创建文件夹后mv 日志 日志日期
9、gzip压缩页面
10、在location中， 设置expires 为1d 、3h等设置过期时间，起到缓存的作用
11、LNMP linux、nginx、mysql、php、python、perl
12、vary: Accept-Encoding告诉代理服务器，缓存两种版本的资源，压缩和非压缩，这有助于避免一些公共代理不能正确检测Content-encoding表头的问题。



HTTP模块、EVENT模块和MAIL模块属于核心模块
HTTP Access模块、HTTP FastCGI模块、HTTP Proxy模块和HTTPRewrite模块属于基本模块
HTTP Upstream Reuest Hash模块、Notice模块 Access Key模块属于第三方模块
5、从功能上分、  处理模块、过滤模块、代理模块
Proxies模块，就是HTTP Upstream之类的模块，这些模块主要是和后端一些服务比如fastcgi等操作交互。实现服务代理与负载均衡
6、nginx配置 main（全局设置）、server（主机设置）、upstream（负载均衡服务器设置）、location（URL匹配特定位置的设置）。 main部分设置将影响其他所有设置。Server部分主要用于指定主机与端口。 upstream指令主要用于负载均衡。location部分用于匹配网页位置..
7、ssl（secure socket layer，安全套接层）模块，保障internet上数据传输安全。https是已安全为目标的HTTP通道，简单讲是HTTP的安全版。即HTTP下加入SSL层，https的安全基础是SSL
8、nginx负载均衡支持4种调度算法1、轮询2、ip hash 同一ip映射到同一台服务器3、weight，权重，权重越大，访问几率越高4、fai，按照页面加载时间长短与页面大小。
9、1）标准的代理缓冲服务器，缓存静态页面，浏览器直接从本地代理服务器那里请求数据而不是向原web站点请求数据2、透明代理服务器，与1相同，但不需要指定代理服务器ip和端口 例如iptables3、反向代理服务器，降低原始web服务器的负载。位于本地web服务器和internet之间，处理所有对web服务器的请求，组织了web服务器与internet的直接通信。
10、FastCGI是一个可伸缩地、高速地在http server和动态脚本语言之间通信接口。发展从cgi中来
11、传统的cgi性能差，每次遇到动态程序都需要重启解析器进行解析。
12、当HTTP服务器遇到动态程序时，可以将其直接交付给FastCGI处理，然后将结果返回浏览器。是http服务器专一的处理静态请求或者将动态脚本服务器的结果返回给客户端。
13、spawn-fcgi与php-fpm是支持php的两个FASTCGI进程管理器。
php-fpm是php的一个补丁。
14、url rewrite，即地址重写。用户得到的全部是处理后的url地址
15、nginx本身不会对php进行解析，终端对php页面的请求，被nginx交给fastcgi进程监听的ip与端口，有php-fpm作为动态解析服务器处理，最后将处理结果交给nginx，其实nginx就是一个反向代理服务器。nginx通过方向代理功能将动态请求转向后端php-fpm，从而实现对php的解析支持，这就是nginx实现php动态解析的原理。
nginx不支持对外部程序的直接调用或解析，所有外部程序必须通过FastCGi接口。这个接口在linux下是socket（这个socket可以是文件socket，也可以是ip socket
因为是多进程，所以比cgi多线程消耗更多的服务器内存。
PHP-FPM、Spawn-FCGI都是守护php-cgi的进程管理器
Tomcat是一个开放源码的，运行servlet和JSP应用软件的基于Java和web应用软件的容器。 Tomcat Server是根据servlet和JSP规范执行的。 因此也可以说Tomcat server实行了Apache规范。  缺点是对静态文件、高并发处理较弱。
