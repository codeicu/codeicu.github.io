---
layout: post
title:  "fiddler抓包_proxy一直被修改"
date:   2022-05-03 14:48:38 +0800
categories: windows
---

## fiddler一直提示: Fiddler not capturing traffic. Proxy settings keep getting changed

> ![](/assets/fiddler抓包_proxy一直被修改/1.png)

## 解决方案

参考(Stack Overflow)[https://stackoverflow.com/questions/28764968/fiddler-not-capturing-traffic-proxy-settings-keep-getting-changed]提供的解决方案

1. 下载ProcessMonitor3
2. 过滤关键字: Proxy
3. 找到软件: Sangfor 深信服的 EasyConnect, 直接卸载

## 感悟

当找不到根本原因, 而一直尝试时又不成功时,非常令人沮丧.

8efc6b2d3b069ca89454d10b9ef0018749ddf94d356b53a614ecc3d8f6b91b590cc162017c532b97df3105ea6c7ab2b470de55faefa29acb5a7f7a34111e38bf