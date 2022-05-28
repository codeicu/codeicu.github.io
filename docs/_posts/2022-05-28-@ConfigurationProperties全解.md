---
layout: post
title:  "@ConfigurationProperties全解"
date:   2022-05-28 21:30:38 +0800
categories: springboot
---

参考[@ConfigurationProperties 注解使用姿势，这一篇就够了](https://blog.csdn.net/yusimiao/article/details/97622666)

## 基本用法

@ConfigurationProperties 的基本用法非常简单:我们为每个要捕获的外部属性提供一个带有字段的类。请注意以下几点:

- 前缀定义了哪些外部属性将绑定到类的字段上
- 根据 Spring Boot 宽松的绑定规则，类的属性名称必须与外部属性的名称匹配
- 我们可以简单地用一个值初始化一个字段来定义一个默认值
- 类本身可以是包私有的
- 类的字段必须有公共 setter 方法

> ### Spring 宽松绑定规则 (relaxed binding)
> Spring使用一些宽松的绑定属性规则。因此，以下变体都将绑定到 hostName 属性上:

```
mail.hostName=localhost
mail.hostname=localhost
mail.host_name=localhost
mail.host-name=localhost
```

## 激活 @ConfigurationProperties

1. 首先，我们可以通过添加 @Component 注解让 Component Scan 扫描到. 很显然，只有当类所在的包被 Spring @ComponentScan 注解扫描到才会生效，默认情况下，该注解会扫描在主应用类下的所有包结构
```
@Component
@ConfigurationProperties(prefix = "myapp.mail")
class MailModuleProps{

}
```

2. 我们也可以通过 Spring 的 Java Configuration 特性实现同样的效果:

```
@Configuration
class MailModuleCfg{

    @ConfigurationProperties(prefix = "myapp.mail")
    @Bean
    public MailModuleProps mailModuleProps(){
        return new MailModuleProps();
    }
}
```

## 无法转换的属性

如果我们在 application.properties 属性上定义的属性不能被正确的解析会发生什么？假如我们为原本应该为布尔值的属性提供的值为 'foo':

```
myapp.mail.enabled=foo
```

默认情况下，Spring Boot 将会启动失败，并抛出异常:

```
Failed to bind properties under 'myapp.mail.enabled' to java.lang.Boolean:

    Property: myapp.mail.enabled
    Value: foo
    Origin: class path resource [application.properties]:1:20
    Reason: failed to convert java.lang.String to java.lang.Boolean
```

当我们为属性配置错误的值时，而又不希望 Spring Boot 应用启动失败，我们可以设置 ignoreInvalidFields 属性为 true (默认为 false)

```
@ConfigurationProperties(prefix = "myapp.mail", ignoreInvalidFields = true)
```

## 未知的属性

然而，当配置文件中有一个属性实际上没有绑定到 @ConfigurationProperties 类时，我们可能希望启动失败. 也许我们以前使用过这个配置属性，但是它已经被删除了，这种情况我们希望被触发告知手动从 application.properties 删除这个属性

```
@ConfigurationProperties(prefix = "myapp.mail", ignoreUnknownFields  = false)
```

## 启动时校验 @ConfigurationProperties

如果我们希望配置参数在传入到应用中时有效的，我们可以通过在字段上添加 bean validation 注解，同时在类上添加 @Validated 注解

## 复杂属性类型

多数情况，我们传递给应用的参数是基本的字符串或数字。但是，有时我们需要传递诸如 List 的数据类型

### List 和 Set

```
myapp.mail.smtpServers[0]=server1
myapp.mail.smtpServers[1]=server2
```

```
myapp:
    mail:
        smtpServers:
            - server1
            - server2
```

### Duration

```
myapp.mail.pause-between-mails=5s

@DurationUnit(ChronoUnit.SECONDS)
private Duration pauseBetweenMails;
```

### DataSize

```
myapp.mail.max-attachment-size=1MB

@DataSizeUnit(DataUnit.MEGABYTES)
private Duration maxAttachmentSize;
```

### 使用 Spring Boot Configuration Processor 完成自动补全

我们向项目中添加依赖:

```
spring-boot-configuration-processor
```

重新 build 项目之后，configuration processor 会为我们创建一个 JSON 文件:

这样，当我们在 application.properties 和 application.yml 中写配置的时候会有自动提醒