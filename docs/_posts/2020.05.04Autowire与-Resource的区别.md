---
title: '@Autowire与@Resource的区别'
date: 2020-05-04 16:49:07
tags: Spring
categories:
---

1. @Resource的作用相当于@Autowired，均可标注在字段或属性的setter方法上。

2. @Autowired是由org.springframework.beans.factory.annotation.Autowired提供，换句话说就是由Spring提供；@Resource是由javax.annotation.Resource提供，即J2EE提供，需要JDK1.6及以上。

3. @Autowired默认按照byType 注入；@Resource默认按byName自动注入，也提供按照byType 注入；

```java
@Autowired
private UserService userService;
```
这段代码会在初始化时找到一个类型为UserService的Bean注入.

但是如果UserService有多个实现类时:
```java
public class UserSerivce1 implements UserSerivce
public class UserSerivce2 implements UserSerivce
```
按类型注入就会报错. 那么如何应对呢
```java
@Autowired
private UserService userService1;
@Autowired
private UserService userService2;
@Autowired
@Qualifier(value = "userService2")
private UserService userService3;
```


4. @Autowired按类型装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false。如果我们想使用按名称装配，可以结合@Qualifier注解一起使用。@Resource有两个中重要的属性：name和type。name属性指定byName，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。需要注意的是，@Resource如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。

@Resource装配顺序

1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常

2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常

3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常

4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配；

@Autowired applies to fields, constructors, and multi-argument methods,@Resource is supported only for fields and bean property setter methods with a single argument. As a consequence, stick with qualifiers if your injection target is a constructor or a multi-argument method.

If you intend to express annotation-driven injection by name, do not primarily use @Autowired, even if is technically capable of referring to a bean name through @Qualifier values. Instead, use the JSR-250 @Resource annotation, which is semantically defined to identify a specific target component by its unique name, with the declared type being irrelevant for the matching process.

As a specific consequence of this semantic(语意) difference, beans that are themselves defined as a collection or map type cannot be injected through@Autowired, because type matching is not properly applicable to them. Use @Resource for such beans, referring to the specific collection or map bean by unique name.