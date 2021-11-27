---
title: cookie中的path设置
tags: 项目
date: 2018-07-24 15:06:08
---
# 一个由path引发的"血案"
本项目是类似于知乎的问答网站,用户注册/登录后会下发一个cookie,但每次登录的cookie可以正常通过HttpServletResponse来获取,但注册后下发的cookie就没用.
在网页调试中发现,登录后path为"/",而注册后cookie的path为"/reg".所以猜想这个是问题的根源.
然后对比了reg和login的代码,发现login中有一步设置cookie.path="/",而注册没有,所以造成了这个小问题.
# 分析cookie.path的作用
查看源码:
```
Specifies a path for the cookie to which the client should return the cookie.
*  The cookie is visible to all the pages in the directory you specify, and
* all the pages in that directory's subdirectories. A cookie's path must
* include the servlet that set the cookie, for example, <i>/catalog</i>,
* which makes the cookie visible to all directories on the server under
* <i>/catalog</i>.
```
大概含义是,cookie只有在path下才可见.因为没有设置path="/",所以默认cookie在"/reg"目录下才可见,导致了以上的问题.
