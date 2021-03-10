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

集成测试方法会调用 `Demo.processor`，然后 `processor` 方法会调用带有@Transactional注解的 `save` 方法。save方法中的 `serviceA.save` 和 `serviceB.save` 方法都是带有@Transactional注解的。

但是经过测试发现，ServiceA和serviceB的事务都能生效，但是Demo类中的save方法的事务无法生效，会出现了serviceA成功，serviceB失败，数据库中只写入了serviceA的结果的情况。很明显，Demo.save方法上的@Transactional注解失效了。
<!-- more -->

## 问题排查

首先，我想到的是事务的传播方式。可是@Transactional注解默认的传播方式为 `Propagation.REQUIRED`，即如果当前存在事务，则加入该事务，如果当前不存在事务，则创建一个新的事务。在上面的例子中，Demo.save方法中serviceA.save和ServiceB.save都包含事务，那么这三个事务会合成一个大事务，是符合预期的。

之后，通过搜索引擎了解到，​这是因为Spring AOP动态代理造成的，因为只有当事务方法被当前类以外的代码调用时，才会由Spring生成的代理对象来管理。在Demo类上添加了@Transactional注解，问题解决。

@Transactional注解可以放在类和方法上，放在类上，表明该类的所有 `public` 方法都配置相同的事务信息。当配置在方法上时，方法的事务配置会覆盖类的事务配置。

但同时带来了一个小问题，此时processor方法也加上了@Transactional注解，而这段代码本不需要事务化，对性能会有一定影响。

## 探究Spring AOP

上面提到，在默认的代理模式下，只有目标方法由外部调用，才能被 Spring 的事务拦截器拦截。在同一个类中的两个方法直接调用，不会被 Spring 的事务拦截器拦截。

但这种说法很奇怪。为什么只有目标方法由外部类调用，Spring才能拦截这一请求呢？Spring完全可以做到在调用带有@Transactional注解的方法时就进行拦截，不管这个方法是由谁调用的啊？

所以说，搜索引擎上的结论**没有讲到点子上**。最起码，没有深入思考。要搞清楚这个问题，需要深入了解Spring AOP的实现方式。

首先，我们知道Spring AOP是通过动态代理的方式实现的。那么，Spring是在什么时候生成代理类的呢？网络上有人说是在运行时，有人说是在类加载期，但最准确的说法是在*IoC容器初始化Bean* 时。详情可见  
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

回到代码，在Spring IoC容器注入Demo类时，Spring 扫描到该类中有方法使用了@Transactional注解，并且并没有实现接口，便使用CGLIB生成了代理类，此代理类继承Demo，由这个代理类来统一接受请求。

也就是说，不管@Transactional注解是加在processor上、加在save上或者加在类上，Spring都会创建这个类的代理类，并不会因为外部调用的方法没有@Transactional注解就不生成代理类。

![Snipaste_2021-03-10_16-41-25.png](https://i.loli.net/2021/03/10/LwIQDEdTHCt8Nfg.png)

但是Spring AOP无论是JDK动态代理，还是CGLIB动态代理，都保留了原类和增强类两个类。它的意思是，增强类仍然持有原类（CGLIB采用继承的方式，JDK动态代理采用反射的方式），体现在图中就是代理类把原类包装了起来，*原类并没有被增强*。

当外部调用processor方法，代理类判断此方法不需进行事务拦截，直接调用原类。原类再调用this.save，此时this指向的是原类，并不含有事务拦截逻辑（事务拦截逻辑在代理类中），因此注解失效。

此时，如果外部调用save方法，是会经过代理类拦截的，就产生了网络上“只有目标方法由外部调用，才能被 Spring 的事务拦截器拦截”的说法。

![Snipaste_2021-03-10_16-52-51.png](https://i.loli.net/2021/03/10/zY7OneaoR1vxD84.png)

## 深层原因

从上图中可以看出，此时@Transactional注解失效的原因在于原类中的this并没有被增强。如果this能够指向外部的Proxy类，这个问题就不会发生了。或者更进一步，既然我们都有代理类了，为什么还需要原类？直接在原类上增强，生成一个新的类，并只用这个增强类不就一劳永逸了吗？实际上，AspectJ就是用这种方式增强类、实现AOP的。

但是Spring AOP采用这样的逻辑是有原因的。Spring AOP的核心理念就是**动态代理**，并且把JDK代理（通过接口使用反射增强类）和CGLIB增强（使用字节码增强生成子类）两种实现方式封装在一起，并能无缝切换。因此，尽管CGLIB增强能够生成一个全新的类，但Spring AOP还是通过依附于原类生成代理类的方式进行增强。

Spring AOP采用动态代理的好处就是做到了**非侵入**。这样一来，Debug时会调用到原类，方便定位到问题，提高了排查问题的效率；此外，代码增强并不是100%成功的，最可怕的场景是类被成功增强了，但是逻辑不符合预期。Spring AOP就算生成的代理类存在问题，起码还能保证原类的逻辑是正确的。若是像AspectJ这样只使用增强类，若是增强过程发生了问题，其后果就会严重很多。

这也导致了Spring AOP相比于AspectJ，功能很弱。例如，Spring AOP只能在public方法上进行拦截，对于private和protected方法不会生效，AspectJ可以。又例如，Spring AOP没办法在类的属性上、构造器上进行拦截，AspectJ可以。但引入AspectJ需要用到AJC (AspectJ Compiler)，且不太好和Lombok共存，引入的成本有点高。

## 最佳实践

如果事务方法会被类内部的方法使用，为了不让@Transactional注解失效，按解决成本从低到高排序，有三种方法：

1. 把事务方法提到一个单独的类中。这也是我最后使用的方法。
2. 使用编程式事务，手动开启事务。
3. 改用AspectJ进行AOP增强。