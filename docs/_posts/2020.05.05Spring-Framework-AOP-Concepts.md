---
title: Spring_Framework_AOP_Concepts
date: 2020-05-05 22:05:17
tags: Spring
categories:
---

学习一下Spring FrameWork的官方文档.
为什么要看官方文档呢?

1. 全面: 每一个知识点, 用法以及最佳实践, 意外情况等等都包括其中. 即使没有展开讲述, 一般也会给出参考文档或链接. 
2. 准确: 从作者的角度出发, 不仅有准确的描述, 还有设计思想, 理念, 原则等等, 让人对技术的理解更加深入. 
3. 明晰: 官方文档结构都很简洁清晰, 读来令人舒畅.
4. 地道: 原汁原味, 这些技术文档本身也是非常棒的英语学习材料.

就像教科书一样, 虽然对于做题来说没有辅导材料来的快,来的直接, 但读多了就是极好的基础, 能解决许多别人没有提到的问题.

# AOP Concepts

Let us begin by defining some central AOP concepts and terminology. These terms are not Spring-specific. Unfortunately, AOP terminology is not particularly intuitive(直观). However, it would be even more confusing if Spring used its own terminology.

- Aspect: A modularization of a concern that cuts across multiple classes. Transaction management is a good example of a crosscutting concern in enterprise Java applications. In Spring AOP, aspects are implemented by using regular classes or regular classes annotated with the @Aspect annotation.

- Join point: A point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.

- Advice: Action taken by an aspect at a particular join point. Different types of advice include “around”, “before” and “after” advice. (Advice types are discussed later.) Many AOP frameworks, including Spring, model an advice as an interceptor and maintain a chain of interceptors around the join point.

- Pointcut: A predicate that matches join points. Advice is associated with a pointcut expression and runs at any join point matched by the pointcut (for example, the execution of a method with a certain name). The concept of join points as matched by pointcut expressions is central to AOP, and Spring uses the AspectJ pointcut expression language by default.

- Introduction: Declaring additional methods or fields on behalf of a type. Spring AOP lets you introduce new interfaces (and a corresponding implementation) to any advised object. For example, you could use an introduction to make a bean implement an IsModified interface, to simplify caching. (An introduction is known as an inter-type declaration in the AspectJ community.)

- Target object: An object being advised by one or more aspects. Also referred to as the “advised object”. Since Spring AOP is implemented by using runtime proxies(代理人), this object is always a proxied object.

- AOP proxy: An object created by the AOP framework in order to implement the aspect contracts (advise method executions and so on). In the Spring Framework, an AOP proxy is a JDK dynamic proxy or a CGLIB proxy.

- Weaving: linking aspects with other application types or objects to create an advised object. This can be done at compile time (using the AspectJ compiler, for example), load time, or at runtime. Spring AOP, like other pure Java AOP frameworks, performs weaving at runtime.

Spring AOP includes the following types of advice:

- Before advice: Advice that runs before a join point but that does not have the ability to prevent execution flow proceeding to the join point (unless it throws an exception).

- After returning advice: Advice to be run after a join point completes normally (for example, if a method returns without throwing an exception).

- After throwing advice: Advice to be executed if a method exits by throwing an exception.

- After (finally) advice: Advice to be executed regardless of the means by which a join point exits (normal or exceptional return).

- Around advice: Advice that surrounds a join point such as a method invocation. This is the most powerful kind of advice. Around advice can perform custom behavior before and after the method invocation. It is also responsible for choosing whether to proceed to the join point or to shortcut the advised method execution by returning its own return value or throwing an exception.

(权限约束, 功能明晰, 如果滥用Around, 可能会fail to invoke it)
Around advice is the most general kind of advice. Since Spring AOP, like AspectJ, provides a full range of advice types, we recommend that you use the least powerful advice type that can implement the required behavior. For example, if you need only to update a cache with the return value of a method, you are better off implementing an after returning advice than an around advice, although an around advice can accomplish the same thing. Using the most specific advice type provides a simpler programming model with less potential for errors. For example, you do not need to invoke the proceed() method on the JoinPoint used for around advice, and, hence, you cannot fail to invoke it.

All advice parameters are statically typed so that you work with advice parameters of the appropriate type (e.g. the type of the return value from a method execution) rather than Object arrays.

The concept of join points matched by pointcuts is the key to AOP, which distinguishes it from older technologies offering only interception. Pointcuts enable advice to be targeted independently of the object-oriented hierarchy. For example, you can apply an around advice providing declarative transaction management to a set of methods that span multiple objects (such as all business operations in the service layer).


## Introduction

Adivce都是在目标方法范围内织入, 而Introduction实在类级别添加目标未实现的接口方法.
```java
public interface Auto{
    void driving();
}
public class MyCar implements Auto {
	@Override
	public void driving() {
		System.out.println("开车了");
	}
}
//新建一个接口类
public interface Intelligent {
	void selfDriving();
}
//实现IntelligentCar
public class IntelligentCar extends DelegatingIntroductionInterceptor implements Intelligent {
 
	@Override
	public void selfDriving() {
		System.out.println("开启无人驾驶了, 别挡路");
	}
 
	public Object invoke(MethodInvocation invocation) throws Throwable {
		Object obj = super.invoke(invocation);
		return obj;
	}
}
```



# Spring AOP Capabilities and Goals

Spring AOP is implemented in pure Java. There is no need for a special compilation(编写) process. Spring AOP does not need to control the class loader hierarchy(等级制度) and is thus suitable for use in a servlet container or application server.

Spring AOP currently supports only method execution join points (advising the execution of methods on Spring beans)(注意这个动词, advising, 在执行方法前后给出建议). Field interception is not implemented, although support for field interception could be added without breaking the core Spring AOP APIs. If you need to advise field access and update join points, consider a language such as AspectJ.

Spring AOP’s approach to AOP differs from that of most other AOP frameworks. The aim is not to provide the most complete AOP implementation (although Spring AOP is quite capable). Rather, the aim is to provide a close integration between AOP implementation and Spring IoC, to help solve common problems in enterprise applications.

Thus, for example, the Spring Framework’s AOP functionality is normally used in conjunction with the Spring IoC container. Aspects are configured by using normal bean definition syntax (although this allows powerful “auto-proxying” capabilities). This is a crucial difference from other AOP implementations. You cannot do some things easily or efficiently with Spring AOP, such as advise very fine-grained(细粒度) objects (typically, domain objects). AspectJ is the best choice in such cases. However, our experience is that Spring AOP provides an excellent solution to most problems in enterprise Java applications that are amenable(服从) to AOP.

(合作共存, 而非竞争)
Spring AOP never strives to compete with AspectJ to provide a comprehensive AOP solution. We believe that both proxy-based frameworks such as Spring AOP and full-blown frameworks such as AspectJ are valuable and that they are complementary, rather than in competition. Spring seamlessly integrates Spring AOP and IoC with AspectJ, to enable all uses of AOP within a consistent Spring-based application architecture. This integration does not affect the Spring AOP API or the AOP Alliance API. Spring AOP remains backward-compatible. See the following chapter for a discussion of the Spring AOP APIs.

(Spring给你选择权)
- One of the central tenets(原则) of the Spring Framework is that of non-invasiveness(非侵袭). This is the idea that you should not be forced to introduce framework-specific classes and interfaces into your business or domain model. However, in some places, the Spring Framework does give you the option to introduce Spring Framework-specific dependencies into your codebase. The rationale in giving you such options is because, in certain scenarios, it might be just plain easier to read or code some specific piece of functionality in such a way. However, the Spring Framework (almost) always offers you the choice: You have the freedom to make an informed decision as to which option best suits your particular use case or scenario.
- One such choice that is relevant to this chapter is that of which AOP framework (and which AOP style) to choose. You have the choice of AspectJ, Spring AOP, or both. You also have the choice of either the @AspectJ annotation-style approach or the Spring XML configuration-style approach. The fact that this chapter chooses to introduce the @AspectJ-style approach first should not be taken as an indication that the Spring team favors the @AspectJ annotation-style approach over the Spring XML configuration-style.

(@Aspect不支持被自动扫描, 需要@component协助)
- You can register aspect classes as regular beans in your Spring XML configuration or autodetect them through classpath scanning — the same as any other Spring-managed bean. However, note that the @Aspect annotation is not sufficient for autodetection in the classpath. For that purpose, you need to add a separate @Component annotation (or, alternatively, a custom stereotype annotation that qualifies, as per the rules of Spring’s component scanner).

# AOP Proxies

Spring AOP is proxy-based. It is vitally important that you grasp the semantics(抓住语义) of what that last statement actually means before you write your own aspects or use any of the Spring AOP-based aspects supplied with the Spring Framework.

Consider first the scenario where you have a plain-vanilla(无特色的), un-proxied, nothing-special-about-it, straight object reference, as the following code snippet shows:
```java
public class SimplePojo implements Pojo{
    public void foo(){
        //this next method invocation is a direct call on the 'this' reference
        this.bar();
    }
    public void bar(){
        //some logic
    }
}
```
If you invoke a method on an object reference, the method is invoked directly on that object reference, as the following image and listing show:
{% asset_img aop-proxy-plain-pojo-call.png %}
```java
public class Main {

    public static void main(String[] args) {
        Pojo pojo = new SimplePojo();
        // this is a direct method call on the 'pojo' reference
        pojo.foo();
    }
}
```

Things change slightly when the reference that client code has is a proxy. Consider the following diagram and code snippet:
{% asset_img aop-proxy-call.png %}

```java
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}

```

The key thing to understand here is that the client code inside the main(..) method of the Main class has a reference to the proxy. This means that method calls on that object reference are calls on the proxy. As a result, the proxy can delegate(代表) to all of the interceptors (advice) that are relevant to that particular method call. However, once the call has finally reached the target object (the SimplePojo, reference in this case), any method calls that it may make on itself, such as this.bar() or this.foo(), are going to be invoked against the this reference, and not the proxy. This has important implications. It means that self-invocation is not going to result in the advice associated with a method invocation getting a chance to execute.

Okay, so what is to be done about this? The best approach (the term, “best,” is used loosely(宽松) here) is to refactor your code such that the self-invocation does not happen. This does entail(必需) some work on your part, but it is the best, least-invasive approach. The next approach is absolutely horrendous(讨厌地难以容忍), and we hesitate to point it out, precisely because it is so horrendous. You can (painful as it is to us) totally tie the logic within your class to Spring AOP, as the following example shows:

```java
public class SimplePojo implements Pojo {

    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
```

This totally couples your code to Spring AOP, and it makes the class itself aware of the fact that it is being used in an AOP context, which flies in the face of AOP. It also requires some additional configuration when the proxy is being created, as the following example shows:
```java
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        factory.setExposeProxy(true);

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}
```
Finally, it must be noted that AspectJ does not have this self-invocation issue because it is not a proxy-based AOP framework.


Spring AOP defaults to using standard JDK dynamic proxies for AOP proxies. This enables any interface (or set of interfaces) to be proxied.

Spring AOP can also use CGLIB proxies. This is necessary to proxy classes rather than interfaces. By default, CGLIB is used if a business object does not implement an interface. As it is good practice to program to interfaces rather than classes, business classes normally implement one or more business interfaces. It is possible to force the use of CGLIB, in those (hopefully rare) cases where you need to advise a method that is not declared on an interface or where you need to pass a proxied object to a method as a concrete type.
