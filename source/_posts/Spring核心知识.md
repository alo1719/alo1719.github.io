---
title: Spring核心知识
date: 2019-08-08 22:10:51
tags: [java, spring]
---

> 笔记摘录于《Spring 实战》第四版，结合个人总结与思考。

Spring 是一个轻量级的控制反转 (IoC) 和面向切面 (AOP) 的容器框架。Spring 不仅仅局限于服务器端开发，任何 Java 应用都能在简易性、可测试性和松耦合等方面从 Spring 中获益。

## 版本变更

Spring 2.5 引入了基于注解的组件扫描，大量消除了显式 XML 配置。  
Spring 3.0 引入了基于 Java 的配置，可以替代 XML。  
Spring 4.0 引入了条件化配置。Spring Boot 就是利用条件化配置实现自动配置的。

## POJO

POJO (Plain Old Java Object)，简单老式 Java 对象。Spring 竭力避免因自身的 API 弄乱你的应用代码。Spring 不会强制你实现 Spring 规范的接口或继承 Spring 规范的类，相反，在基于 Spring 构建的应用中，它的类通常没有任何痕迹表明你使用了 Spring。最坏的场景是，一个类或许会使用 Spring 注解，但它仍然是 POJO。
<!-- more -->

## JavaBeans

JavaBeans 是 Java 中一种特殊的类，可以将多个对象封装到一个对象 (bean) 中。特点是*可序列化*，提供*无参构造器*，提供 getter 方法和 setter 方法访问对象的属性。名称中的“Bean”是用于Java的可重用软件组件的惯用叫法。

## 依赖注入 (DI)

Spring 把相互协作的关系称为依赖关系。假如A组件调用了B组件的方法，我们可称A组件依赖于B组件。在传统的程序设计过程中，通常由调用者来创建被调用者的实例。在依赖注入的模式下，创建被调用者的工作不再由调用者来完成，因此称为控制反转；创建被调用者实例的工作通常由 Spring 容器来完成，然后注入给调用者，因此也称为依赖注入。

通过 DI，对象的依赖关系由系统中负责协调各对象的第三方组件在创建对象的时候进行设定。对象无需自行创建或管理它们的依赖关系，依赖关系将被自动注入到需要他们的对象当中去。

## AOP

AOP 允许把遍布应用各处的功能分离出来形成可重用的组件。AOP能使日志等服务模块化，并以声明的方式将它们应用到它们需要影响的组件中去。所造成的结果就是这些组件会具有更高的内聚性并且会更加关注自身的业务，完全不需要了解设计系统服务所带来的复杂性。

## 应用上下文

Spring 通过应用上下文 (Application Context) 装载 bean 的定义并把它们组装起来。Spring 应用上下文**全权负责**对象的创建和组装。

## Spring容器

Spring 自带了多个容器实现，可以归为两种不同的类型。

bean工厂 (org.springframework.beans.factory.BeanFactory) 是最简单的容器，提供基本的DI支持。

应用上下文 (org.springframework.context.ApplicationContext) 基于BeanFactory构建，最常用。

## Spring容器生命周期

![Spring容器生命周期](https://i.loli.net/2019/07/30/5d3fb280c110424872.png)

## Spring功能划分

![Spring功能划分](https://i.loli.net/2019/07/30/5d3fb34ccf49b86312.png)

## Spring装配机制

创建应用组件之间协作的行为通常称为装配 (wiring)。Spring有3种装配bean的方式。

- 在XML中进行显式配置
- 在Java中进行显示配置
- 隐式的bean发现机制和自动装配

尽可能使用自动装配的机制。进行显式配置时，JavaConfig是更好的方案，因为它更强大、类型安全并且对重构友好，它就是Java代码。

## 常用注解

`@Component` 注解表明该类会作为**组件类**，并告知Spring要为这个类创建 Bean。不过，组件扫描默认是不启用的，还需要显式配置一下Spring。`@ComponentScan` 注解能够在Spring中启用组件扫描。也可以在XML中使用 `<context:component-scan>` 元素。Spring会根据类名自动为扫描的bean指定一个ID。如果一个bean的类名为`SgtPeppers`，那么bean给定的ID为`sgtPeppers`，也就是类名首字母改为小写。

`@Autowired` 注解可以用在类的**任何方法**上。不管是构造器，Setter方法还是其他的方法，Spring都会尝试满足方法参数上所声明的依赖。假如有且只有一个 bean 匹配以来需求的话，那么这个 bean 将会被装配进来。如果没有匹配的 bean，那么在应用上下文创建的时候，Spring会抛出一个异常。如果有多个bean都能满足依赖关系的话，Spring也会抛出一个异常，表明没有明确指定要选择哪个bean进行自动装配。`@Autowired` 也可以用 `@Inject` 替换（大多数场景下可以相互替换）。

## 配置类

创建配置类的关键在于添加 `@Configuration` 注解。

使用配置类需要在类内部声明要返回的Bean，如下所示:

```java
@Bean
public CompactDisc sgtPeppers() {
  return new SgtPeppers();
}
```

实现依赖注入采用如下的方式:

```java
@Bean
public CDPlayer cdPlayer(CompactDisc compactDisc) {
  return new CDPlayer(compactDisc);
}
```

## XML配置

XML 配置 Spring 配置要以 `<beans>` 元素为根。

使用 `<bean>` 类似于 `@Bean` 注解。

```xml
<bean id="compactDisc" class="soundsystem.SgtPeppers" />
```

### 构造器注入bean引用

`<constructor-arg>` 元素会告知 Spring 要将一个 ID 为 compactDisc 的 bean 引用传递到CDPlayer 的构造器中。

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer">
  <constructor-arg ref="compactDisc" />
</bean>
```

使用c-命名空间实现DI:

```xml
<bean id="cdPlayer" class="soundsystem.CDPlayer"
      c:cd-ref="compactDisc" />
```

其中，`c:` 为c-命名空间前缀，`cd` 为构造器参数名（一般用_0，_1的形式），`-ref` 表示注入bean引用。

### 将字面量注入到构造器中

```xml
<bean id="compactDisc"
      class="soundsystem.BlankDisc">
  <constructor-arg value="Sgt. Pepper's Lonely Hearts Club Band" />
  <constructor-arg value="The Beatles" />
</bean>
```

### 设置属性

首先要编写属性的 Setter 方法，然后：

```xml
<bean id="compactDisc" class="soundsystem.BlankDisc">
  <property name="title" value="Sgt. Pepper's Lonely Hearts Club Band" />
</bean>
```

## 导入和混合配置

JavaConfig 使用 `@Import(SampleConfig.class)` 注解和 `@ImportResource("classpath:sample-config.xml")`。

XML使用 `<import resource="sample-config.xml" />`。

`<import>` 元素只能导入其他的XML配置文件，不能导入JavaConfig类，导入JavaConfig应该使用 `<bean>`。

## 高级装配

### 条件化的bean

使用 `@Conditional` 注解。

### 处理自动装配的歧义性

使用 `@Primary` 注解表明此 bean 为首选的 bean。

更强大的是 `@Qualifier` 注解，可以和 `@Autowired` 或 `@Inject` 协同使用。例如:

```java
@Autowired
@Qualifier("iceCream")
public void setDessert(Dessert dessert) {
    this.dessert = dessert;
}
```

也可以创建自定义的限定符:

```java
@Component
@Qualifier("cold")
public class IceCream implements Dessert { ... }
```

### Bean的作用域

在默认情况下，Spring 应用上下文中所有 bean 都是作为以单例 (singleton) 的形式创建的。也就是说，不管给定的一个bean被注入到其他bean多少次，每次所注入的都是同一个实例。

但有时候，如果所使用的类是易变的 (mutable)，它们会保持一些状态，因此重用是不安全的。Spring定义了多种作用域:

1. 单例 (singleton)
2. 原型 (prototype)：每次注入或者通过应用上下文获取的时候，都会创建一个新的实例。
3. 会话 (Session)：Web应用中，为每个会话创建一个实例。
4. 请求 (Request)：Web应用中，为每个请求创建一个实例。

可以通过 `@Scope("prototype")` 注解改变作用域。

### 运行时注入

#### 属性占位符

将属性定义到外部的文件中，并使用占位符值将其插入到 bean 中。占位符的形式为 `${...}`，例如:

```java
public BlankDisc(
      @Value("${disc.title}") String title,
      @Value("${disc.artist}") String artist) {
  this.title = title;
  this.artist = artist;
}
```

#### Spring表达式语言

SqEL的形式是 `#{...}`，`#{1}` 的结果就是1。

`#{T(System).currentTimeMillis()}` 中，`T()` 表达式会将 java.lang.System 视为 Java 中对应的类型，因此可以调用其 static 修饰的 `currentTimeMillis()` 方法。

## AOP的Spring

在软件开发中，散布于应用中多处的功能被称为横切关注点 (cross-cutting concern)。通常来讲，这些横切关注点从概念上是与应用的业务逻辑相分离的 (但是往往会直接嵌入到应用的业务逻辑之中)。把这些横切关注点与业务逻辑相分离正是面向切面编程所要解决的问题。AOP可以实现横切关注点与它们所影响的对象之间的解耦。

在使用面向切面编程时，我们仍然在一个地方定义通用功能，但是可以通过声明的方式定义这个功能要以何种方式在何处应用，而无需修改受影响的类。

### AOP术语

通知 (Advice): 切面的工作。Spring切面可以应用5种类型的通知:

1. 前置通知 (Before)：在目标方法被调用之前通知功能
2. 后置通知 (After)：在目标方法完成之后调用通知，此时不会关心方法的输出是什么
3. 返回通知 (After-returning)：在目标方法成功执行之后调用通知
4. 异常通知 (After-throwing)：在目标方法抛出异常后调用通知
5. 环绕通知 (Around)：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。

连接点 (Join point)：在应用执行过程中能够插入切面的一个点。  
切点 (Pointcut)：切点的定义会匹配通知所要织入的一个或多个连接点。  
切面 (Aspect)：切面是通知和切点的结合。  
引入 (Introduction)：引入允许我们向现有的类添加新方法或属性。  
织入 (Weaving)：织入是把切面应用到目标对象并创建新的代理对象的过程。在目标对象的生命周期里有多个点可以进行织入。

- 编译期：切面在目标类编译时被织入。这种方式需要特殊的编译器，AspectJ的织入编译器就是以这种方式织入切面的。
- 类加载期：切面在目标类加载到JVM时被织入。这种方式需要特殊的类加载器 (ClassLoader)，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ 5的加载时织入 (load-time weaving, LTW) 就支持以这种方式织入切面。
- 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式织入切面的。

### Spring对AOP的支持

Spring提供了4种类型的AOP支持，前3种都是Spring AOP实现的变体，Spring AOP构建在动态代理的基础上，因此，Spring对AOP的支持局限于方法拦截。如果AOP需求超过了简单的方法调用（如构造器或属性拦截），那么需要考虑使用AspectJ来实现切面。

- 基于代理的经典 Spring AOP
- 纯 POJO 切面
- @AspectJ 注解驱动的切面
- 注入式 AspectJ 切面（适用于Spring各个版本）

### Spring在运行时通知对象

![Spring在运行时通知对象.png](https://i.loli.net/2019/08/03/ZhwLsOWd96zYIrb.png)

通过在代理类中包裹切面，Spring在运行期把切面织入到Spring管理的bean中。

### 编写切点

![编写切点.png](https://i.loli.net/2019/08/03/wdGVmvpXL8zNAMI.png)

使用 `execution()` 指示器选择 Performance 的 perform() 方法，方法表达式以 * 开始，表明不关心方法返回值的类型。然后指明全限定类名和方法名。对于方法参数列表，使用两个点号表明切点要选择任意的 perform() 方法，无论该方法的入参是什么。

### AOP注解

`@Aspect` 注解定义切面类，`@Pointcut` 注解定义（可重用的）切点，`@Before`，`@After`，`@Around`，`@AfterReturning`，`@AfterThrowing` 定义通知方法。

```java
@Aspect
public class Audience {
    @Before("execution(** concert.Performance.perform(..))")
    public void silenceCellphones() {
      System.out.println("Silencing cell phones");
    }

    @After("execution(** concert.Performance.perform(..))")
    public void applause() {
      System.out.println("CLAP CLAP CLAP!!!");
    }    
}
```

```java
@Aspect
public class Audience {

    @Pointcut("execution(** concert.Performance.perform(..))")
    public void performance() {}

    @Before("performance()")
    public void silenceCellphones() {
      System.out.println("Silencing cell phones");
    }

    @After("performance()")
    public void applause() {
      System.out.println("CLAP CLAP CLAP!!!");
    }    
}
```

### 启用AOP

如果使用JavaConfig的话，可以在配置类上通过使用 `@EnableAspectJAutoproxy` 注解启用自动代理功能。假如你在 Spring 中要使用 XML 来装配 bean 的话，那么需要使用 Spring aop 命名空间中的 `<aop:aspectj-autoproxy>` 元素。

### 环绕通知

使用 `@Around` 注解，方法必须接受 `ProceedingJoinPoint` 参数。当要将控制权交给被通知的方法时，需要调用 `ProceedingJoinPoint` 的 `proceed()` 方法。

```java
@Aspect
public class Audience {
    @Pointcut(execution(** concert.Performance.perform(..)))
    public void performance() {}

    @Around("performance()")
    public void watchPerformance(ProceedingJoinPoint jp) {
        System.out.println("Silencing cell phones");
        jp.proceed();
        System.out.println("CLAP CLAP CLAP!!!");
    }
}
```

### 在XML中声明切面

使用XML声明切面的话就可以去掉所有注解。

```xml
<aop:config>
  <aop:aspect ref="audience">
    <aop:pointcut
      id="performance"
      expression="execution(* concert.Performance.perform(..))"/>
    <aop:before
      pointcut-ref="performance"
      method="silenceCellPhones"/>
  </aop:aspect>
</aop:config>
```

## Spring Boot

初期的 Spring 通过代码加配置的形式为项目提供了良好的灵活性和扩展性，但随着 Spring 越来越庞大，其配置文件也越来越繁琐，xml 文件太多一直是 Spring 被人诟病的地方。特别是近些年其他简洁的 Web 方案层出不穷，如 Python 或 Node.js，几行代码就能实现一个 Web 服务器，对比起来，大家渐渐觉得 Spring 那一套太过繁琐。此时，Spring 社区推出了 Spring Boot，它的目的在于实现自动配置，降低项目搭建的复杂度，如需要搭建一个接口服务，通过 Spring Boot 几行代码即可实现。可以理解为 Spring Boot 是“开箱即用”的 Spring 框架。

它使用“习惯优于配置”（项目中存在大量的配置，此外还内置了一个习惯性的配置，让你无需手动进行配置）的理念让你的项目快速运行起来。使用 Spring Boot 很容易创建一个独立运行（运行 jar，内嵌 Servlet 容器）、准生产级别的基于 Spring 框架的项目，使用 Spring Boot 可以不用或者只需要很少的 Spring 配置，适合微服务建构的开发。

## 微服务

微服务架构强调的重点是：业务系统需要彻底的组件化和服务化，原有的单个业务系统会拆分为多个可以独立开发、设计、运行和运维的小应用，这些小应用之间通过服务完成交互和集成。在微服务架构中我们强调彻底的组件化和服务化，每个微服务都可以独立的部署和投产，其实也就意味着很多的微服务有自己独立的数据库。

### Spring Cloud 生态圈

- **Spring Cloud Config 配置中心**，利用 Git 集中管理程序的配置。
- **Spring Cloud Netflix Eureka 服务中心**（类似于管家的概念，需要什么直接从这里取就可以了），一个基于 REST 的服务，用于定位服务，以实现服务发现和故障转移。
- **Spring Cloud Netflix Hystrix 熔断器**，容错管理工具，旨在通过熔断机制控制节点，从而对延迟和故障提供更强大的容错能力。
- **Spring Cloud Netflix Zuul 网关**，是在云平台上提供动态路由、监控、弹性、安全等边缘服务的框架。Web 后端所有请求的前门。
-  Spring Cloud Netflix Archaius 配置管理 API。
- **Spring Cloud Netflix Ribbon 负载均衡**。
- **Spring Cloud Netflix Fegin REST客户端**。
- **Spring Cloud Bus 消息总线**，利用分布式消息将服务和服务实例连接在一起，用于在一个集群中传播状态的变化。
- **Spring Cloud Cluster 集群工具**。
- Spring Cloud Consul 基于 Hashicorp Consul 实现的服务发现和配置管理。
- **Spring Cloud Security 安全控制**。
- **Spring Cloud Sleuth 分布式链路监控**，Spring Cloud 应用的分布式追踪系统，和 Zipkin、HTrace、ELK 兼容。
- Spring Cloud Task 任务工具。
- Spring Cloud Zookeeper 服务发现和配置管理，基于 Apache Zookeeper。
- Spring Cloud for Amazon Web Services 快速和亚马逊网络服务集成。

## Spring 解决循环依赖

```java

public class BeanA {
    @Autowired
    private BeanB beanB;
}

public class BeanB {
    @Autowired
    private BeanA beanA;
}
```

IOC 容器在读到上面的代码时，会按照顺序，先去实例化 beanA。然后发现 beanA 依赖于 beanB，接在又去实例化 beanB。实例化 beanB 时，发现 beanB 又依赖于 beanA。如果容器不处理循环依赖的话，容器会无限执行上面的流程，直到内存溢出、程序崩溃。当然，Spring 是不会让这种情况发生的。在容器再次发现 beanB 依赖于 beanA 时，容器会获取 beanA 对象的一个早期的引用（early reference），并把这个早期引用注入到 beanB 中，让 beanB 先完成实例化。beanB 完成实例化，beanA 就可以获取到 beanB 的引用，beanA 随之完成实例化。

| 缓存                  | 用途                                                         |
| --------------------- | ------------------------------------------------------------ |
| singletonObjects      | 用于存放完全初始化好的 bean，从该缓存中取出的 bean 可以直接使用 |
| earlySingletonObjects | 存放原始的 bean 对象（尚未填充属性），用于解决循环依赖       |
| singletonFactories    | 存放 bean 工厂对象，用于解决循环依赖                         |

这就是 Spring 的**三级缓存**机制。在创建 bean 的时候，首先想到的是从 cache 中获取这个单例的 bean，这个缓存就是 singletonObjects。如果获取不到，就再从二级缓存 earlySingletonObjects 中获取。如果还是获取不到且允许 singletonFactories 通过getObject() 获取，就从三级缓存 singletonFactory.getObject() 获取，如果获取到了则把工厂对象从 singletonFactories 中移除，并把 bean 放入 earlySingletonObjects 中。

