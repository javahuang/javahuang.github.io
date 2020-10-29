# netty

Netty is a NIO client server framework which enables quick and easy development of network applications such as protocol servers and clients. It greatly simplifies and streamlines network programming such as TCP and UDP socket server.

1. `NioEventLoopGroup`  is a multithreaded event loop that handles I/O operation. 创建两个，boss 用来接收并分发请求，worker 用来处理请求
2. `ServerBootstrap` 用来配置 server
3. `NioServerSocketChannel` 用来实例化一个 Channel
4. `ChannelInitializer` 用来配置一个新的 Channel

In a stream-based transport such as TCP/IP, received data is stored into a socket receive buffer. Unfortunately, the buffer of a stream-based transport is not a queue of packets but a queue of bytes. It means, even if you sent two messages as two independent packets, an operating system will not treat them as two messages but as just a bunch of bytes. Therefore, there is no guarantee that what you read is exactly what your remote peer wrote. For example, let us assume that the TCP/IP stack of an operating system has received three packets:

netty 里面的 package 是啥

相对于Tomcat这种Web Server（顾名思义主要是提供Web协议相关的服务的），Netty是一个Network Server，是处于Web Server更下层的网络框架，也就是说你可以使用Netty模仿Tomcat做一个提供HTTP服务的Web容器。

简而言之，Netty通过使用NIO的很多新特性，对TCP/UDP编程进行了简化和封装，提供了更容易使用的网络编程接口，让你可以根据自己的需要封装独特的HTTP Server活着FTP Server等.

## Netty3 源码解析

Netty 的事件驱动思想，事件触发，相应的 Handler 会被调用，所有事件来自 `ChannelEvent` 接口。

内部的连接处理、协议编解码、超时等机制都是通过 handler 完成的

```txt
org
└── jboss
└── netty
├── bootstrap 配置并启动服务的类
├── buffer 缓冲相关类，对NIO Buffer做了一些封装
├── channel 核心部分，处理连接
├── container 连接其他容器的代码
├── example 使用示例
├── handler 基于handler的扩展部分，实现协议编解码等附加功能
├── logging 日志
└── util 工具类
```

### buffer

```text
      +-------------------+------------------+------------------+
      | discardable bytes |  readable bytes  |  writable bytes  |
      |                   |     (CONTENT)    |                  |
      +-------------------+------------------+------------------+
      |                   |                  |                  |
      0      <=      readerIndex   <=   writerIndex    <=    capacity
```

- readerIndex 每次都将从这个索引位置读取
- writerIndex 每次都会从这个索引位置写入
- capacity 当前 buffer 的容量

### Channel

ChannelPipeline

### Netty 和 Reactor 模式

Reactor模式的答案是：由一个不断等待和循环的单独进程（线程）来做这件事，它接受所有handler的注册，并负责先操作系统查询IO是否就绪，在就绪后就调用指定handler进行处理，这个角色的名字就叫做Reactor。



## 参考

[netty 源码解读](http://ifeve.com/netty1/#comments)

[netty](https://netty.io/)