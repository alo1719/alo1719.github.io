---
title: SpringBoot+Mybatis实战
date: 2019-04-14 22:35:05
tags: [java, spring]
---

## ORM与MyBatis

Object Relational Mapping(ORM), 对象关系映射, 用于实现面向对象编程语言中不同类型系统的数据之间的转换. 实际上, 它创建了一个可在编程语言中使用的"虚拟对象数据库". 而这之中进行映射的就是Mapper.

MyBatis是ORM的一种实现框架, 是对JDBC的一种封装.  
`mybatis-spring-boot-starter`可以完全注解不用写配置文件, 简单配置轻松上手.

## Maven

Apache Maven, 是一个软件项目管理及自动构建工具, 使用项目对象模型(Project Object Model, POM)来配置.

## 业务逻辑

业务逻辑: controller -> service interface -> servicelmpl -> dao interface -> daolmpl -> mapper -> database

<!-- more -->

总共4层, controller和service命名相对固定, 其中dao层还可以取名为mapper, model层还可以取名为entity或者domain, 个人使用controller -> service -> dao -> model 的命名层级.

controller一般作URL映射, service作业务逻辑, Dao进行实体类和数据库的映射, model定义实体类.

## Spring Boot+Mybatis配置

### Spring Boot 基础结构

`src/main/java` 程序开发以及主程序入口  
`src/main/resources` 配置文件  
`src/test/java` 测试程序

`src/main/java`目录下:  
`model`(entity, domain)实体层, 存放实体类; 与数据库中的属性值基本保持一致, 实现set和get的方法.  
`service`服务层, 主要是业务类代码.  
`controller`(web)控制层负责页面访问控制.  
`dao`(mapper)dao层

### 引入Web模块

在`pom.xml`中添加web模块, 可在IDEA中Spring Initializr中勾选:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

`pom.xml`中默认有两个模块: `spring-boot-starter`和`spring-boot-starter-test`.

`spring-boot-starter`是核心模块, 包括自动配置支持/日志和YAML, 如果引入了`spring-boot-starter-web`*可以*去掉此配置, 因为web模块自动依赖此模块.

`spring-boot-starter-test`是测试模块, 包括JUnit / Hamcrest / Mockito.

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

这时候访问`http://localhost:8080/hello`应该就可以看到Hello World了.

### 自定义Property

配置在application.propertioes中:

1. 配置Maven文件, 如果通过Intellij IDEA中的`Spring Initializr`创建项目的话, Maven应该是自动配置好的.

2. `resources/application.properties`添加数据库连接的相关配置

    ```text
    spring.datasource.url = jdbc:mysql://localhost:3306/test1?useUnicode=true&characterEncoding=utf-8
    spring.datasource.username = root
    spring.datasource.password = root
    ```

    SpringBoot会自动加载spring.datasource.*中的相关配置, 数据源会自动注入到sqlSessionFactory中, sqlSessionFactory会自动注入到Mapper中, 开箱即用.

### Spring Boot常用注解

使用Spring Boot中的`@Controller`, `@Service`以及`@Autowired`注入进行controller层和service层的编写.

#### 声明Bean的注解

`@Component`: 没有明确角色的组件  
`@Service`: 在业务逻辑层(service层)使用  
`@Repository`: 在数据访问层(dao层)使用  
`@Controller`: 用于标注控制层(controller层)组件  
`@RestController`: 是一个组合注解, 等于`@Controller`+`@ResponseBody`

`@Scope`: 作用在类和方法上, 用于配置Spring Bean的作用域.  
`@RequestMapping`: 用来处理请求地址映射.  
`@ResponseBody`: 支持将返回值放在response体内, 而不是返回一个视图

#### 注入Bean的注解

`@Autowired`: 实现自动装配, Spring IoC容器扫描到`@Autowired`注解会去查询实现类, 并自动注入.  
`@Qualifier`: 如果`@Autowired`指定的是一个接口, 它有多个实现类, 那么需要`@Qualifier`注解指定装配的实现类的名称.

#### 实现Java配置的注解

`@Configuration`注解声明当前类是一个配置类, 相当于Spring中的一个XML文件.  
`@Bean`注解作用在方法上, 声明当前方法的返回值是一个Bean.

### MyBatis注解

可以使用接口+注解的方式取代xml配置Mapper的方式, 以完成对dao层的编写.

`@Select`是查询类的注解, 所有的查询均使用这个.  
`@Result`修饰返回的结果集, 关联实体类属性和数据库字段一一对应, 如果实体类属性和数据库属性名保持一致, 就不需要这个属性来修饰.  
`@Insert`插入数据库使用, 直接传入实体类会自动解析属性到对应的值.  
`@Update`负责修改, 也可以直接传入对象.  
`@Delete`负责删除.

```java
@Mapper
public interface UserDao {

    @Select("SELECT * FROM user WHERE name = #{name}")
    User findByName(@Param("name") String name);

    @Insert("INSERT INTO user(name, age) VALUES(#{name}, #{age})")
    int insert(@Param("name") String name, @Param("age") Integer age);

    @Update("UPDATE user SET age=#{age} WHERE name=#{name}")
    void update(User user);

    @Delete("DELETE FROM user WHERE id =#{id}")
    void delete(Long id);
}
```

### 实体类的编写

在实体类中定义属性, 实现get和set方法, 以完成把数据库中的数据转换成Java程序能理解的实体类.

### 代码实现

在Controller中要注入依赖的Service.

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

匹配的Service:

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

匹配的Dao:

```java
@Mapper
public interface CarSaleDao {
    // car_sale_month 是数据表名, 其中存着month1...month12总共12个属性值
    @Select({"select * from car_sale_month"})
    CarTotalSaleMonth selectCarTotalSaleEveryMonth();
}

匹配的Model:

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