---
layout: post
title:  "Linux设置Java路径"
date:   2021-12-25 18:25:38 +0800
categories: linux
---

```shell
vi /etc/profile
export JAVA_HOME="path that you found"
export PATH=$JAVA_HOME/bin:$PATH
```

或者
```
# You could use /etc/profile or better a file like /etc/profile.d/jdk_home.sh

export JAVA_HOME=$(readlink -f /usr/bin/javac | sed "s:/bin/javac::")
```