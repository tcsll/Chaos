# 1.什么是spring

Spring是一个轻量级Java开发框架，最早有Rod Johnson创建，目的是为了解决企业级应用开发的业务逻辑层和其他各层的耦合问题。它是一个分层的JavaSE/JavaEE full-stack（一站式）轻量级开源框架，为开发Java应用程序提供全面的基础架构支持。Spring负责基础架构，因此Java开发者可以专注于应用程序的开发。

Spring最根本的使命是解决企业级应用开发的复杂性，即简化Java开发。

Spring可以做很多事情，它为企业级开发提供给了丰富的功能，但是这些功能的底层都依赖于它的两个核心特性，也就是依赖注入（dependency injection，DI）和面向切面编程（aspect-oriented programming，AOP）。

为了降低Java开发的复杂性，Spring采取了以下4种关键策略

基于POJO的轻量级和最小侵入性编程；
通过依赖注入和面向接口实现松耦合；
基于切面和惯例进行声明式编程；
通过切面和模板减少样板式代码。





## Spring bean的注入过程

![img](https://upload-images.jianshu.io/upload_images/460263-74d88a767a80843a.png?imageMogr2/auto-orient/strip|imageView2/2)



# Spring Bean的作用域

<img src="https://img2018.cnblogs.com/blog/816787/201906/816787-20190601153119575-186640670.png" alt="img" style="zoom:80%;" />





## IOC

<!--马勒戈壁老子不草死你-->

### 1.IO 理论

IoC 全称为 `Inversion of Control`，翻译为 “控制反转”，它还有一个别名为 DI（`Dependency Injection`）,即依赖注入。

如何理解“控制反转”好呢？理解好它的关键在于我们需要回答如下四个问题：

1. 谁控制谁
2. 控制什么
3. 为何是反转
4. 哪些方面反转了

在回答这四个问题之前，我们先看 IOC 的定义：

> **所谓 IOC ，就是由 Spring IOC 容器来负责对象的生命周期和对象之间的关系**

上面这句话是整个 IoC 理论的核心。如何来理解这句话？我们引用一个例子来走阐述（看完该例子上面四个问题也就不是问题了）。

以找女朋友为例（对于程序猿来说这个值得探究的问题）。一般情况下我们是如何来找女朋友的呢？首先我们需要根据自己的需求（漂亮、身材好、性格好）找一个妹子，然后到处打听她的兴趣爱好、微信、电话号码，然后各种投其所好送其所要，最后追到手。如下：

```java
/**
 * 年轻小伙子
 */
public class YoungMan {
    private BeautifulGirl beautifulGirl;

    YoungMan(){
        // 可能你比较牛逼，指腹为婚
        // beautifulGirl = new BeautifulGirl();
    }

    public void setBeautifulGirl(BeautifulGirl beautifulGirl) {
        this.beautifulGirl = beautifulGirl;
    }

    public static void main(String[] args){
        YoungMan you = new YoungMan();
        BeautifulGirl beautifulGirl = new BeautifulGirl("你的各种条件");
        beautifulGirl.setxxx("各种投其所好");

        // 然后你有女票了
        you.setBeautifulGirl(beautifulGirl);
    }
}
```

这就是我们通常做事的方式，如果我们需要某个对象，一般都是采用这种直接创建的方式(`new BeautifulGirl()`)，这个过程复杂而又繁琐，而且我们必须要面对每个环节，同时使用完成之后我们还要负责销毁它，在这种情况下我们的对象与它所依赖的对象耦合在一起。

其实我们需要思考一个问题？我们每次用到自己依赖的对象真的需要自己去创建吗？我们知道，我们依赖对象其实并不是依赖该对象本身，而是依赖它所提供的服务，只要在我们需要它的时候，它能够及时提供服务即可，至于它是我们主动去创建的还是别人送给我们的，其实并不是那么重要。再说了，相比于自己千辛万苦去创建它还要管理、善后而言，直接有人送过来是不是显得更加好呢？

这个给我们送东西的“人” 就是 IOC，在上面的例子中，它就相当于一个婚介公司，作为一个婚介公司它管理着很多男男女女的资料，当我们需要一个女朋友的时候，直接跟婚介公司提出我们的需求，婚介公司则会根据我们的需求提供一个妹子给我们，我们只需要负责谈恋爱，生猴子就行了。你看，这样是不是很简单明了。

诚然，作为婚介公司的 IOC 帮我们省略了找女朋友的繁杂过程，将原来的主动寻找变成了现在的被动接受（符合我们的要求），更加简洁轻便。你想啊，原来你还得鞍马前后，各种巴结，什么东西都需要自己去亲力亲为，现在好了，直接有人把现成的送过来，多么美妙的事情啊。所以，简单点说，IOC 的理念就是**让别人为你服务**

在没有引入 IOC 的时候，被注入的对象直接依赖于被依赖的对象，有了 IOC 后，两者及其他们的关系都是通过 Ioc Service Provider 来统一管理维护的。被注入的对象需要什么，直接跟 Ioc Service Provider 打声招呼，后者就会把相应的被依赖对象注入到被注入的对象中，从而达到 IOC Service Provider 为被注入对象服务的目的。**所以 IoC 就是这么简单！原来是需要什么东西自己去拿，现在是需要什么东西让别人（IOC Service Provider）送过来**

现在在看上面那四个问题，答案就显得非常明显了:

1. **谁控制谁**：在传统的开发模式下，我们都是采用直接 new 一个对象的方式来创建对象，也就是说你依赖的对象直接由你自己控制，但是有了 IOC 容器后，则直接由 IoC 容器来控制。所以“谁控制谁”，当然是 IoC 容器控制对象。
2. **控制什么**：控制对象。
3. **为何是反转**：没有 IoC 的时候我们都是在自己对象中主动去创建被依赖的对象，这是正转。但是有了 IoC 后，所依赖的对象直接由 IoC 容器创建后注入到被注入的对象中，依赖的对象由原来的主动获取变成被动接受，所以是反转。
4. **哪些方面反转了**：所依赖对象的获取被反转了。

妹子有了，但是如何拥有妹子呢？这也是一门学问。

1. 可能你比较牛逼，刚刚出生的时候就指腹为婚了。
2. 大多数情况我们还是会考虑自己想要什么样的妹子，所以还是需要向婚介公司打招呼的。
3. 还有一种情况就是，你根本就不知道自己想要什么样的妹子，直接跟婚介公司说，我就要一个这样的妹子。

所以，IOC Service Provider 为被注入对象提供被依赖对象也有如下几种方式：**构造方法注入、stter方法注入、接口注入**

**构造器注入**

构造器注入，顾名思义就是被注入的对象通过在其构造方法中声明依赖对象的参数列表，让外部知道它需要哪些依赖对象。

```java
YoungMan(BeautifulGirl beautifulGirl){
        this.beautifulGirl = beautifulGirl;
}
```

构造器注入方式比较直观，对象构造完毕后就可以直接使用，这就好比你出生你家里就给你指定了你媳妇。

**setter 方法注入**

对于 JavaBean 对象而言，我们一般都是通过 getter 和 setter 方法来访问和设置对象的属性。所以，当前对象只需要为其所依赖的对象提供相对应的 setter 方法，就可以通过该方法将相应的依赖对象设置到被注入对象中。如下：

```java
public class YoungMan {
    private BeautifulGirl beautifulGirl;

    public void setBeautifulGirl(BeautifulGirl beautifulGirl) {
        this.beautifulGirl = beautifulGirl;
    }
}
```

相比于构造器注入，setter 方式注入会显得比较宽松灵活些，它可以在任何时候进行注入（当然是在使用依赖对象之前），这就好比你可以先把自己想要的妹子想好了，然后再跟婚介公司打招呼，你可以要林志玲款式的，赵丽颖款式的，甚至凤姐哪款的，随意性较强。

**接口方式注入**

接口方式注入显得比较霸道，因为它需要被依赖的对象实现不必要的接口，带有侵入性。一般都不推荐这种方式。

------

关于 IOC 理论部分，笔者不在阐述，这里推荐几篇博客阅读：

- [谈谈对Spring IOC的理解](http://www.cnblogs.com/xdp-gacl/p/4249939.html)：http://www.cnblogs.com/xdp-gacl/p/4249939.html
- [Spring的IOC原理[通俗解释一下\]](https://blog.csdn.net/m13666368773/article/details/7802126)：https://blog.csdn.net/m13666368773/article/details/7802126
- [spring ioc原理（看完后大家可以自己写一个spring）](https://blog.csdn.net/it_man/article/details/4402245)：https://blog.csdn.net/it_man/article/details/4402245



### 2.各个组件

该图为 ClassPathXmlApplicationContext 的类继承体系结构，虽然只有一部分，但是它基本上包含了 IOC 体系中大部分的核心类和接口

![img](http://dl.iteye.com/upload/attachment/558072/d9a0e8c8-29d6-36a3-888e-e22c2d1bb44f.jpg)

下面我们就针对这个图进行简单的拆分和补充说明。

**Resource体系**

Resource，对资源的抽象，它的每一个实现类都代表了一种资源的访问策略，如ClasspathResource 、 URLResource ，FileSystemResource 等。

有了资源，就应该有资源加载，Spring 利用 ResourceLoader 来进行统一资源加载，类图如下：

**BeanFactory 体系**

BeanFactory 是一个非常纯粹的 bean 容器，它是 IOC 必备的数据结构，其中 BeanDefinition 是她的基本结构，它内部维护着一个 BeanDefinition map ，并可根据 BeanDefinition 的描述进行 bean 的创建和管理。

BeanFacoty 有三个直接子类 `ListableBeanFactory`、`HierarchicalBeanFactory` 和 `AutowireCapableBeanFactory`，`DefaultListableBeanFactory` 为最终默认实现，它实现了所有接口。

**Beandefinition 体系**

BeanDefinition 用来描述 Spring 中的 Bean 对象。

**BeandefinitionReader体系**

BeanDefinitionReader 的作用是读取 Spring 的配置文件的内容，并将其转换成 Ioc 容器内部的数据结构：BeanDefinition。

**ApplicationContext体系**

这个就是大名鼎鼎的 Spring 容器，它叫做应用上下文，与我们应用息息相关，她继承 BeanFactory，所以它是 BeanFactory 的扩展升级版，如果BeanFactory 是屌丝的话，那么 ApplicationContext 则是名副其实的高富帅。由于 ApplicationContext 的结构就决定了它与 BeanFactory 的不同，其主要区别有：

1. 继承 MessageSource，提供国际化的标准访问策略。
2. 继承 ApplicationEventPublisher ，提供强大的事件机制。
3. 扩展 ResourceLoader，可以用来加载多个 Resource，可以灵活访问不同的资源。
4. 对 Web 应用的支持。

![img](https://img-blog.csdn.net/20150724162847886)

上面五个体系可以说是 Spring IoC 中最核心的部分，后面博文也是针对这五个部分进行源码分析。其实 IOC 咋一看还是挺简单的，无非就是将配置文件（暂且认为是 xml 文件）进行解析（分析 xml 谁不会啊），然后放到一个 Map 里面就差不多了，初看有道理，其实要面临的问题还是有很多的，后面可能还需要深入的进行研究。









从该继承体系可以看出：

BeanFactory 是一个 bean 工厂的最基本定义，里面包含了一个 bean 工厂的几个最基本的方法，getBean(…) 、 containsBean(…) 等 ,是一个很纯粹的bean工厂，不关注资源、资源位置、事件等

ApplicationContext 是一个容器的最基本接口定义，它继承了 BeanFactory, 拥有工厂的基本方法。同时继承了 ApplicationEventPublisher 、 MessageSource 、 ResourcePatternResolver 等接口，使其 定义了一些额外的功能，如资源、事件等这些额外的功能。

AbstractBeanFactory 和 AbstractAutowireCapableBeanFactory 是两个模板抽象工厂类。

AbstractBeanFactory 提供了 bean 工厂的抽象基类，同时提供了 ConfigurableBeanFactory 的完整实现。 AbstractAutowireCapableBeanFactory 是继承了 AbstractBeanFactory 的抽象工厂，里面提供了bean 创建的支持，包括 bean 的创建、依赖注入、检查等等功能，是一个核心的 bean 工厂基类。

ClassPathXmlApplicationContext之所以拥有 bean 工厂的功能是通过持有一个真正的 bean 工厂DefaultListableBeanFactory 的实例，并通过 代理 该工厂完成。

 ClassPathXmlApplicationContext 的初始化过程是对本身容器的初始化同时也是对其持有的DefaultListableBeanFactory 的初始化。



## 容器初始化过程

![img](http://dl.iteye.com/upload/attachment/558065/370101b5-b3d1-3b79-b9b2-3bba91d092d7.jpg)

整个过程可以理解为是容器的初始化过程。

第一个过程是 ApplicationContext 的职责范围，

第二步是BeanFactory 的职责范围。可以看出 ApplicationContext 是一个运行时的容器需要提供不容资源环境的支持，屏蔽不同环境的差异化。而 BeanDifinition 是内部关于 bean 定义的基本结构。 Bean 的创建就是基于它，回头会介绍一下改结构的定义。下面看一下整个容器的初始化过程。

容器的初始化是通过调用 refresh() 来实现。该方法是非常重要的一个方法，定义在AbstractApplicationContext 接口里。 AbstractApplicationContext 是容器的最基础的一个抽象父类。也就是说在该里面定义了一个容器初始化的基本流程，流程里的各个方法有些有提供了具体实现，有些是抽象的( 因为不同的容器实例不一样 ) ，由继承它的每一个具体容器完成定制。看看 refresh 的基本流程：

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();
			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);
			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				// Initialize message source for this context.
				initMessageSource();
				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();
				// Initialize other special beans in specific context subclasses.
				onRefresh();
				// Check for listener beans and register them.
				registerListeners();
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);
				// Last step: publish corresponding event.
				finishRefresh();
			}
			catch (BeansException ex) {
				// Destroy already created singletons to avoid dangling resources.
				beanFactory.destroySingletons();
				// Reset 'active' flag.
				cancelRefresh(ex);
				// Propagate exception to caller.
				throw ex;
			}
		}
	}

```



## AOP

### 1.理论

OOP(Object-Oriented Programming)面向对象编程，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

AOP(Aspect-Oriented Programming)，一般称为面向切面编程，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为“切面”（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理等。

### Spring AOP and AspectJ AOP 有什么区别？AOP 有哪些实现方式？

AOP实现的关键在于 代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ；动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

### JDK动态代理和CGLIB动态代理的区别

Spring AOP中的动态代理主要有两种方式，JDK动态代理和CGLIB动态代理：

JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用 InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

InvocationHandler 的 invoke(Object proxy,Method method,Object[] args)：proxy是最终生成的代理实例; method 是被代理目标实例的某个具体方法; args 是被代理目标实例某个方法的具体入参, 在方法反射调用时使用。



# 循环依赖

![image-20200622205721465](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20200622205721465.png)

spring bean的创建分为两个部分  一个是创建对象 一个是设置对象属性
循环依赖就是
spring通过ApplicationContext.getBean()获取A
但是发现A没有存在就先创建A
然后要设置属性B  通过ApplicationContext.getBean() 获取B  然后发现没有B 然后创建B
B要设置A的属性  又通过ApplicationContext.getBean() 获取到A  因为A已经被创建出来 虽然没有数据
获取A设置完之后B就准备好了  然后B就又会返回到A中 
最后两个对象都创建成功

对于被创建出来但是没有设置属性值的对象可以称之为半成品

Spring在实例化一个bean的时候，是首先递归的实例化其所依赖的所有bean，直到某个bean没有依赖其他bean，此时就会将该实例返回，然后反递归的将获取到的bean设置为各个上层bean的属性的。
简称为套娃




## Spring事务

![img](https://upload-images.jianshu.io/upload_images/3493621-6c787fc2f9aed053.jpg?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

而在Spring的容器内,调用者实际上是通过调用Spring的事务接口来实现事务的管理.

![img](https://upload-images.jianshu.io/upload_images/3493621-7d0f2a1b6f9b9b7c.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

### Spring框架的事务管理有哪些优点

- 为不同的事务API 如 JTA，JDBC，Hibernate，JPA 和JDO，提供一个不变的编程模式
- 为编程式事务管理提供了一套简单的API而不是一些复杂的事务API
- 支持声明式事务管理
- 和Spring各种数据访问抽象层很好得集成

### Spring支持的事务管理类型

Spring支持两种类型的事务管理：

**编程式事务管理**：这意味你通过编程的方式管理事务，给你带来极大的灵活性，但是难维护。

**声明式事务管理**：这意味着你可以将业务代码和事务管理分离，你只需用注解和XML配置来管理事务

### 你更倾向用那种事务管理类型

大多数Spring框架的用户选择声明式事务管理，因为它对应用代码的影响最小，因此更符合一个无侵入的轻量级容器的思想。声明式事务管理要优于编程式事务管理，虽然比编程式事务管理（这种方式允许你通过代码控制事务）少了一点灵活性。唯一不足地方是，最细粒度只能作用到方法级别，无法做到像编程式事务那样可以作用到代码块级别。

### Spring 事务实现方式/原理

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。真正的数据库层的事务提交和回滚是通过binlog或者redo log实现的。

### Spring的7个事务传播行为

1. PROPAGATION_REQUIRED：(required)

   如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的

2. PROPAGATION_SUPPORTS：(supports)

   支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行

3. PROPAGATION_MANDATORY：(mandatory)

   支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常

4. PROPAGATION_REQUIRES_NEW：(new)

   创建新事务，无论当前存不存在事务，都创建新事务

5. PROPAGATION_NOT_SUPPORTED：(supported)

   以非事务方式执行操作，如果当前存在事务，就把当前事务挂起

6. PROPAGATION_NEVER：(never)

   以非事务方式执行，如果当前存在事务，则抛出异常

7. PROPAGATION_NESTED：(nested)

   如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行

### Spring的5大事务隔离级别

spring 有五大隔离级别，默认值为 ISOLATION_DEFAULT（使用数据库的设置），其他四个隔离级别和数据库的隔离级别一致：

1. ISOLATION_DEFAULT：(default)

   用底层数据库的设置隔离级别，数据库设置的是什么我就用什么

2. ISOLATION_READ_UNCOMMITTED：(read_uncommitted)

   未提交读，最低隔离级别、事务未提交前，就可被其他事务读取（会出现幻读、脏读、不可重复读）

3. ISOLATION_READ_COMMITTED：(read_committed)

   提交读，一个事务提交后才能被其他事务读取到（会造成幻读、不可重复读），SQL server 的默认级别

4. ISOLATION_REPEATABLE_READ：(repeatable)

   可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据（会造成幻读），MySQL 的默认级别；

5. ISOLATION_SERIALIZABLE：(serializable)

   序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。

> 脏读 ：表示一个事务能够读取另一个事务中还未提交的数据。比如，某个事务尝试插入记录 A，此时该事务还未提交，然后另一个事务尝试读取到了记录 A。
>
> 不可重复读 ：是指在一个事务内，多次读同一数据但是结果不一致。
>
> 幻读 ：指同一个事务内多次查询返回的结果集不一样。比如同一个事务 A 第一次查询时候有 n 条记录，但是第二次同等条件下查询却有 n+1 条记录，这就好像产生了幻觉。发生幻读的原因也是另外一个事务新增或者删除或者修改了第一个事务结果集里面的数据，同一个记录的数据内容被修改了，所有数据行的记录就变多或者变少了。
> 



