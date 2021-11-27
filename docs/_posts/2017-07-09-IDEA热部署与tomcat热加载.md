---
layout: post
title: IDEA热部署与tomcat热加载
date: 2017-07-09 09:42:02
tags: 技巧
---
- 1.打开Configurations
- 2.On 'Update' action: Redeploy
- 3.On frame deactivation: Update classes and resources
- 4.最下面Build中选项目
- 

---
打开tomcat中conf的context.xml,在context后添加成如下语句:
<Context reloadable="true">

---
css
- id选择器 以#开头
- 类选择器 以.开头
- 元素选择器 选择所有元素"div"
---
修改类中的text
```
var ele = $(".myclass");
ele.text("888");

```
---
属性选择器
```
$("[name=age]")属性名为name,值为age的元素
 alert($("[name=age]").val("hello")); 将其值改为hello
```
---
获取input中name的值为"name"的元素的id
```
$(":input[name=name]").attr("id")
```
将input中name的值为"name"的元素的id改为idxxx
```
$(":input[name=name]").attr("id","idxxx")
```
删除
```
removeAttr(name)
```
---
给id为mydiv的追加xxx的样式
```
$("#mydiv").addClass("xxx");

<style type="text/css">
    .xxx{
        border: 1px solid red;
        width: 100px;
        height: 100px;
    }
</style>
```
---
css把元素隐藏
```
$("#mydiv").css("display","none");
```
css把元素属性更改
```
$("#mydiv").css("border","1px solid blue").css("width","100").css("height",100);
```
循环遍历
```
$("input").each(function(){
    
});
```
增加点击事件
```
$("#btn").click(function(){
   alert("hello"); 
});
```
给btn绑定click再解除
```

```

其他事件
```
$("#xxx").hover(
    function () {
        alert("进来");
},
function () {
    alert("出去");
}
)
```
发送异步请求
```
$("#xxx1").blur(function () {
    alert("走了");
    $.ajax({
        url:"",
        data:{},
        async:true,
        cache:false,
        type:"post",
        dataType:"json",
        success:function (result) {
            if( result){
            }else {
            }
        }
    })
});
```
