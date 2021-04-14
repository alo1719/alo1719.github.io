---
title: 断路器与Resilience4j
date: 2020-03-01 14:55:40
tags: [java, circuit breaker, resilience4j]
---

断路器 (Circuit Breaker) 一词来源于电气工程，在软件领域则是一种设计模式。

它的作用类似于家庭里的空气开关，当某一个电器发生故障时，所在房间的空气开关会自动断路，防止故障扩散到其他房间，引发更大的事故。

它的一个典型应用场景是：一个服务要发送 100 个请求，其中 99 个请求都正常，但是其中 1 个请求的接口发生了故障，要等待 5 秒。那么这个服务要完成加载就需要 5 秒以上的时间。如果发生故障的请求不在核心链路上，那么引入断路器就能以某种策略，不再请求那个出故障的接口，让服务正常加载。

如果出故障的接口在核心链路上，可以为这个请求加上降级策略：如果调用该接口出现了故障，那么断路器可以将请求指向一个备用接口，来实现高可用性。

## 断路器的状态

断路器有三种状态：Closed、Open 和 Half-Open。

![h_2732904-992404e807e83288.jpg](https://i.loli.net/2020/03/01/oDvSaYkwhUGVHMO.jpg)

Closed 状态是断路器连通的状态，是默认状态；

当失败次数达到某个阈值，就会进入 Open 状态，这个时候断路器会拦截所有调用并直接抛出 Exception；

Half-Open 状态是“尝试”状态，在 Open 状态下会不定时尝试接口是否正常，若仍有故障退回 Open 状态，故障消除了的话就回到 Closed 状态。
<!-- more -->

## Hystrix 和 Resilience4j

Java 语言生态中，常见的断路器有两个：Hystrix 和 Resilience4j。Hystrix 由 Netfilx 开发，目前已经停止开发，所以本文将以 Resilience4j 为主。

[Resilience4j](https://github.com/resilience4j/resilience4j) 是一个开源项目，在 Github 进行托管，其特点是全部代码都遵从函数式编程范式。

## 引入 Resilience4j

如果使用 Spring Boot 的话，只需要使用：

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
    <version>1.2.0</version>
</dependency>
```

## 使用 Resilience4j

Resilience4j 默认使用装饰者模式，但在 Spring Boot 项目中肯定是用 `@CircuitBreaker` 的 AOP 注解更方便、侵入性也更小。

详细的使用文档可以看 [这里](https://resilience4j.readme.io/docs)。

## Resilience4j 组件

Retry：重试  
Circuit Breaker：断路器  
Rate Limiter：限制比率  
Time Limiter：限制时间  
Bulkhead：隔板，控制等待区（环形缓冲区）容量  
Cache：缓存  
Fallback：后备

## Resilience4j 配置文件

```yaml
resilience4j.circuitbreaker:
    backends:
        default:
        # The size of ring buffer when the CircuitBreaker is closed
        ringBufferSizeInClosedState: 3
        ringBufferSizeInHalfOpenState: 3
        # The wait duration in millis which defines how long the CircuitBreaker should stay open before it switches to half open
        waitInterval: 1000
        # The failure rate threshold above which the CircuitBreaker opens and starts short-curcuiting calls
        failureRateThreshold: 20
```

## 演示内容

1. 断路器拦截请求的功能
2. 断路器自动恢复到 Half_Open 状态的功能
3. Half_Open 的回归逻辑
4. 断路器拦截慢请求的功能
5. 断路器后备方法的使用（fallback 仍视作一次失败，要绕过的话可以使用 transitionToClosedState）