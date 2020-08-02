## 什么是 Spring Boot？



## Starter

### starer是什么

### 怎么实现starter

Spring Boot 有哪些优点？

Spring Boot 的核心注解是哪个？它主要由哪几个注解组成的？

什么是 JavaConfig

Spring Boot 自动配置原理是什么

你如何理解 Spring Boot 配置加载顺序









7.YAML 配置

8.Spring Boot 是否可以使用 XML 配置 

9.spring boot 核心配置文件是什么

### 什么是 Spring Profiles

### 如何在自定义端口上运行 Spring Boot 应用程序？

### 如何实现 Spring Boot 应用程序的安全性

### 比较一下 Spring Security 和 Shiro 各自的优缺点 

### Spring Boot 中如何解决跨域问题

### 什么是 CSRF 攻击

## 监视器

### Spring Boot 中的监视器是什么

### 如何在 Spring Boot 中禁用 Actuator 端点安全性

### 我们如何监视所有 Spring Boot 微服务





如何重新加载 Spring Boot 上的更改，而无需重新启动服务器？Spring Boot项目如何热部署？
这可以使用 DEV 工具来实现。通过这种依赖关系，您可以节省任何更改，嵌入式tomcat 将重新启动。Spring Boot 有一个开发工具（DevTools）模块，它有助于提高开发人员的生产力。Java 开发人员面临的一个主要挑战是将文件更改自动部署到服务器并自动重启服务器。开发人员可以重新加载 Spring Boot 上的更改，而无需重新启动服务器。这将消除每次手动部署更改的需要。Spring Boot 在发布它的第一个版本时没有这个功能。这是开发人员最需要的功能。DevTools 模块完全满足开发人员的需求。该模块将在生产环境中被禁用。它还提供 H2 数据库控制台以更好地测试应用程序。

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
</dependency>
1
2
3
4
您使用了哪些 starter maven 依赖项？
使用了下面的一些依赖项

spring-boot-starter-activemq

spring-boot-starter-security

这有助于增加更少的依赖关系，并减少版本的冲突。

Spring Boot 中的 starter 到底是什么 ?
首先，这个 Starter 并非什么新的技术点，基本上还是基于 Spring 已有功能来实现的。首先它提供了一个自动化配置类，一般命名为 XXXAutoConfiguration ，在这个配置类中通过条件注解来决定一个配置是否生效（条件注解就是 Spring 中原本就有的），然后它还会提供一系列的默认配置，也允许开发者根据实际情况自定义相关配置，然后通过类型安全的属性注入将这些配置属性注入进来，新注入的属性会代替掉默认属性。正因为如此，很多第三方框架，我们只需要引入依赖就可以直接使用了。当然，开发者也可以自定义 Starter

spring-boot-starter-parent 有什么用 ?
我们都知道，新创建一个 Spring Boot 项目，默认都是有 parent 的，这个 parent 就是 spring-boot-starter-parent ，spring-boot-starter-parent 主要有如下作用：

定义了 Java 编译版本为 1.8 。
使用 UTF-8 格式编码。
继承自 spring-boot-dependencies，这个里边定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号。
执行打包操作的配置。
自动化的资源过滤。
自动化的插件配置。
针对 application.properties 和 application.yml 的资源过滤，包括通过 profile 定义的不同环境的配置文件，例如 application-dev.properties 和 application-dev.yml。
Spring Boot 打成的 jar 和普通的 jar 有什么区别 ?
Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过 java -jar xxx.jar 命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。

Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes 目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。

运行 Spring Boot 有哪几种方式？
1）打包用命令或者放到容器中运行

2）用 Maven/ Gradle 插件运行

3）直接执行 main 方法运行

Spring Boot 需要独立的容器运行吗？
可以不需要，内置了 Tomcat/ Jetty 等容器。

开启 Spring Boot 特性有哪几种方式？
1）继承spring-boot-starter-parent项目

2）导入spring-boot-dependencies项目依赖

如何使用 Spring Boot 实现异常处理？
Spring 提供了一种使用 ControllerAdvice 处理异常的非常有用的方法。 我们通过实现一个 ControlerAdvice 类，来处理控制器类抛出的所有异常。

如何使用 Spring Boot 实现分页和排序？
使用 Spring Boot 实现分页非常简单。使用 Spring Data-JPA 可以实现将可分页的传递给存储库方法。

微服务中如何实现 session 共享 ?
在微服务中，一个完整的项目被拆分成多个不相同的独立的服务，各个服务独立部署在不同的服务器上，各自的 session 被从物理空间上隔离开了，但是经常，我们需要在不同微服务之间共享 session ，常见的方案就是 Spring Session + Redis 来实现 session 共享。将所有微服务的 session 统一保存在 Redis 上，当各个微服务对 session 有相关的读写操作时，都去操作 Redis 上的 session 。这样就实现了 session 共享，Spring Session 基于 Spring 中的代理过滤器实现，使得 session 的同步操作对开发人员而言是透明的，非常简便。

Spring Boot 中如何实现定时任务 ?
定时任务也是一个常见的需求，Spring Boot 中对于定时任务的支持主要还是来自 Spring 框架。

在 Spring Boot 中使用定时任务主要有两种不同的方式，一个就是使用 Spring 中的 @Scheduled 注解，另一个则是使用第三方框架 Quartz。

使用 Spring 中的 @Scheduled 的方式主要通过 @Scheduled 注解来实现。

使用 Quartz ，则按照 Quartz 的方式，定义 Job 和 Trigger 即可。

## 实现自定义注解

自定义注解类编写的一些规则:
1. Annotation型定义为@interface, 所有的Annotation会自动继承java.lang.Annotation这一接口,并且不能再去继承别的类或是接口.
2. 参数成员只能用public或默认(default)这两个访问权修饰
3. 参数成员只能用基本类型byte,short,char,int,long,float,double,boolean八种基本数据类型和String、Enum、Class、annotations等数据类型,以及这一些类型的数组.
4. 要获取类方法和字段的注解信息，必须通过Java的反射技术来获取 Annotation对象,因为你除此之外没有别的获取注解对象的方法
5. 注解也可以没有定义成员, 不过这样注解就没啥用了

### 例如aop日志处理

#### 1.引入pom包

主要原理还是利用AOP的原理，因此需要在项目中定义一个切面类，需要使用到@Aspect注解，在pom.xml中添加jar包：spring-boot-starter-aop

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/20190109212815152.png)

#### 2.常用三个注解

```java
/**
* 指明了修饰的这个注解的使用范围，即被描述的注解可以用在哪里
* ElementType的取值包含以下几种：
* TYPE:类，接口或者枚举
* FIELD:域，包含枚举常量
* METHOD:方法
* PARAMETER:参数
* CONSTRUCTOR:构造方法
* LOCAL_VARIABLE:局部变量
* ANNOTATION_TYPE:注解类型
* PACKAGE:包
*/
@Target(ElementType.METHOD)
/**
* 指明修饰的注解的生存周期，即会保留到哪个阶段
* RetentionPolicy的取值包含以下三种：
* SOURCE：源码级别保留，编译后即丢弃。
* CLASS:编译级别保留，编译后的class文件中存在，在jvm运行时丢弃，这是默认值。
* RUNTIME： 运行级别保留，编译后的class文件中存在，在jvm运行时保留，可以被反射调用
*/
@Retention(RetentionPolicy.RUNTIME)
//指明修饰的注解，可以被例如javadoc此类的工具文档化，只负责标记，没有成员取值。
@Documented
```

#### 3.服务层

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 服务层日志注解,加载方法上
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SystemServicesLog {
    String value();
}

```

#### 4.控制层

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 控制层日志注解，添加在方法上
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SystemControllerLog {
    String value();
}
```

####  5.切面类的实现

```java
import com.mp.web.annotation.SystemControllerLog;
import com.mp.web.annotation.SystemServicesLog;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.stereotype.Component;

/**配置日志切面类，@Aspect把当前类标识为一个切面类工容器读取*/
@Aspect
@Component
public class MpLogAop {

    /**
     * 使用后置通知，并配置切入点表达式，联合自定义控制层日志注解，记录日志
     * @param joinPoint
     * @param systemControllerLog
     */
    @After("execution(* com.mp.web.controller.*.*(..)) && @annotation(systemControllerLog)")
    public void afterControllerLog(JoinPoint joinPoint,SystemControllerLog systemControllerLog){
        String logValue = systemControllerLog.value();
        System.out.println("后置通知开始执行==》控制层日志log==》用户执行了【"+logValue+"】操作");
    }

    /**
     * 使用后置通知以及自定义服务层日志注解，记录服务层操作日志
     * @param joinPoint
     * @param systemServicesLog
     */
    @After("execution(* com.mp.web.services.*.*(..)) && @annotation(systemServicesLog)")
    public void afterServiceLog(JoinPoint joinPoint,SystemServicesLog systemServicesLog){
        String logValue = systemServicesLog.value();
        System.out.println("后置通知开始执行==》服务层日志log==》用户执行了【"+logValue+"】操作");
    }
}
```

#### 6.添加自定义注解

![å¨è¿éæå¥å¾çæè¿°](https://img-blog.csdnimg.cn/2019010921365081.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L015X25hbWVfaXNfRg==,size_16,color_FFFFFF,t_70)

