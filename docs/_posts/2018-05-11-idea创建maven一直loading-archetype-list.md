---
title: idea创建maven一直loading_archetype_list
date: 2018-05-11 10:48:09
tags: Java
---
# 苦恼的小问题
小问题都有如下的特征
1. 偶尔发生,没有明确的解决办法
2. 懒得去搜怎么解决,但遇到又被弄得很烦躁
3. 只能在忍受不了的时候狠下心去搜索办法.
# 解决办法
```
Setting---->Build Tools → Maven → Importing, set VM options for importer to -Xmx1024m (默认的是-Xmx512m )
```
![图解](http://images2015.cnblogs.com/blog/785703/201602/785703-20160227212621846-1044991160.png)
如果不行,再找其他办法