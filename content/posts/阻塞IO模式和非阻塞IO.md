---
title: "阻塞IO模式和非阻塞IO"
date: 2025-08-12T13:10:09+08:00
draft: fasle
tags: [基础]
---



众所周知java一共有三种IO

1. BIO Java BIO[Blocking I/O] | 同步阻塞I/O模式
2. NIO Java NIO[New I/O] | 同步非阻塞模式
3. AIO Java AIO[Asynchronous I/O] | 异步非阻塞I/O模型

那什么是阻塞什么是非阻塞呢？

## 阻塞IO

io操作是应用程序向操作系统内核（Kernel）请求数据（读）或提交数据（写）的过程。由于磁盘、网络等硬件设备的速度远慢于CPU，很多时候性能瓶颈都于此有关。

当进程开始处理io操作的时候，如果操作中某项资源没有获取，该调用会一直等待，直到数据准备好并将数据从内核空间复制到用户空间后，调用才会返回。在此期间，应用程序的这个线程会被挂起（阻塞），无法执行任何其他操作。

以recvfrom调用为例：

1. 调用recvfrom。
2. 内核没有接受到网络传输的数据，开始等待。
3. 内核将该进程/线程置于休眠状态，不消耗CPU。
4. 接收到网络传输的数据，从内核复制数据到指定的内存地址。
5. 进程被唤醒。

> 在阻塞IO下，一个线程在任何时刻只能处理一个I/O请求。如果这个I/O请求非常慢（如网络延迟高），整个线程都会被拖累，无法处理其他客户端，导致系统吞吐量极低。

## 非阻塞IO

当用户进程调用一个I/O操作时，如果数据还没有准备好，内核会立即返回一个错误码（如EWOULDBLOCK或EAGAIN），而不会让应用程序等待。应用程序可以继续执行其他任务，但需要不断地主动询问内核数据是否准备好了。

还是以recvfrom为例：

1. 调用设置为非阻塞模式的 recvfrom
2. 内核检查数据。
3. 返回检查结果。
   + **未就绪**，返回错误码。
   + **就绪**，将数据复制到内存。
4. 应用进程在收到错误码后，不会被阻塞，可以继续做其他事。但为了获取数据，它必须在稍后再次调用 recvfrom（即轮询），直到调用成功返回为止。

一直轮询的情况下会使CPU处于“忙等待”（busy-waiting）状态，浪费大量CPU周期。这在连接数很多但活跃连接数很少的场景下尤其低效。

## IO多路复用

对于现代应用的io，更多是使用I/O多路复用，Linux下的 `select, poll, epoll`.

阻塞I/O性能差，而非阻塞I/O又过于消耗CPU，作为优化多路复用采用：

1. 有一个线程同时监控多个IO描述符
2. 监控线程本身是阻塞的任何一个I/O就绪时被唤醒
3. 一旦被唤醒，应用程序就可以使用非阻塞I/O的方式去处理那些已经确定“就绪”的数据，因为此时调用I/O函数会立即成功，不会等待。

## 如何配置spring boot的io模型

```java
@Configuration
 public class TomcatConfig {
    //自定义SpringBoot嵌入式Tomcat
    @Bean
    public TomcatServletWebServerFactory servletContainer() {
        TomcatServletWebServerFactory tomcat = new 
TomcatServletWebServerFactory() {};
        
        tomcat.addAdditionalTomcatConnectors(http11Nio2Connector());
        return tomcat;
    }
   
    //配置连接器nio2
    public Connector http11Nio2Connector() {
        Connector connector=new 
Connector("org.apache.coyote.http11.Http11Nio2Protocol");
        Http11Nio2Protocol nio2Protocol = (Http11Nio2Protocol) 
connector.getProtocolHandler();
        //等待队列最多允许1000个线程在队列中等待
        nio2Protocol.setAcceptCount(1000);
        // 设置最大线程数
        nio2Protocol.setMaxThreads(1000);
        // 设置最大连接数
        nio2Protocol.setMaxConnections(20000);
        //定制化keepalivetimeout,设置30秒内没有请求则服务端自动断开keepalive链接
        nio2Protocol.setKeepAliveTimeout(30000);
        //当客户端发送超过10000个请求则自动断开keepalive链接
        nio2Protocol.setMaxKeepAliveRequests(10000);
        // 请求方式
        connector.setScheme("http");
        connector.setPort(9003);                    //自定义的端口，与源端口9001可以共用，也是是说通过9001或者9003都可以访问
        connector.setRedirectPort(8443);
        return connector;
    }
 }
```



