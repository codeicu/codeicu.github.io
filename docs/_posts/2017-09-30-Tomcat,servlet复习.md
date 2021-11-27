---
layout: post
title:  Tomcat servlet复习
date: 2017-9-30 07:55:58
tags: Java Framework
---
1.tomcat的conf文件下service.xml中配置默认端口8080

---

2.tomcat目录层次结构
- bin  存放启动和关闭Tomcat的脚本文件
- conf tomcat的配置文件
- lib  tomcat所需jar包
- logs 日志文件
- temp 运行时临时文件
- webapps 供外界访问的web资源存放目录
- work 工作目录
- 

---

3.进入服务器conf中的server.xml 中,配置
context代表web应用,path是虚拟路径,docBase是应用位置
修改service后需要重启服务器.


```
<Context path="/itcast" docBase="c:\news"/>
```
这种配置需要重启服务器

---

4.配置缺省web应用
```
<Context path="" docBase="c:\news"/>
```
但这样会覆盖原来的页面

---

5.web.xml文件是web应用中最重要的配置文件,必须放在WEB-INF目录中. 他的作用有
- 将某个web资源配置为网站首页
- 将servlet程序映射到某个url地址
- 为web应用配置监听器
- 为web应用配置过滤器
- 

---

6.web应用的组成结构

- 根目录: html,jsp,css,js文件
- WEB-INF目录: (外界无法访问,由web服务器直接访问)
    
    -classes java类
    
    -lib jar包

    -web.xml
    
---
    
7.配置虚拟主机service.xml中,增加
```
<Host name="www.sina.com" appBase="c:\sina">
    <Context path="/mail" docBase="c:\sina/mail"/>
</Host>
```
然后在hosts文件中配置192.168.1.1 www.sina.com

---

8.配置web应用的首页
```
<welcome-file-list>
    <welcome-file>.html</welcome-file>
</welcome-file-list>
```
9.


