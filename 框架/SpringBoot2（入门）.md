[toc]

# SpringBoot入门

### Spring Boot简介

> 简化Spring应用开发的框架

> 整个Spring技术栈的一个大整合

> J2EE开发的一站式解决方案

- 优点：
  - 快速创建独立运行的Spring项目以及与主流框架集成
  - 使用嵌入式的Servlet容器，应用无需打成WAR包 
  - starter自动依赖与版本控制
  - 大量的自动配置，简化开发，也可修改默认值
  - 无需配置XML，无代码生成，开箱即用
  - 准生产环境的运行时应用监控
  - 与云计算的天然集成

---

### 1. 微服务

2014，Martin fowler，架构风格（服务微化）

- 一个应用应该是一组小型服务；可以通过HTTP方式互通
- 每一个功能最终都是一个可独立替换和独立升级的软件单元

### 2. 基础配置

- 配置jdk
  - JAVA_HOME：jdk根目录
  - path：jdk根目录\bin
  - classpath：.;jdk根目录\bin

- 配置Maven
  - MAVEN_HOME：maven根目录
  - path：maven根目录\bin
  - 配置Maven本地仓库：mvn根目录/conf/setting.xml【配置本地仓库（和jdk）】
  - STS配置：window->preference->maven->installations\user settings
  - IDEA配置：File->Settings->Maven

### 3. 第一个SpringBoot程序

- 目录结构resources
  - static：静态资源（js、css、图片、音频、视频）
  - templates：模板文件（模板引擎freemarker、thymeleaf；默认不支持jsp）
  - application.properties：配置文件

- spring boot内置了tomcat，并且不需要打成war再执行，可以在application.properties对端口号等服务端信息进行配置
- spring boot将各个应用/三方框架设置成了一个个stater（场景），选好引入的场景后，spring boot会将该场景所需要的所有依赖自动注入

### 4. 解读

> spring boot通过XxAutoConfiguration实现自动装配，

==解读spring boot==：查看XxAutoConfiguration及XxProperties

@SpringBootApplication是SpringBoot的主配置类，该注解包含：
- @SpringBootConfiguration，此注解包含@Configuration
  - 加了@Configuration注解的类，表示该类是一个配置类，且会自动纳入Spring容器
- @EnableAutoConfiguration，使得SpringBoot可以自动配置：可以找到@SpringBootApplication所在类的包
  - 作用：就会将该包及所有子包全部纳入spring容器【自动配置自己写的代码】
  - SpringBoot在启动时，会根据spring-boot-autoconfigure-2.0.3.RELEASE.jar中的META-INF/spring.factories找到相应的三方依赖，并将这些依赖引入项目【自动配置三方依赖】

每一个EnableAutoConfiguration都有很多条件@ConditionalOnXxx，当这些条件都满足时，此配置自动装配生效。【也可以手动修改自动装配：XxxProperties文件中的prefix.属性名=value】

> 全局配置文件中的key，来源于某个Properties文件中的prefix+属性名

- 如何知道springboot开启\禁止了哪些自动装配
  - application.properties中    debug=true
  - Positive matches列表表示spring boot自动开启的装配
  - Negative matches列表表示spring boot在此时并没有启用的自动装配


### 5. 配置文件

作用：修改spring boot默认的自动配置（约定）

默认全局配置文件：
- application.properties  【k=v】
- application.yml 
  - 【k:空格v】
  - 通过垂直对齐指定层次关系
  - 默认可以不写引号，""会将其中的转义符进行转义

yaml不是一个标记文档（xml是标记文档）

> 行内写法【k: v，[Set/Listt/数组]{map,对象类型的属性},并且[]可省{}不能省】


### 6. yaml给对象注入值

##### @ConfigurationProperties方式

注入值（yaml）
```
student:
    #name: zs
    #age: 23
    sex: true
    birthday: 2019/02/12
```

注入值（properties）【可互补】
```
student.name=zs
student.age=23
```

绑定
```
@Component  //将此JavaBean注入spring
@ConfigurationProperties(prefix="student")
public class Student{   ...     }
```

##### @value方式

```
@value("ww")
private String name;
@value("33")
private int age;
```

总结
- 两种方式可互补，但方式一的优先级较高

* | @ConfigurationProperties | @value
---|---|---
注入值 | 批量注入 | 单个注入
松散语法 | 支持 | 不支持
SpEL | 不支持 | 支持
JSR303 | 支持 | 不支持
注入复杂类型 | 支持 | 不支持


> 简单类型：8个基本类型+String+Date

> @Validated：开启jsr303数据校验的注解


### 7. @PropertySource

默认会加载application.properties\application.yml文件中的数据

@PropertySource(value={"classpath:conf.properties"})
引入非默认文件，此处加载conf.properties中的数据

@PropertySource只能加载properties，不能加载yml


### 8. @ImportResource

- spring boot自动装配/自动配置，默认将spring等配置文件自动配置好，自己编写的spring等配置文件默认不识别
- 如果要识别，则需要在spring boot主配置类上通过@ImportResource(locations={"classpath:spring.xml"})指定配置文件的路径【不推荐】

> 不推荐手写xml配置文件，一般通过注解进行配置

### 9. 配置类

通过@Configuration和@Bean编写配置类（替代掉spring.xml）
```
@Configuration
public class AppConfig{
    @Bean
    public StudentService studentService(){     //<bean id="方法名"
        StudentService stuService = new StudentService();
        
        StudentDao stuDao = new StudentDao();
        stuService.setStudentDao(stuDao);
        
        return stuService;      //<bean class="返回值"
    }
}
```

### 10. 占位符表达式

- 随机值
  - ${random.int}随机整数，等等
- 引用变量值
  - yml中name: ${student.user.name:无名},引用的是properties中的student.user.name=zl
  - 其中:xx是设置默认值


### 11. 多环境设置及切换

##### properties

默认boot会读取application.properties环境 
  
- 如果有多个，则application-环境名.properties
- 选择某一个具体环境：
  - 在application.properties中指定spring.profiles.active=环境名

##### yml

主环境
```
server:
    port: 8883
spring:
    profiles:
        active: dev
```

dev
```
server:
    port: 8884
spring:
    profiles: dev
```

test
```
server:
    port: 8885
spring:
    profiles: test
```

##### 动态切换环境

- 方式一【通过运行参数指定环境】
  - 在STS（eclipse）：Run Configuration - Argument - program Argument
  - --spring.profiles.active=环境名
- 方式二【通过运行参数指定环境】
  - 在命令行
  - java -jar 项目名.jar --spring.profiles.active=环境名
- 方式三【通过vm参数指定环境】
  - 在STS（eclipse）：Run Configuration - Argument - VM
  - -Dspring.profiles.active=环境名


### 12. 配置文件的位置

##### 项目内部的配置文件

> properties和yml中的配置相互补充，如果冲突，则properties优先级较高

spring boot默认可读取的application.properties和application.yml可以存在于以下四个地方
- file:根目录/config
- file:根目录
- classpath:根目录/config
- classpath:根目录

> 注：若某项配置冲突，则优先级从上到下递减；若不冲突则互补结合使用

配置项目名：properties文件中server.servlet.context-path=/boot


##### 项目外部的配置文件

- 在STS（eclipse）- Run Configuration - Argument - program Argument：
  - --spring.config.location=文件路径
- 如果同一个配置同时存在与内部配置文件和外部配置文件，则外部>内部

> 应用场景

项目打成jar包后补救（即不改代码，补充外部配置文件）：
- 通过命令行调用外部配置文件
  - java -jar 项目.jar --spring.config.location=文件路径


##### 项目运行参数

> 上种方法适用于修改大量配置，此种方法适用于修改个别配置

- 在STS（eclipse）- Run Configuration - Argument - program Argument：
  - --server.port=8883 --server.xxx=xxx ...
- 通过命令行
  - java -jar 项目.jar --server.port=8883


---
多个地方配置时，如果冲突，优先级：

命令参数（运行参数>调用外部的配置文件）>内部文件（properties>yaml>）


### 13. 日志

> 日志框架:UCL JUL jboss-logging logback log4j log4j2 slf4j

spring boot默认选用slf4j，logback，直接使用即可

- 日志级别：TRACE<DEBUG<INFO<WARN<ERROR<FATAL<OFF
- spring boot默认的日志级别是INFO，即只打印INFO及之后级别的信息

---

- 自定义日志级别（全局配置）
```
logging.level.主配置类包名=XXX
```

- 通过配置将日志信息存储到文件中，
```
//存储到项目根目录
logging.file=springboot.log

//存储到具体指定位置
logging.file=D:/xxx

//存储到指定文件夹中（默认文件名是spring.log）
logging.path=D:/log/
```

- 指定日志显示格式
  - 日志显示在console中
  - 日志显示在文件中

```
//日志显示在console中
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n

//日志显示在文件中
logging.pattern.file=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
```

> 默认的日志格式配置在jar包中相应包的xml文件中

### 14. spring boot处理web静态资源

创建spring boot项目：new - spring starter project - 设置（选择需要的场景）

> springboot是一个jar，因此静态资源不再是存放到webapps中

- 静态资源的存放路径通过WebMvcAutoConfiguration类-addResourceHandlers()方法指定：/webjars/
- spring boot将静态资源存入jar包中

> 以前传统web项目中，修改静态资源不需重启，但在spring boot项目中需要重启

##### 引入

引入jar，从该jar的目录结构的webjars开始：http://localhost/8080/wenjars/....

##### 自己编写

自己编写的静态资源：

- 方法一：打成jar，引入，同上【不推荐】
- 方法二：spring boot约定【推荐】

放在spring boot设置的静态资源存放目录结构中即可，此目录结构在ResourceProperties类中的CLASSPATH_RESOURCE_LOCATIONS中：
```
"classpath:/META-INF/resources/",
"classpath:/resources/",
"classpath:/static/",
"classpath:/public/"
```
在以上目录存放资源文件后，访问时不需要加前缀，直接访问即可

##### 设置欢迎页

> 解读：WebMvcAutoConfiguration类中的welcomePageHandlerMapping()-->getIndexHtml()-->location+"index.html"

- 即：任意静态资源目录中的Index.html就是欢迎页

##### 自定义网页logo

> 解读WebMvcAutoConfiguration得知：

- 将名为favicon.ico的logo放入任意静态资源目录中即可

##### 自定义静态资源目录

> Properties文件中prefix+属性

- 自定义静态资源目录后，以前默认的目录会失效

```
spring.resources.static-locations=calsspath:/res/,calsspath:/img/
```

### 15. 模板引擎thymeleaf

> spring boot默认不支持JSP

动态资源：推荐模板引擎thymeleaf，网页=模板+数据

- **引入thymeleaf**：pom中增加官网查询到的thymeleaf的依赖

> 查看ThymeleafAutoConfiguration及ThymeleafProperties得知：

- **使用Thymeleaf**，只需要将文件放入目录:"classpath:/templates/";文件后缀为:".html"

```
<p th:text="${welcome}">welcome to thymeleaf...</p>
```

- 以上，先从${welcome}中取值，有则直接显示，没有则显示中间部分welcome to thymeleaf...
- th就是替换原有thml的值，th:html属性名="${...}"

> 参见Thymeleaf官网doc文档第十章

- th:text   获取文本值  如<h1/ hello </h1，将hello渲染后显示
- th:utext  获取文本值  不转义，原样显示

> 除了$还有其他符号，具体参见第四章


### 16. spring boot整合JSP开发

> 之前不需要打war包，默认自带内置tomcat，直接通过jar即可运行

- 要整合JSP（默认不支持），需要单独配置一个外置tomcat，需要打war包

1. 新建spring boot项目【war】
2. 建立基本的web项目所需要的目录结构【webapps/WEB-INF】
3. 创建tomcat实例、部署项目

- 启动tomcat服务器时，先启动tomcat，又启动spring boot

> 原理：war包的spring boot项目，在启动服务器tomcat时，会自动调用ServletInitializer类中的configure方法，configure方法会调用spring boot主配置类，从而启动spring boot

