## 1.什么是Spring Cloud

Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、智能路由、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。Spring Cloud并没有重复制造轮子，它只是将各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

## SpringBoot和SpringCloud的区别

SpringBoot专注于快速方便的开发单个个体微服务。

SpringCloud是关注全局的微服务协调整理治理框架，它将SpringBoot开发的一个个单体微服务整合并管理起来，

为各个微服务之间提供，配置管理、服务发现、断路器、路由、微代理、事件总线、全局锁、决策竞选、分布式会话等等集成服务

SpringBoot可以离开SpringCloud独立使用开发项目， 但是SpringCloud离不开SpringBoot ，属于依赖的关系

SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。

## Spring Cloud 和Dubbo区别

（1）服务调用方式 dubbo是RPC springcloud Rest Api

（2）注册中心,dubbo 是zookeeper ，springcloud是eureka，也可以是zookeeper

（3）服务网关,dubbo本身没有实现，只能通过其他第三方技术整合，springcloud有Zuul路由网关，作为路由服务器，进行消费者的请求分发,springcloud支持断路器，与git完美集成配置文件支持版本控制，事物总线实现配置文件的更新与服务自动装配等等一系列的微服务架构要素。



## 设计目标与优缺点

### 设计目标

**协调各个微服务，简化分布式系统开发**

### 优缺点

微服务的框架那么多比如：dubbo、Kubernetes，为什么就要使用Spring Cloud的呢？

优点：

产出于Spring大家族，Spring在企业级开发框架中无人能敌，来头很大，可以保证后续的更新、完善
组件丰富，功能齐全。Spring Cloud 为微服务架构提供了非常完整的支持。例如、配置管理、服务发现、断路器、微服务网关等；
Spring Cloud 社区活跃度很高，教程很丰富，遇到问题很容易找到解决方案
服务拆分粒度更细，耦合度比较低，有利于资源重复利用，有利于提高开发效率
可以更精准的制定优化服务方案，提高系统的可维护性
减轻团队的成本，可以并行开发，不用关注其他人怎么开发，先关注自己的开发
微服务可以是跨平台的，可以用任何一种语言开发
适于互联网时代，产品迭代周期更短

缺点：

微服务过多，治理成本高，不利于维护系统
分布式系统开发的成本高（容错，分布式事务等）对团队挑战大
总的来说优点大过于缺点，目前看来Spring Cloud是一套非常完善的分布式框架，目前很多企业开始用微服务、Spring Cloud的优势是显而易见的。因此对于想研究微服务架构的同学来说，学习Spring Cloud是一个不错的选择。

## Spring Cloud整体架构

<img src="https://img-blog.csdnimg.cn/20191226143921760.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly90aGlua3dvbi5ibG9nLmNzZG4ubmV0,size_16,color_FFFFFF,t_70" alt="img" style="zoom: 67%;" />

## Spring Cloud Config

集中配置管理工具，分布式系统中统一的外部配置管理，默认使用Git来存储配置，可以支持客户端配置的刷新及加密、解密操作。

## Spring Cloud Netflix

Netflix OSS 开源组件集成，包括Eureka、Hystrix、Ribbon、Feign、Zuul等核心组件。

### 1.Eureka

服务治理组件，包括服务端的注册中心和客户端的服务发现机制；

### 2.Ribbon

负载均衡的服务调用组件，具有多种负载均衡调用策略；

### 3.Hystrix

服务容错组件，实现了断路器模式，为依赖服务的出错和延迟提供了容错能力；

#### 如何实现容错

### 4.Feign

基于Ribbon和Hystrix的声明式服务调用组件；

### 5.Zuul

API网关组件，对请求提供路由及过滤功能。



## Spring Cloud Bus

用于传播集群状态变化的消息总线，使用轻量级消息代理链接分布式系统中的节点，可以用来动态刷新集群中的服务配置。

## Spring Cloud Consul

基于Hashicorp Consul的服务治理组件

## Spring Cloud Security

安全工具包，对Zuul代理中的负载均衡OAuth2客户端及登录认证进行支持

## Spring Cloud Sleuth

Spring Cloud应用程序的分布式请求链路跟踪，支持使用Zipkin、HTrace和基于日志（例如ELK）的跟踪

## Spring Cloud Stream

轻量级事件驱动微服务框架，可以使用简单的声明式模型来发送及接收消息，主要实现为Apache Kafka及RabbitMQ

## Spring Cloud Task

用于快速构建短暂、有限数据处理任务的微服务框架，用于向应用中添加功能性和非功能性的特性。

## Spring Cloud Zookeeper

基于Apache Zookeeper的服务治理组件

## Spring Cloud Gateway

API网关组件，对请求提供路由及过滤功能

## Spring Cloud OpenFeign

基于Ribbon和Hystrix的声明式服务调用组件，可以动态创建基于Spring MVC注解的接口实现用于服务调用，在Spring Cloud 2.0中已经取代Feign成为了一等公民

## Dubbo

### RPC调用接口超时

DUBBO消费端设置额超时时间不能随心所欲，需要根据业务实际情况来设定，如果设置的时间太短，导致复杂业务本来就需要很长时间完成，导致在设定的超时时间内无法完成正常的业务处理。如果消费端达到超时时间，那么dubbo会进行重试机制（如果配置了dubbo.reference.retries>1），这种情况其实给服务提供端带来莫名的压力，而压力是正常值*dubbo.reference.retries，最终dubbo的消费端会出现RpcException提示retry了多少次还是失败。这种情况就是没有合理设置接口超时时间带来的问题。 

说完超时时间，再说说重试机制。重试机制是在等待超时时间到了之后或者服务提供端出现异常进行再次重试的机制。这个并不代表服务提供端完全执行失败了。所以不是所有接口都适合重试，如果一个服务是不等幂，那么不适合重试的机制，因为会存在重复提交的问题，否则是可以进行重试的。比如提交一个订单的接口是不能进行重试的，而查询类型的接口是可以重试的

<!--好像还有什么服务降级什么的-->

### dubbo的调用链路

服务暴露，monotor,contain,容器，与bean容器整合





