---
title: SpringBoot+MyBatis实战
date: 2019-04-14 22:35:05
tags: [java，spring，mybatis]
categories: [入门教程]
---

## ORM与MyBatis

Object Relational Mapping (ORM)，对象关系映射，用于实现数据库系统和面向对象编程语言类型系统之间的数据转换。效果上，它创建了一个可在编程语言中使用的“虚拟对象数据库”。

MyBatis是ORM的一种实现框架，对JDBC进行了又一层的封装。和Hibernate这种重量级的ORM框架相比，MyBatis非常轻量，其源码可以在数天之间就全部看完。这是因为和Hibernate提供了全套映射机制、自动生成SQL语句相比，MyBatis选择了拥抱SQL，让程序员自己写SQL，只是把查询结果转换到POJO中。

## Maven

Apache Maven，是一个软件项目管理及自动构建工具，使用项目对象模型 (Project Object Model，POM) 进行配置。

## 代码分层

一般而言，一个Web项目，自顶向下的访问顺序为：Controller -> Service -> Dao -> MyBatis -> Database

Controller用作URL映射；Service写业务逻辑；Dao 即 Data Access Object，数据访问对象，用途正如其名；Model定义实体类，即MyBatis从数据库中取出的实体。

此外，Controller和Service命名相对固定，Dao常被命名为Mapper或者Repository，Model还常被命名为Entity或者Domain。以下使用Controller -> Service -> Dao -> Model 的命名。
<!-- more -->

## SpringBoot+Mybatis配置

### SpringBoot 基础结构

`src/main/java` 程序开发以及主程序入口  
`src/main/resources` 配置文件  
`src/test/java` 测试程序

`src/main/java`目录下:  
model (entity, domain)：实体层，存放实体类；与数据库中的属性值基本保持一致，实现get和set的方法。  
service：服务层，主要是业务类代码。  
controller：控制层，负责页面访问控制。  
dao (mapper, repository)：数据访问层，和数据库通信。

### 引入Web模块

在 `pom.xml` 中添加web模块，也可在IDEA中的Spring Initializr中勾选:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

`pom.xml` 中默认有两个模块：`spring-boot-starter` 和 `spring-boot-starter-test`。

`spring-boot-starter` 是核心模块，包括自动配置支持、日志和YAML。如果引入了 `spring-boot-starter-web` *可以* 去掉此模块，因为web模块自动依赖此模块。

`spring-boot-starter-test` 是测试模块，包括JUnit、Hamcrest和Mockito。

`mybatis-spring-boot-starter` 是SpringBoot集成的MyBatis模块，可以完全使用注解实现CRUD，几乎完全不用写XML配置文件，轻松上手。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
</dependency>
```

### 第一个Controller

```java
@RestController
public class HelloWorldController {
    @RequestMapping("/hello")
    public String index() {
        return "Hello World";
    }
}
```

这时候访问 `http://localhost:8080/hello` 应该就可以看到Hello World了。

### 自定义Property

1. 配置Maven文件，如果是使用Intellij IDEA中的 `Spring Initializr` 创建项目的话，Maven应该是自动配置好的。

2. 在 `resources/application.properties` 添加数据库连接的相关配置

    ```text
    spring.datasource.url = jdbc:mysql://localhost:3306/test1?useUnicode=true&characterEncoding=utf-8
    spring.datasource.username = root
    spring.datasource.password = root
    ```

    SpringBoot会自动加载spring.datasource.*中的相关配置，数据源会自动注入到sqlSessionFactory中，sqlSessionFactory会自动注入到Mapper中，做到开箱即用。

### Spring Boot常用注解

#### 声明Bean的注解

`@Component`：没有明确角色的组件  
`@Service`：在业务逻辑层（service层）使用  
`@Repository`：在数据访问层（dao层）使用  
`@Controller`：在控制层（controller层）使用  
`@RestController`：是一个组合注解，等于 `@Controller` + `@ResponseBody`

此外，  
`@Scope`：作用在类和方法上，用于配置Spring Bean的作用域  
`@RequestMapping`：用来处理请求地址映射  
`@ResponseBody`：支持将返回值放在Response体内，而不是返回一个视图

#### 注入Bean的注解

`@Autowired`：实现自动装配，Spring IoC容器扫描到 `@Autowired` 注解会去查询实现类，并自动注入。  
`@Qualifier`：如果 `@Autowired` 指定的是一个接口，它有多个实现类，那么需要 `@Qualifier` 注解指定注入的实现类的名称。

#### 实现Java配置的注解

`@Configuration` 注解声明当前类是一个配置类，相当于Spring中的一个XML文件。  
`@Bean` 注解作用在方法上，声明当前方法的返回值是一个Java Bean。

### MyBatis注解

可以使用接口+注解的方式取代XML配置Mapper的方式，以完成Dao层的编写。在项目小、数据库交互少的情况下，这是更简洁、更优雅的方式。但当项目增长到一定规模，有大量复杂的CRUD的情况下，还是使用XML配置文件管理SQL语句更有效率。

`@Select` 负责查询。  
`@Insert` 负责插入。  
`@Update` 负责修改。  
`@Delete` 负责删除。  
`@Result` 修饰返回的结果集，关联实体类属性和数据库字段。如果实体类属性和数据库字段保持一致，就不需要这个注解。

```java
@Mapper
public interface UserDao {

    @Select("SELECT * FROM user WHERE name = #{name}")
    User findByName(@Param("name") String name);

    @Insert("INSERT INTO user(name,age) VALUES(#{name},#{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);

    @Update("UPDATE user SET age=#{age} WHERE name=#{name}")
    void update(User user);

    @Delete("DELETE FROM user WHERE id =#{id}")
    void delete(Long id);
}
```

### 实体类的编写

在实体类中定义属性，实现get和set方法，把数据库中的数据转换成Java程序能理解的实体类。

### 代码实现

以下以展示汽车销量的应用为例。

在Controller中注入依赖的Service。

```java
@Controller
public class CarSaleController {
    @Autowired
    CarSaleService carSaleService;

    @RequestMapping(value = {"/car/TotalSaleMonth"})
    @ResponseBody
    public CarTotalSaleMonth getCarTotalSaleEveryMonth() {
        return carSaleService.getCarTotalSaleEveryMonth();
    }
```

匹配的Service：

```java
@Service
public class CarSaleService {
    @AutoWired
    CarSaleDao carSaleDao;
    
    public CarTotalSaleMonth getCarTotalSaleEveryMonth() {
        return carSaleDao.selectCarTotalSaleEveryMonth();
    }
}
```

匹配的Dao：

```java
@Mapper
public interface CarSaleDao {
    // car_sale_month 是数据表名，其中存着month1...month12总共12个属性值
    @Select({"select * from car_sale_month"})
    CarTotalSaleMonth selectCarTotalSaleEveryMonth();
}
```

匹配的Model：

```java
public class CarTotalSaleMonth {
    int month1;
    int month2;
    // ...
    int month12;

    public int getMonth1() {
        return month1;
    }

    public void setMonth1(int month1) {
        this.month1 = month1;
    }

    // ...
}
```