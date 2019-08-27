---
title: Spring Boot学习笔记
date: 2019-08-27 16:56:05
tags: [java, spring]
---

## Spring Boot简介

Spring Boot提供了四个主要的特性, 能够改变开发Spring应用程序的方式:

- 自动配置: Spring Boot的自动配置特性利用了Spring 4对条件化配置的支持, 合理地推测应用所需的bean并自动化配置它们.
- Spring Boot Starter: 将常用的依赖分组进行了整合, 将其合并到一个依赖中, 这样就可以一次性添加到项目的Maven或Gradle构建中.
- 命令行接口(Command-line interface, CLI): Spring Boot的CLI发挥了Groovy编程语言的优势, 并结合自动配置进一步简化Spring应用的开发.
- Actuator: 深入运行中的Spring Boot应用程序, 在运行时检视应用程序内部情况.

Spring Boot采用自动配置和Starter来消除Spring项目中的样板式配置.
<!-- more -->
自动配置指: 如果Spring Boot在应用程序的classpath里发现了H2数据库的库, 那么它就会自动配置一个嵌入式H2数据库. 如果在classpath中发现`JdbcTemplate`, 那么它还会配置一个`JdbcTemplate`的Bean.

Spring Boot可以把Web应用程序变为可自执行的JAR文件, 不用部署到传统Java应用服务器里就能在命令行里运行. Spring Boot在应用程序里嵌入了一个Servlet容器(Tomcat, Jetty或Undertow). 从根本上来说, Spring Boot项目只是普通的Spring项目, 只是它们正好用到了Spring Boot的起步依赖和自动配置而已(没有它你自己也会去做).

## Spring Initializr

Spring Initializr能提供一个基本的项目结构, 以及一个用于构建代码的Maven或Gradle构建说明文件. 也可以用Spring Boot CLI初始化项目.

### 目录结构

`build.gradle`: Gradle构建说明文件.
`pom.xml`: Maven构建说明文件.
`src/main/java/myapp/Application.java`: 一个带有`main()`方法的类, 用于引导启动应用程序. 它不仅是启动引导类, 也是配置类.
`src/test/java/myapp/ApplicationTests.java`: 一个空的JUnit测试类, 加载了一个使用Spring Boot自动配置功能的Spring应用程序上下文.
`src/main/resources/application.properties`: 一个空的properties文件, 可以根据需要添加配置属性.
`src/main/resources/static`: 存放Web应用程序的静态内容.
`src/main/resources/templates`: 存放呈现模型数据的模板.
`src/test/resources`: 存放测试资源.

## Starter依赖

Spring Boot提供更加粗粒度的依赖, 例如, `spring-boot-starter-web`包含了`spring-boot-starter`、 `spring-boot-starter-tomcat`、 `jackson-databind`、 `spring-web`、 `spring-webmvc`的依赖. `spring-boot-starter-test`为测试模块, 包括JUnit, Hamcrest和Mockito. `spring-boot-starter`是核心模块, 包括自动配置支持/日志和YAML.

如果使用Maven进行管理, 配置方式为:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

使用Gradle的话配置方式为:

```groovy
dependencies {
  compile("org.springframework.boot:spring-boot-starter-web")
}
```

## Groovy

```groovy
@RestController
class HelloController {
    @RequestMapping("/")
    def hello() {
        "Hello World"
    }
}
```

只需要`spring run HelloController.groovy`即可启动整个应用程序.

## @SpringBootApplication注解

```java
@SpringBootApplication
public class ReadingListApplication {
    public static void main(String[] args) {
        SpringApplication.run(ReadingListApplication.class, args);
    }
}
```

`@SpringBootApplication`开启了Spring的组件扫描和Spring Boot的自动配置功能. 实际上, `@SpringBootApplication`将`@SpringBootConfiguration`, `@ComponentScan`和`@EnableAutoConfiguration`三个有用的注解组合在了一起. 源码如下所示:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
        @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
        @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
    ...
}
```

### @SpringBootConfiguration

首先来看`@SpringBootConfiguration`的源码:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```

其中的`@Configuration`作用是配置Spring容器, 源自Spring. `@Configuration`和`@SpringBootConfiguration`都是将当前类标注为配置类, 并将当前类里以`@Bean`注解标记的方法的实例注入到Spring容器中, 实例名就是方法名.

### @EnableAutoConfiguration

`@EnableAutoConfiguration`的源码如下:

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
}
```

`@EnableAutoConfiguration`注解启用自动配置, 可以帮助`SpringBoot`应用将符合条件的`@Configuration`配置都加载到当前IoC容器之中.

从宏观角度概括, `@EnableAutoConfiguration`从classpath下扫描所有的 `META-INF/spring.factories` 配置文件, 并将 `spring.factories` 文件中的 `EnableAutoConfiguration` 对应的配置项通过反射机制实例化为对应标注了 `@Configuration` 的形式的IoC容器配置类, 然后注入IoC容器.

### @ComponentScan

`@ComponentScan` 对应于XML配置形式中的 `context:component-scan`, 用于将一些标注了特定注解的bean定义批量采集注册到Spring的IoC容器之中, 这些特定的注解大致包括:

- @Controller
- @Entity
- @Component
- @Service
- @Repository

等等

## application.properties文件

加入一行`server.port=8000`就可以把监听端口从默认的8080改变到8000.

`application.properties`文件可以细粒度地调整Spring Boot地自动配置. 完全不用告诉Spring Boot加载`application.properties`, 只要它存在就会被加载, Spring和应用程序代码都能获取其中的属性.

## Spring Boot构建过程解析

Spring Boot为Gradle和Maven提供了构建插件, 以便辅助构建Spring Boot项目. 构建插件的主要功能是把项目打包成一个可执行的超级JAR(uber-JAR), 其中的内容能够用`java -jar`来运行应用程序.

使用Maven构建的话还可以将`spring-boot-starter-parent`作为上一级, 利用Maven的依赖管理功能, 继承很多常用库的依赖版本, 不用再去指定版本号. Gradle原生没有这样的功能, Spring Boot Gradle插件提供了这个特性.

### 排除传递依赖

Spring Boot的Web起步依赖传递依赖了Jackson JSON库, 排除掉传递依赖可以为项目瘦身.

```groovy
compile("org.springframework.boot:spring-boot-starter-web") {   exclude group: 'com.fasterxml.jackson.core'
}
```

```xml
<dependency>
  <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <groupId>com.fasterxml.jackson.core</groupId>
    </exclusion>
  </exclusions>
</dependency>
```

Maven总是会用最近的依赖, 而Gradle使用库的更新版本. 所以Gradle使用老版本的Jackson时需要把Web起步依赖传递依赖的版本排除掉, 并加入老版本.

### 定义领域模型

```java
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;
    private String reader;
    private String title;

    public Long getID() {
        return id;
    }

    public void setID(Long id) {
        this.id = id;
    }

    public String getReader() {
        return reader;
    }

    public void setReader(String reader) {
        this.reader = reader;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }
}
```

`@Entity`注解表明它是一个JPA实体, `id`属性加了`@Id`和`@GeneratedValue`注解, 说明这个字段是实体的唯一标识, 并且这个字段的值是自动生成的.

### 定义仓库接口

把`Book`对象持久化到数据库仓库. 因为用了Spring Data JPA, 只要简单地定义一个接口, 拓展一下Spring Data JPA地`JpaRepository`接口.

```java
public interface ReadingListRepository extends JpaRepository<Book, Long> {
    List<Book> findByReader(String reader);
}
```

通过拓展`JpaRepository`, `ReadingListRepository`直接继承了18个执行常用持久化操作的方法. `JpaRepository`是个泛型接口, 有两个参数: 仓库操作的领域对象类型, 及其ID属性的类型. 此外, 还增加了一个`findByTitle()`方法, 可以根据标题来查找阅读列表.

### 创建Web界面

```java
@Controller
@RequestMapping("/")
public class ReadingListController {
    private ReadingListRepository readingListRepository;

    @Autowired
    public ReadingListController(ReadingListRepository readingListRepository) {
        this.readingListRepository = readingListRepository;
    }

    @RequestMapping(value="/{reader}", method=RequestMethod.GET)
    public String readersBooks(
            @PathVariable("reader") String reader, Model model) {
        List<Book> readingList = readingListRepository.findByReader(reader);
        if (readingList != null) {
            model.addAttribute("books", readingList);
        }
        return "readingList";
    }

    @RequestMapping(value="/{reader}", method=RequestMethod.POST)
    public String addToReadingList(
            @PathVariable("reader") String reader, Book book) {
        book.setReader(reader);
        readingListRepository.save(book);
        return "redirect:/{reader}";
    }
}
```

`readersBooks()`处理`/{reader}`上的HTTP GET请求, 根据路径里指定的读者, 从(通过控制器的构造器注入的)仓库获取Book列表. 随后将这个列表塞入模型, 用的键是books, 最后返回readingList作为呈现模型的试图逻辑名称.

`addToReadingList()`处理`/{reader}`上的HTTP POST请求, 将请求正文里的数据绑定到一个Book对象上. 该方法把Book对象的reader属性设置为读者的姓名, 随后通过仓库的`save()`方法保存修改后的Book对象, 最后重定向到/{reader}(控制器中的另一个方法会处理该请求).

书中的前端页面采用了Thymeleaf模板, 此处不再赘述.

至此, 一个完整的Spring Boot应用程序构建完成.

## 覆盖Spring Boot自动配置

Spring Boot对安全特性的自动配置做得还不够好. 覆盖自动配置很简单, 就当自动配置不存在, 直接显式地写一段配置. 这段显式配置的形式不限, Spring支持的XML和Groovy形式配置都可以.

## 通过属性文件外置配置

Spring Boot应用程序有多种设置途径. Spring Boot能从多种属性源获得属性, 包括如下几处:

1. 命令行参数
2. `java:comp/env`里的JNDI属性
3. JVM系统属性
4. 操作系统环境变量
5. 随机生成的带`random.*`前缀的属性(在设置其他属性时, 可以引用它们, 比如`${random.long}`)
6. 应用程序以外的application.properties或者appliaction.yml文件
7. 打包在应用程序内的application.properties或者appliaction.yml文件
8. 通过`@PropertySource`标注的属性源
9. 默认属性

这个列表按照优先级排序, 也就是说, 任何在高优先级属性源里设置的属性都会覆盖低优先级的相同属性.

application.properties和application.yml文件能放在以下四个位置.

1. 外置, 在相对于应用程序运行目录的/config子目录里.
2. 外置，在应用程序运行的目录里
3. 内置，在config包内
4. 内置，在classpath根目录

同样, 这个列表按照优先级排序. 也就是说, /config子目录里的application.properties会覆盖应用程序classpath里的application.properties中的相同属性.

### 常用配置

在application.properties中设置(也可以用application.yml配置):

更改日志配置:

```text
logging.path=/var/logs/
logging.file=BookWorm.log
logging.level.root=WARN
logging.level.root.org.springframework.security=DEBUG
```

配置数据源:

```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8
    username: dbuser
    password: dbpass
```

## Spring Boot测试

在编写单元测试的时候, Spring通常不需要介入. 但是, 集成测试要用到Spring.

Spring自1.1.1版就向集成测试提供了极佳的支持. 自Spring 2.5开始, 集成测试支持的形式就变成了SpringJUnit4ClassRunner. 这是一个JUnit类运行器, 会为JUnit测试加载Spring应用程序上下文, 并为测试类自动织入所需的Bean.

要恰当地测试一个Web应用程序, 你需要投入一些实际的HTTP请求, 确认它能正确地处理那些请求. 幸运的是, Spring Boot开发者有两个可选的方案能实现这类测试.

- Spring Mock MVC: 能在一个近似真实的模拟Servlet容器里测试控制器, 而不用实际启动应用服务器.
- Web集成测试: 在嵌入式Servlet容器(比如Tomcat或Jetty)里启动应用程序, 在真正的应用服务器里执行测试.

## Groovy与Spring Boot CLI

Groovy并不要求有`public`和`private`这样的限定符, 也不要求行尾有分号.

## Actuator

要启用Actuator的端点, 只需在项目中引入Actuator的起步依赖即可.

```groovy
compile 'org.springframework.boot:spring-boot-starter-actuator'
```

```xml
<dependency>
  <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### 获得Bean装配报告

向`/beans`发起GET请求后, 会获得Bean装配报告的JSON文档. 所有的Bean条目都有5类信息.

- bean: Bean的名称或ID
- resource: `.class`文件的物理位置, 通常是一个URL, 指向构建出的JAR文件. 这会随着 应用程序的构建和运行方式发生变化.
- dependencies: 当前Bean注入的Bean ID列表.
- scope: Bean的作用域(通常是单例).
- type: Bean的Java类型.

`/autoconfig`端点能报告为什么会有这个Bean, 或者为什么没有这个Bean. `/env`端点会生成应用程序可用的所有环境属性的列表, 包括环境变量、JVM属性、命令行参数，以及application.properties或application.yml提供的属性. `/env`还能获取单个属性的值, 例如`/env/amazon.associate_id`.

`/mappings`罗列出应用程序发布的全部端点. `/metrics`端点能查看运行时数据, 例如GC、内存、线程池等等. `/trace`端点能报告所有Web请求的详细信息. `/dump`端点生成当前线程活动的快照. `/health`端点列出健康指示器, 例如数据库是否能连上、邮件服务器是否能连上等.

## 部署Spring Boot应用程序

### 运行方式

- 在IDE中运行应用程序
- 使用Maven的`spring-boot:run`或Gradle的`bootRun`
- 使用Maven或Gradle生成可运行的JAR文件, 随后在命令行中运行
- 使用Spring Boot CLI在命令行中运行Groovy脚本
- 使用Spring Boot CLI来生成可运行的JAR文件, 随后在命令行中运行

追对应用服务器的部署, 常需要把应用程序打包成一个WAR文件.

### 构建WAR文件

如果使用Gradle, 只需应用WAR插件`apply plugin: 'war'`, 并在`build.gradle`里用war配置替换原来的jar配置(只需把jar换成war).

使用Maven的话, 获取WAR会更加容易, 只要把`<packaing>`元素的值从jar改成war. 但是以上方法要启用Spring MVC DispatcherServlet的`web.xml`文件或者Servlet初始化类.

而Spring Boot提供的SpringBootServletInitializer是一个支持Spring Boot的Spring WebApplicationInitializer实现.

要使用SpringBootServletInitializer, 只需创建一个子类, 覆盖configure()方法来指定Spring配置类.

```java
public class ReadingListServletInitializer extends SpringBootServletInitializer {
    @Override
    protected SpringApplicationBuilder configure( SpringApplicationBuilder builder) {
        return builder.sources(Application.class);
    }
}
```

`configure()`方法传入了一个SpringApplicationBuilder参数, 并将其作为结果返回. 期间它调用了`sources()`方法注册了一个Spring配置类(即Application.class). 这个类既是启动类(带有main方法), 也是一个Spring配置类.

之后可以使用`gradle build`或者`mvn package`打包.

## Spring Boot开发者工具

```groovy
compile "org.springframework.boot:spring-boot-devtools"
```

```xml
<dependency>
    <groupId>org.springframework.boot</groupId> <artifactId>spring-boot-devtools</artifactId>
</dependency>
```

以添加开发者工具. 在激活了开发者工具后, Classpath里对文件做任何修改都会触发应用程序重启. 为了让重启速度够快, 不会修改的类(比如第三方JAR文件里的类)都加载到了基础类加载器里, 而应用程序的代码则会加载到一个单独的重启类加载器里. 检测到变更时, 只有重启类加载器重启.
