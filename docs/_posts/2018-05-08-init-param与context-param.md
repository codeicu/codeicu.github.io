---
layout: post
title: init-param 与 context-param
date: 2018-05-08 15:39:31
tags: Java
---
# 一些碎话
又好多天没写了, 5月1放假,5.4放假, 都没有怎么学习, 想到记写笔记就有些怠倦, 但还是要记录一下.

对于ssm的配置,我一直稀里糊涂的, 这些跟着抄对了,下次就指不定那里出错了,所以还是要一点点把每个配置搞清楚.
# <context-param>配置是一组键值对
```
<context-param>
	<param-name>home-page</param-name>
	<param-value>home.jsp</param-value>
</context-param>
```
param-name是键,param-value是值, 想到与参数值.
# 当服务器启动时，服务器会读取web.xml配置
当服务器启动时，服务器会读取web.xml配置，当读到<listener></listener>和<context-param></context-param>这两个节点的时候，容器会将这两个节点set到ServletContext(上下文对象)中，这样我们在程序中就能通过这个上下文对象去取得我们这个配置值。
具体代码实现:
```
String sHomePage = getServletContext().getInitParameter("home-page");
```
通过上面这句代码，我们就可以取得web.xml中配置的home.jsp这个值。
说白了，他就相当于设定了一个固定值，我们可以在程序中去使用它。就这么个作用。
注：我看到很多文章都是把它和监听一起说的，写说这个配置在监听中怎么用。我要说的他并不是为了监听去设定的。程序中的所有servlet可以利用这个值，我在这里强调一下这一点，希望大家不要被误导
# <context-param>配置和<init-param>的区别：
```
<servlet>
        <servlet-name>ServletInit</servlet-name>
        <servlet-class>com.sunrain.datalk.wserver.util.servlet.ServletInit</servlet-class>
        <init-param>
                  <param-name>home-page</param-name>
                 <param-value>home.jsp</param-value>
        </init-param>
  </servlet>
```
所以他们的区别就有点像全局变量和和局部变量的<context-param>是针对整个项目，所有的servlet都可以取得使用，<init-param>只能是在那个servlet下面





