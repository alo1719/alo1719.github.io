---
title: 从一次@Transactional注解失效探究Spring AOP的代理机制
date: 2021-03-09 16:04:40
tags: [java, aop]
categories: 
---

## 问题还原

```java
public class Demo {

    public void processor(Message message) {
        
        Result result = save(message);

    }

    @Transactional(timeout = 30, rollbackFor = Exception.class)
    public Result save(Message message) {
        serviceA.save(message);

        Result result = serviceB.save(message);

        return result;
    }
}
```

集成测试方法会调用 `Demo.processor`，然后 `processor` 方法会调用带有 @Transactional 注解的 `save` 方法。save 方法中的 `serviceA.save` 和 `serviceB.save` 方法都是带有 @Transactional 注解的。

但是经过测试发现，ServiceA 和 serviceB 的事务都能生效，但是 Demo 类中 save 方法的事务无法生效，会出现 serviceA 成功，serviceB 失败，数据库中只写入了 serviceA 的结果的情况。很明显，Demo.save 方法上的 @Transactional 注解失效了。
<!-- more -->

## 问题排查

首先，我想到的是事务的传播方式。可是 @Transactional 注解默认的传播方式为 `Propagation.REQUIRED`，即如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。在上面的例子中，Demo.save 方法中 serviceA.save 和 ServiceB.save 都包含事务，那么这三个事务会合成一个大事务，是符合预期的。

之后，通过搜索引擎了解到，​这是 Spring AOP 动态代理造成的，因为只有当事务方法被当前类以外的代码调用时，才会由 Spring 生成的代理对象来管理。在 Demo 类上添加了 @Transactional 注解，问题解决。（@Transactional 注解可以放在类和方法上，放在类上，表明该类的所有 `public` 方法都配置相同的事务信息。当配置在方法上时，方法的事务配置会覆盖类的事务配置。）

但同时带来了一个小问题，此时 processor 方法也加上了 @Transactional 注解，而这段代码本不需要事务，对性能会有一定影响。

## 探究Spring AOP

上面提到，在 Spring 默认的代理模式下，只有目标方法由外部调用，才能被 Spring 的事务拦截器拦截。在同一个类中的两个方法直接调用，不会被 Spring 的事务拦截器拦截。

但这种说法很奇怪。为什么只有目标方法由外部类调用，Spring 才能拦截这一请求呢？Spring 完全可以做到在调用带有 @Transactional 注解的方法时就进行拦截，不管这个方法是由谁调用的啊？

所以说，搜索引擎上的归因**没有讲到点子上**。最起码，这个原因无法让人信服。要搞清楚这个问题，需要深入了解 Spring AOP 的实现方式。

首先，我们知道 Spring AOP 是通过动态代理的方式实现的。那么，Spring 是在什么时候生成代理类的呢？网络上有人说是在运行时，有人说是在类加载期，但最准确的说法是在 *IoC容器初始化Bean* 时。详情可见  
`org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator`：

```java
/**
* Create a proxy with the configured interceptors if the bean is
* identified as one to proxy by the subclass.
* @see #getAdvicesAndAdvisorsForBean
*/
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

回到上面的例子，在 Spring IoC 容器初始化 Demo 类时，Spring 扫描到该类中有方法使用了 @Transactional 注解，并且并没有实现接口，便使用 CGLIB 生成了代理类，此代理类继承 Demo，由这个代理类来统一接受请求。

也就是说，不管 @Transactional 注解是加在 processor 上、加在 save 上或者加在类上，Spring 都会创建这个类的代理类，并不会因为外部调用的方法没有 @Transactional 注解就不生成代理类。

![Snipaste_2021-03-10_16-41-25.png](https://i.loli.net/2021/03/10/LwIQDEdTHCt8Nfg.png)

但是 Spring AOP 无论是 JDK 动态代理，还是 CGLIB 动态代理，都保留了原类和增强类两个类。它的意思是，增强类仍然持有原类（CGLIB 采用继承的方式，JDK 动态代理采用反射的方式），体现在图中就是代理类把原类包装了起来，*原类并没有被增强*。

当外部调用 processor 方法，代理类判断此方法不需进行事务拦截，直接调用原类。原类再调用 this.save，此时 this 指向的是原类，并不含有事务拦截逻辑（事务拦截逻辑在代理类中），因此注解失效。

此时，如果外部调用 save 方法，是会经过代理类拦截的，于是产生了网络上“只有目标方法由外部调用，才能被 Spring 的事务拦截器拦截”的说法。

![Snipaste_2021-03-10_16-52-51.png](https://i.loli.net/2021/03/10/zY7OneaoR1vxD84.png)

## 深层原因

从上图中可以看出，此时 @Transactional 注解失效的原因在于原类中的 this 并没有被增强。如果 this 能够指向外部的 Proxy 类，这个问题就不会发生了。或者更进一步，既然我们都有代理类了，为什么还需要原类？直接在原类上增强，生成一个新的类，并只用这个增强类不就**一劳永逸**了吗？实际上，AspectJ 就是用这种方式增强类、实现 AOP 的。

但是 Spring AOP 采用这样的逻辑是有原因的。Spring AOP 的核心理念就是**动态代理**，并且把 JDK 代理（通过接口使用反射增强类）和 CGLIB 增强（使用字节码增强生成子类）两种实现方式封装在一起，并能无缝切换。因此，尽管 CGLIB 增强能够生成一个全新的类，但 Spring AOP 还是通过依附于原类生成代理类的方式进行增强。（从目前的视角看，JDK 代理相比于 CGLIB 增强没有任何优势，但在 Java 5 时代 JDK 代理的性能是显著优于 CGLIB 增强的，算是一个历史遗留问题了。）

Spring AOP 采用动态代理的好处就是做到了**非侵入**。这样做一在 Debug 时会调用到原类，方便定位到问题，提高了排查问题的效率；二代码增强并不是 100% 成功的，最可怕的场景是类被成功增强了，但是逻辑不符合预期。Spring AOP 就算生成的代理类存在问题，起码还能保证原类的逻辑是正确的。若是像 AspectJ 这样只使用增强类，若是增强过程发生了问题，其后果就会严重很多。

这也导致了 Spring AOP 相比于 AspectJ，功能很弱。例如，Spring AOP 只能在 public 方法上进行拦截，对于 private 和 protected 方法不会生效，AspectJ 没有这个限制。又例如，Spring AOP 没办法在类的属性上、构造器上进行拦截，AspectJ 没有这个限制。但引入 AspectJ 需要用到 AJC (AspectJ Compiler)，且不太好和 Lombok 共存，引入的成本比较高。

总结而言，Spring AOP 的动态代理实现方式**并不符合直觉**，也因此导致了上面例子中的问题。但是它这样实现也是有利有弊的，最大的好处是做到了侵入性很弱。

## 最佳实践

如果事务方法会被类内部的方法使用，为了不让 @Transactional 注解失效，按解决成本从低到高排序，有三种方法：

1. 把事务方法提到一个单独的类中。这也是我最后使用的方法。
2. 使用编程式事务，手动开启事务。
3. 改用 AspectJ 进行 AOP 增强。