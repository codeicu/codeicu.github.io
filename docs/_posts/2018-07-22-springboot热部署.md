---
layout: post
title: springboot热部署
tags: 技巧
date: 2018-07-22 10:40:09
---
# 成功实现热部署
1. pom.xml增加
```
<dependency>
	<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
	<optional>true</optional>
</dependency>
```
2. file>setting>Build,Execution,Deployment>Compiler>Bulid project automatically
3. shift + ctrl + alt + / > Registry
4. compiler.automake.allow.when.app.running
