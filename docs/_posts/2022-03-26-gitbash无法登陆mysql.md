---
layout: post
title:  "gitbash无法登陆mysql"
date:   2022-03-26 17:20:38 +0800
categories: gitbash
---

我直接输入以下指令，发现卡主了，完全没有反应
```
$ mysql -uroot -p
```

输入以下指定就可以正常登陆MySQL了
```
$ winpty mysql -uroot -p
```