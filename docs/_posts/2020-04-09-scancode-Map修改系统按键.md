---
layout: post
title: scancode-Map修改系统按键
date: 2020-04-09 11:21:08
tags: 技巧
categories:
---

```
在 :
计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Keyboard Layout
下新建二进制文件:
Scancode Map
内容为:
00 00 00 00 00 00 00 00 
03 00 00 00 30 E0 49 E0
2E E0 51 E0 00 00 00 00
即可把Page up/down 改键为 音量升降
```
