---
title: "Tomcat调优"
date: 2025-08-10T23:09:58+08:00
draft: true
tags: []
---

## spring boot 内置tomcat配置

> 如何确定你的配置生效了？
> 在application.yml中配置以下选项。
>
> ``` yaml
> # 暴露所有的监控点
> management.endpoints.web.exposure.include: '*'
>  # 定义Actuator访问路径
> management.endpoints.web.base-path: /actuator
>  # 开启endpoint 关闭服务功能
> management.endpoint.shutdown.enabled: true
> ```
>
> 这三条命令做了什么？
>
> ``` yaml
> management.endpoints.web.exposure.include: '*'
> ```
>
> 将所有可用的 Actuator Web 端点都通过 HTTP 协议暴露出来，使其可以被访问。默认情况下，出于安全考虑，Spring Boot 只会暴露 /health 和 /info 这两个端点。使用 * 会将所有端点，如 /beans, /env, /metrics, /heapdump, /threaddump, /shutdown 等全部暴露出来。
>
> ``` yaml
> management.endpoints.web.base-path: /actuator
> ```
>
> 定义所有 Actuator Web 端点的统一访问路径前缀。所有 Actuator 的端点都必须在 /actuator 这个路径下进行访问。
>
> + 如果没有这行配置，健康检查的 URL 将是 http://your-app/health
> + 配置了这行之后，URL 会变成 http://your-app/actuator/health。
>
> ``` yaml
> management.endpoint.shutdown.enabled: true
> ```
>
> 允许通过访问 /shutdown 端点来关闭 Spring Boot 应用程序。

Actuator 是 Spring Boot 生态中的一个子项目，它通过提供一系列称为 “端点 (Endpoints)” 的 HTTP 或 JMX 接口，将应用程序的监控和管理功能暴露出来。它是构建生产就绪 (Production-Ready) 应用的核心组件。

每个端点都是一个专注于特定功能的访问点。当在项目中引入 Actuator 依赖后，它会自动配置好一系列端点。

| 端点 (`/actuator/...`) | 功能描述                                                     | 默认是否暴露 (Web)   |
| :--------------------- | :----------------------------------------------------------- | :------------------- |
| **`health`**           | **健康检查**：显示应用健康状况。它会自动检查数据库、磁盘空间等，可以自定义。**这是最重要的端点**。 | **是**               |
| **`info`**             | **应用信息**：显示自定义的应用信息，如版本号、作者、Git 提交信息等。 | **是**               |
| **`metrics`**          | **核心指标**：提供详细的运行时指标，如 JVM 内存使用、CPU 使用率、HTTP 请求统计（响应时间、QPS）等。 | 否                   |
| **`prometheus`**       | **Prometheus 指标**：以 Prometheus 监控系统支持的格式暴露 `metrics` 数据，便于集成。 | 否                   |
| **`env`**              | **环境变量**：显示所有可用的环境变量、JVM 属性、配置文件 (`application.properties`) 中的配置项。**非常敏感！** | 否                   |
| **`loggers`**          | **日志级别管理**：显示并允许在运行时修改指定 Logger 的日志级别（如从 INFO 改为 DEBUG）。 | 否                   |
| **`beans`**            | **Bean 列表**：显示 Spring IoC 容器中所有 Bean 的信息。      | 否                   |
| **`heapdump`**         | **堆内存快照**：生成并下载一个 JVM 堆内存快照文件 (`.hprof`)，用于内存泄漏分析。 | 否                   |
| **`threaddump`**       | **线程快照**：生成并下载一个 JVM 线程快照，用于分析线程死锁、性能问题。 | 否                   |
| **`shutdown`**         | **关闭应用**：通过 POST 请求可以正常关闭应用程序。**极其危险！** | 否（且默认禁用功能） |

### 三大配置，最大线程数 最大连接等待数 最大连接数

1. maxThreads：最大线程数
   1.  当请求到达web服务器时tomcat创建一个线程处理请求。
   2. 最大线程数不是越大越好，因为创建线程也是有性能开销的，线程的上下文切换也有一定的性能开销。
   3. 对于低RT的系统来说不需要增加线程数
   4. 经验值：
      + 1C2G，线程数200 
      + 4C8G，线程数800
2. accept-count：最大等待连接数
   1. 当调用HTTP请求数达到Tomcat的最大线程数时，还有新的请求进来，这时Tomcat会将该剩余请 求放到等待队列中。
   2. 这个值就是等待队列的最大值。
3. Max Connections：最大连接数
   1. 最大连接数是指在同一时间内，Tomcat能够接受的最大连接数。如果设置为-1，则表示不限制
   2. 在bio和nio模式下的默认值不同
   3. 当连接数达到最大值Max Connections后系统会继续接收连接，但不会超过acceptCount限制
