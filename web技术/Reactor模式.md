# Reactor 模式

Reactor模式又叫Dispatcher模式， 即I/O多路复用监听事件，收到事件后，根据事件类型分配给某个进程/线程

单Reactor单进程/线程：redis的方案
单Reactor多进程/线程：基本都是单Reactor/多线程。一个Reactor对象承担了所有事件的监听和响应，容易成为性能瓶颈
多Reactor多进程/线程：netty 和memcache的方案是多Reactor多线程， nginx是多Reactor多进程

Proactor模式
proactor是异步I/O操作