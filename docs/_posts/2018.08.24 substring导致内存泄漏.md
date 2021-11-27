---
title: substring导致内存泄漏
tags: 面试
date: 2018-08-24 15:56:14
---
# JDK6中会泄漏，在JDK7中得到解决

String是通过字符数组实现的。在jdk 6 中，String类包含三个成员变量：char value[]， int offset，int count。他们分别用来存储真正的字符数组，数组的第一个位置索引以及字符串中包含的字符个数。
当调用substring方法的时候，会创建一个新的string对象，但是这个string的值仍然指向堆中的同一个字符数组。这两个对象中只有count和offset 的值是不同的。
在jdk 7 中，substring方法会在堆内存中创建一个新的数组。
