---
title: linux-ssh-保持长连接-keep-alive
date: 2020-03-31 21:46:39
tags: Linux
categories:
---
服务器总是3分钟就断开连接, 比较麻烦. 所以要设置Keep alive时长

```perl5
1. 打开/etc/ssh/sshd_config
设定ClientAliveCountMax
2. 重启sshd: 
service sshd restart
3. vim .bash_profile
export TMOUT=100000
```
