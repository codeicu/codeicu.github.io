---
layout: post
title:  "gitbash的proxy设置"
date:   2022-03-27 10:30:38 +0800
categories: vue
---

# 最简单的方式: 
```
vi ~/.gitconfig
检查proxy配置, 其正确格式如下: 

[http]
        proxy = http://127.0.0.1:1081
[https]
        proxy = http://127.0.0.1:1081

[user]
        name = zink
        email = zink@no.email
~

```

# 或者
```shell
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890

git config --global user.name "FIRST_NAME LAST_NAME"
git config --global user.email "MY_NAME@example.com"
```