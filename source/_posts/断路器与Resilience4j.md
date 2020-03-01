---
title: 断路器与Resilience4j
date: 2020-03-01 14:55:40
tags: [java, circuit breaker, resilience4j]
---

断路器的作用就相当于家庭里的空气开关, 当某一个电器发生故障时, 所在房间的空气开关会自动断路, 防止故障扩散到其他房间, 引发更大的事故. 断路器 (Circuit Breaker) 本身来自于电气工程, 是一种设计模式.

断路器的一个典型应用场景是: 一个服务要发送100个请求, 其中99个请求都没有问题, 但是其中1个请求的接口出了故障, 要等待10秒左右的时间. 那么这个服务要完成加载就需要10秒左右的时间. 但如果那个发生故障的请求不是非常必要的, 那么引入断路器就可以 (在一定阈值后) 直接"封掉"这个请求, 让网页正常加载, 而不用再等待10秒左右的时间.

更成熟的办法是为这个请求加上fallback方法: 如果调用这个接口出现了故障, 那么断路器可以指向一个备用接口, 来实现高可用性.

## 断路器的状态

断路器的状态可以分为三种: Closed, Open 和 Half-Open.

![h_2732904-992404e807e83288.jpg](https://i.loli.net/2020/03/01/oDvSaYkwhUGVHMO.jpg)

Closed的状态就是断路器连通的状态, 是默认状态;

当失败次数达到一定阈值, 就会进入Open状态, 这个时候断路器会拦截所有调用并直接返回Exception;

Half-Open状态是"尝试"状态, 在Open状态时会不定时尝试接口是否还存在故障, 如果仍有故障退回Open状态, 故障消除了的话就回到Closed状态.<!-- more -->

## Hystrix 和 Resilience4j

活跃于Java语言的断路器有两个: Hystrix 和 Resilience4j. Hystrix 由 Netfilx 开发, 目前已经停止开发, 所以本文将以 Resilience4j为主.

[Resilience4j](https://github.com/resilience4j/resilience4j)是一个开源项目, 在Github进行托管. 其特点是使用函数式编程范式编写.

## 引入Resilience4j

如果使用Spring Boot的话, 只需要使用:

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot2</artifactId>
    <version>1.2.0</version>
</dependency>
```

## 使用Resilience4j

Resilience4j默认使用装饰者模式, 但在Spring Boot项目中肯定是使用 `@CircuitBreaker` 的AOP注解方便.

详细的文档可以看[这里](https://resilience4j.readme.io/docs)

## Resilience4j组件

Retry: 重试

Circuit Breaker: 断路器

Rate Limiter: 限制比率

Time Limiter: 限制时间

Bulkhead: 隔板, 控制等待区(环形缓冲区)数量

Cache: 缓存

Fallback: 后备

## Resilience4j配置文件

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

## 重点内容

1. 断路器阻拦请求的功能
2. 断路器自动恢复到 HALF_OPEN 状态的功能
3. HALF_OPEN 的回归逻辑
4. 断路器拦截慢请求的功能
5. 断路器后备方法的使用 (fallback仍视作一次失败, 要绕过的话可以使用transitionToClosedState)