---
title: 2017.11.1 SpringMVC整合spring学习
date: 2017-10-31 07:55:58
tags: Java Framework
---
和spring框架整合:他们分家
```
Spring负责管理service,dao;
Springmvc负责controller;

注解
@Controller 标识类为Controller,无需继承任何一个controller
在controller中将一个集合给页面访问,Model对象,他是一个map,页面就可以按jstl方式访问
标识每一个
```
redirect 不要request
日期的特殊处理
```
    @InitBinder
    public void initBinder(WebDataBinder binder){
        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(true);
        binder.registerCustomEditor(Date.class,new CustomDateEditor(dateFormat,true));
    }
```
springmvc标签
```
struts2标签非常丰富,远远超过springmvc
文件上传
使用标签
<%@ taglib prefix="sf" uri="http://www.springframework.org/tags/form" %>
<sf:form action="/person/update.action" method="post" modelAttribute="person">

<sf:hidden path="id"/>
<sf:input path="name"/>

</sf:form>


```



文件上传
