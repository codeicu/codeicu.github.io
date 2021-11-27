---
layout: post
title: '@Controller,@Component,@Service,@Repository的区别'
date: 2020-05-04 17:16:48
tags: Spring
categories:
---
@Controller，@Service，@Repository都有带@Component父注解，四个注解都可以说成是Component级别的注解，Spring框架自动扫描的注解也是检测是否有Component注解标记。把普通pojo实例化到spring容器中，相当于配置文件中的<bean id="" class=""/>。这三个注解它们除了@Component的属性外还有其他的的场景应用。

## Repository
这个注解用来标识这个类是用来直接访问数据库的，dao层使用@repository注解。如@Repository(value="userDao")注解是告诉Spring，让Spring创建一个名字叫“userDao”的UserDaoImpl实例。当Service需要使用Spring创建的名字叫“userDao”的UserDaoImpl实例时，就可以使用@Resource(name = "userDao")注解告诉Spring，Spring把创建好的userDao注入给Service即可。

## Service
应用的业务逻辑通常在service层实现，因为我们一般使用@Service注解标记这个类属于业务逻辑层。如@Service("userService")注解是告诉spring，当Spring要创建UserServiceImpl的的实例时，bean的名字必须叫做"userService"，这样当Action需要使用UserServiceImpl的的实例时,就可以由Spring创建好的"userService"，然后注入给Action。

## Controller
这个注解主要告诉Spring这个类作为控制器，可以看做标记为暴露给前端的入口。@Controller 用于标记在一个类上，使用它标记的类就是一个SpringMVC Controller 对象。分发处理器将会扫描使用了该注解的类的方法。通俗来说，被Controller标记的类就是一个控制器，这个类中的方法，就是相应的动作。@RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

因此，当你的一个类被@Component所注解，那么就意味着同样可以用@Repository, @Service, @Controller来替代它，同时这些注解会具备有更多的功能，而且功能各异。

比如, @Repository早已被支持了在你的持久层作为一个标记可以去自动处理数据库操作产生的异常（译者注：因为原生的java操作数据库所产生的异常只定义了几种，但是产生数据库异常的原因却有很多种，这样对于数据库操作的报错排查造成了一定的影响；而Spring拓展了原生的持久层异常，针对不同的产生原因有了更多的异常进行描述。所以，在注解了@Repository的类上如果数据库操作中抛出了异常，就能对其进行处理，转而抛出的是翻译后的spring专属数据库异常，方便我们对异常进行排查处理）。

如果想使用自定义的组件注解，那么只要在你定义的新注解中加上@Component即可：

@Component 
@Scope("prototype")
public @interface ScheduleJob {...}
这样，所有被@ScheduleJob注解的类就都可以注入到spring容器来进行管理。我们所需要做的，就是写一些新的代码来处理这个自定义注解（译者注：可以用反射的方法），进而执行我们想要执行的工作。
