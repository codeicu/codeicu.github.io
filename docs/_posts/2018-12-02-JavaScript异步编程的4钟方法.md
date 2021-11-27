---
title: JavaScript异步编程的4钟方法
date: 2018-12-02 11:00:10
tags: Java
---
# JavaScript语言将任务的执行模式分为两种:同步和异步
同步模式不用多解释,异步模式的每一个任务有一个或多个回调函数,前一个任务结束后,不是执行后一个任务,而是执行回调函数.后一个任务是不等前一个任务结束就执行.所以程序的执行顺序与任务的排序是不一致的.

# 回调函数
这是异步编程最近基本的方法.
假定有两个函数f1();f2();
如果f1是一个很耗时的任务,可以改写f1,把f2写成f1的回调函数.
```
function f1(callback){
	setTimeout(function(){
		//f1的任务代码
		callback();
	},1000);
}
```
代码执行期间变成下面这样:
f1(f2);

采用这种方式,简单,容易理解和部署,但不利于代码阅读和维护.各部分高度耦合.每个任务只能指定一个回调函数.
# 事件监听
另一个思路是采用事件驱动模式.
还是以f1和f2为例
```
f1.on('done',f2);
```
然后对f1改写
```
function f1(){
	setTimeout(function(){
		//f1的代码
		f1.trigger('done');
	},1000);
}
```
这种方法的优点是比较容易理解,可以去耦合,缺点是整个程序都变成事件驱动,运行流程会变得不清晰.

# 发布订阅
上一节的"事件"，完全可以理解成"信号"。
我们假定，存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行。这就叫做"发布/订阅模式"（publish-subscribe pattern），又称"观察者模式"（observer pattern）。
这个模式有多种实现，下面采用的是Ben Alman的Tiny Pub/Sub，这是jQuery的一个插件。
首先，f2向"信号中心"jQuery订阅"done"信号。
```
JQuery.subscribe("done",f2);
```
然后,f1进行如下改写.
```
function f1(){
	setTimeout(function(){
		//f1的任务代码
		jQuery.publish("done");	
	},1000);
}
```
这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行。
# Promises对象
Promises对象是CommonJS工作组提出的一种规范，目的是为异步编程提供统一接口。
简单说，它的思想是，每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。比如，f1的回调函数f2,可以写成：f1().then(f2);
f1进行如下改写
```
function f1(){
	var dfd = $.Deferred();
	setTimeout(function(){
		//f1的任务代码
		dtd.resolve();

	},500);
	return dfd.promise;
}
```
# 参考链接
[Javascript异步编程的4种方法](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)






































