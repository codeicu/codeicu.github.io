---
layout: post
title: "Docker安装prometheus,jenkins"
date: 2020-11-20T20:50:08+08:00
draft: false
tags: 
categories:
author: Zink
---
非常好用的[Prometheus安装指导](https://www.cnblogs.com/xiao987334176/p/9930517.html), 以及[Jenkins的安装指导](https://www.cnblogs.com/nhdlb/p/12576273.html).

Prometheus + Grafana + Springboot [教程](https://www.cnblogs.com/larrydpk/p/12563497.html)在这里.

需要注意的坑: 

从 Grafana Dashboard 市场查找 Spring Boot 的看板，复制 ID 导入到 Grafana 中，如 6756
导入之后发现有些数据不能正确显示，这是因为设置了变量，需要修改变量的值：
Dashboard Setting -> Variables，选择相应的变量进行修改，这里修改两个：applicaiton 和 instance
```
application: label_values(application)
instance: label_values(jvm_memory_used_bytes{application="$application"},instance)
```
