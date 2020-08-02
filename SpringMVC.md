# 什么是Spring MVC

Spring MVC是一个基于Java的实现了MVC设计模式的请求驱动类型的轻量级Web框架，通过把模型-视图-控制器分离，将web层进行职责解耦，把复杂的web应用分成逻辑清晰的几部分，简化开发，减少出错，方便组内开发人员之间的配合

# Spring MVC的优点

（1）可以支持各种视图技术,而不仅仅局限于JSP；

（2）与Spring框架集成（如IOC容器、AOP等）；

（3）清晰的角色分配：前端控制器(dispatcherServlet) , 请求到处理器映射（handlerMapping), 处理器适配器（HandlerAdapter), 视图解析器（ViewResolver）。

（4） 支持各种请求资源的映射策略。

# SpringMVC核心是什么

Dispatcher，分发的意思。其实就是DispatcherServlet（前端控制器）。想想也是，整个springMvc的核心部分就是这个分发器

再来看一遍springMvc的整个执行流程：

（1）请求发送至前端控制器DispatcherServlet

（2）前端控制器收到请求调用HandlerMapping处理器映射器

（3）处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器（如果有则生成）一并返回给DispatcherServlet

（4）DispatcherServlet通过HandlerAdapter处理器调用处理器

（5）执行处理器

（6）Controller执行完返回ModelAndView

（7）HandlerAdapter将ModelAndView返回给DispatcherServlet

（8）DispatcherServlet将ModelAndView传给ViewReslover视图解析器

（9）ViewReslover解析后返回具体的View

（10）DisaptcherServlet对View进行渲染视图

（11）DispatcherServlet响应用户

在整个执行流程中，DispatcherServlet起到分发、响应的作用，是整个流程的中心，而springMvc的核心也就在这里。




