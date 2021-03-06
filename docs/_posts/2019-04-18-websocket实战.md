---
layout: post
title: websocket实战
date: 2019-04-18 08:53:09
tags: Java
---
先记录一个困扰我很久的bug吧!
```
<!-- springboot tomcat 支持 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>compile</scope>
		</dependency>
```
此处scope如果为provided,就疯狂报错

对于scope=compile的情况（默认scope),也就是说这个项目在编译，测试，运行阶段都需要这个artifact对应的jar包在classpath中。

而对于scope=provided的情况，则可以认为这个provided是目标容器已经provide这个artifact。换句话说，它只影响到编译，测试阶段。在编译测试阶段，我们需要这个artifact对应的jar包在classpath中，而在运行阶段，假定目标的容器（比如我们这里的liferay容器）已经提供了这个jar包，所以无需我们这个artifact对应的jar包了。

比如说，假定我们自己的项目ProjectABC 中有一个类叫C1,而这个C1中会import这个portal-impl的artifact中的类B1，那么在编译阶段，我们肯定需要这个B1，否则C1通不过编译，因为我们的scope设置为provided了，所以编译阶段起作用，所以C1正确的通过了编译。测试阶段类似，故忽略。

那么最后我们要吧ProjectABC部署到Liferay服务器上了，这时候，我们到$liferay-tomcat-home\webapps\ROOT\WEB-INF\lib下发现，里面已经有了一个portal-impl.jar了，换句话说，容器已经提供了这个artifact对应的jar,所以，我们在运行阶段，这个C1类直接可以用容器提供的portal-impl.jar中的B1类，而不会出任何问题。

刚才我们讲述的是理论部分，现在我们看下，实际插件在运行时候，是如何来区别对待scope=compile和scope=provided的情况的。

做一个实验就可以很容易发现，当我们用maven install生成最终的构件包ProjectABC.war后，在其下的WEB-INF/lib中，会包含我们被标注为scope=compile的构件的jar包，而不会包含我们被标注为scope=provided的构件的jar包。这也避免了此类构件当部署到目标容器后产生包依赖冲突。
