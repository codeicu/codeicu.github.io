---
layout: post
title: 关闭idea的searchEverywhere
date: 2018-05-24 20:00:41
tags: Java
---
# 为何关闭
英文不存在shift切换输入法的问题, 所以idea把shift连按作为searchEverywhere的快捷键. 然而中文需要不断的切换输入语言,所以这个小问题困扰了我大半年
# 如何关闭
![](https://i.stack.imgur.com/thuZP.png)
![](https://i.stack.imgur.com/v59wL.png)
1. To disable the "Search everywhere" feature, you need to invoke "Go to action" (Ctrl+Shift+A), then type "Registry...".
2. Scroll down to "ide.suppress.double.click.handler" and check the box.
Done!!!
