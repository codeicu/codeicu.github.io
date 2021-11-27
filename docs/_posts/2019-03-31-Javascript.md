---
title: javascript学习
date: 2019-03-31 10:12:45
tags: 前端
---
1. JavaScript用于网页和用户之间的交互，完整的javascript由语言基础，BOM和DOM组成。
2. javascript和DOM结合的一个简单例子
````
<button onclick="document.getElementById('text').style.display='none'">隐藏文本</button>

<button onclick="document.getElementById('text').style.display='block'">显示文本</button>
 
<p id="text"> 这个是一段可以被javascript隐藏的文本</p>
````
3. javascript都是放在script标签中的，一旦加载，就会执行 
4. 如果有多段script代码，会按照从上到下，顺序执行 
5. 声明变量时 关键字var 可有可无 
6. 基本数据类型 undefined，Boolean,Number,String，null 
7. 与java不同的是，javascript中没有字符的概念，只有字符串，所以单引号和双引号，都用来表示字符串。 
8. 变量的类型是动态的，当值是整数的时候，就是Number类型，当值是字符串的时候，就是String类型 
9. 伪对象概念：javascript是一门很有意思的语言，即便是基本类型，也是伪对象，所以他们都有属性和方法。
10. Number转换为字符串的时候有默认模式和基模式两种 
```
  var a=10;
  document.write('默认模式下，数字10转换为十进制的'+a.toString()); //默认模式，即十进制
  document.write("<br>");
 
  document.write('基模式下，数字10转换为二进制的'+a.toString(2)); //基模式，二进制
  document.write("<br>");
   
  document.write('基模式下，数字10转换为八进制的'+a.toString(8)); //基模式，八进制
  document.write("<br>");
 
  document.write('基模式下，数字10转换为十六进制的'+a.toString(16)); //基模式，十六进制
  document.write("<br>");
 
```
11. 数组自定义排序
```
 
function comparator(v1,v2){
   return v2-v1;  //v2-v1表示大的放前面，小的放后面
}
```
12. 连接数组
```
var x = new Array(3,1,4);
var y = new Array(1,5,9,2,6); 
var z = x.concat(y);
```
13. 遍历数组
```
for(i=0;i<x.length;i++){  //普通for循环
  p(x[i]);
}
 
for(i in x){  //for in 循环
  p(x[i]);
}
```
14. 创建数组
```
var x = new Array(); //创建长度是0的数组
p('通过 new Array()创建一个空数组:',x);
x = new Array(5); //创建长度是5的数组,，但是其每一个元素都是undefine
p('通过 new Array(5)创建一个长度是5的数组:',x);
p('像new Array(5) 这样没有赋初值的方式创建的数组，每个元素的值都是:',x[0]);
x = new Array(3,1,4,1,5,9,2,6); //根据参数创建数组
p('创建有初值的数组new Array(3,1,4,1,5,9,2,6) :',x);
```
15. 方法 unshift shift ,分别在最开始的位置插入数据和获取数据(获取后删除) 
16. 获取子数组
```
数组x是:3,1,4,1,5,9,2,6
x.slice(1)获取的子数组是:1,4,1,5,9,2,6
x.slice(1,3)获取的子数组是:1,4
第二个参数取不到
```
17. 删除和插入元素
```
数组x是:3,1,4,1,5,9,2,6
x.splice (3,2) 表示从位置3开始 ，删除2个元素:3,1,4,9,2,6
x.splice(3,0,1,5) 从位置3开始，删除0个元素，但是插入1和5,最后得到:3,1,4,1,5,9,2,6
```
18. 获取一周的第几天
```
var day=new Date().getDay(); //通过日期对象获取数字形式的星期几
var weeks = new Array("星期天","星期一","星期二","星期三","星期四","星期五","星期六");
 
document.write("今天是 ： "+weeks[day]);
```
19. 在JavaScript中可以自定义对象，添加新的属性，添加新的方法 
```
var hero = new Object();
hero.name = "盖伦"; //定义一个属性name，并且赋值
hero.kill = function(){
  document.write(hero.name + " 正在杀敌" ); //定义一个函数kill
}
  
hero.kill(); //调用函数kill
  
```
20.  通过function设计一个对象 
```
通过new Object创建对象有个问题，就是每创建一个对象，都得重新定义属性和函数。这样代码的重用性不好
那么，采用另一种方式，通过function设计一种对象。 然后实例化它 。
这种思路很像Java里的设计一种类，但是 javascript没有类，只有对象，所以我们叫设计一种对象

function Hero(name){
  this.name = name;
  this.kill = function(){
     document.write(this.name + "正在杀敌<br>");
  }
}
 
var gareen = new Hero("盖伦");
gareen.kill();
 
var teemo = new Hero("提莫");
teemo.kill();
```
21.  为已经存在的对象，增加新的方法 
```
function Hero(name){
  this.name = name;
  this.kill = function(){
     document.write(this.name + "正在杀敌<br>");
  }
}
  
var gareen = new Hero("盖伦");
gareen.kill();
  
var teemo = new Hero("提莫");
teemo.kill();
  
Hero.prototype.keng = function(){
  document.write(this.name + "正在坑队友<br>");
}
  
gareen.keng();
teemo.keng();
```
22. BOM即 浏览器对象模型(Browser Object Model)
```
浏览器对象包括
Window(窗口)
Navigator(浏览器)
Screen (客户端屏幕)
History(访问历史)
Location(浏览器地址)
```
23. 获取文档显示区域的高度和宽度 
```
一旦页面加载，就会自动创建window对象，所以无需手动创建window对象。
通过window对象可以获取文档显示区域的高度和宽度 

  document.write("文档显示区域的宽度"+window.innerWidth);
  document.write("文档显示区域的高度"+window.innerHeight);
```
24. 获取外部窗体的宽度和高度 
```
所谓的外部窗体即浏览器，可能用的是360，火狐，IE, Chrome等等
document.write("浏览器的宽度:"+window.outerWidth);
  document.write("<br>");
  document.write("浏览器的高度:"+window.outerHeight);
```
25. 打开一个新的窗口 
```
function openNewWindow(){
  myWindow=window.open("/");
}
```
26. Navigator即浏览器对象，提供浏览器相关的信息 
```
document.write("<p>浏览器产品名称：");
document.write(navigator.appName + "</p>");
 
document.write("<p>浏览器版本号：");
document.write(navigator.appVersion + "</p>");
 
document.write("<p>浏览器内部代码：");
document.write(navigator.appCodeName + "</p>");
 
document.write("<p>操作系统：");
document.write(navigator.platform + "</p>");
 
document.write("<p>是否启用Cookies：");
document.write(navigator.cookieEnabled + "</p>");
 
document.write("<p>浏览器的用户代理报头：");
document.write(navigator.userAgent + "</p>");
```

27. Screen对象表示用户的屏幕相关信息 
```
document.write("用户的屏幕分辨率: ")
document.write(screen.width + "*" + screen.height)
document.write("<br />")
document.write("可用区域大小: ")
document.write(screen.availWidth + "*" + screen.availHeight)
```
28. History用于记录访问历史 
```
function goBack()
  {
     history.back();
  }
 
<button onclick="goBack()">返回</button>
```
29. 跳转到另一个页面
```
function jump(){
  //方法1
  //location="/";
 
  //方法2
  location.assign("/");
   
}
```
30. Location的其他属性
```
p("协议 location.protocol:"+location.protocol);
p("主机名 location.hostname:"+location.hostname);
p("端口号 (默认是80，没有即表示80端口)location.port:"+location.port);
 
p("主机加端口号 location.host: "+location.host);
p("访问的路径  location.pathname: "+location.pathname);
 
p("锚点 location.hash: "+location.hash);
p("参数列表 location.search: "+location.search);
```


