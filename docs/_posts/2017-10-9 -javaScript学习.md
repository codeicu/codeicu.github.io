---
layout: post
title: 2017.10.9 JavaScript学习
date: 2017-10-17 07:55:58
tags: Java Framework
---
1.改变html内容

```
<p id="demo">改变内容</p>

<script>
    function m(){
        x=document.getElementById("demo");
        x.innerHTML="hello javascript";
        x.style.color="#ff0000";
    }
</script>

<button type="button" onclick="m()">点击这里</button>

```
---


2.验证输入
```
<input id="demo" type="text">
<script>
function myFunction()
{
	var x=document.getElementById("demo").value;
	if(x==""||isNaN(x))
	{
		alert("不是数字");
	}
}
</script>
<button type="button" onclick="myFunction()">点击这里</button>
```
---

3.JavaScript输出
- 使用 window.alert() 弹出警告框。
- 使用 document.write() 方法将内容写到 HTML 文档中。
- 使用 innerHTML 写入到 HTML 元素。
- 使用 console.log() 写入到浏览器的控制台。
- 
---

4.javaScript对大小写是敏感的;JavaScript拥有动态数据类型;如果变量在函数内没有声明（没有使用 var 关键字），该变量为全局变量;
---
5.创建数组
```
var cars=new Array();
cars[0]="Saab";
cars[1]="Volvo";
cars[2]="BMW";
//或者
var cars=["Saab","Volvo","BMW"];
```
---
6.JavaScript 对象
```
var person={firstname:"John", lastname:"Doe", id:5566};
//对象寻址方式:
name=person.lastname;
name=person["lastname"];
```
---
