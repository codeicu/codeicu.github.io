---
title: linux搭建SMTP发送邮件服务器
date: 2018-12-24 19:42:26
tags: 技巧
---
# 安装postfix

postfix是一种电子邮件服务器, 他是为了改良sendmail邮件服务器而产生的,最早出现在1990年.
postfix在性能上比sendmail快3倍, 一部运行postfix的台式机每天可以发送上百万封邮件.


```
检查postfix是否已安装
rpm -qa | grep postfix

如果已安装则会显示
postfix-2.6.6-8.el6.x86_64

如果没安装就运行该命令
yum -y install postfix
```

# 修改postfix配置文件

```
文件位置：/etc/postfix/main.cf 修改以下参数,将xxx.com换成你的域名，如果参数前面有#注释，记得去掉 myhostname = mail.xxx.com mydomain = xxx.com myorigin = $mydomain inet_interfaces = all inet_protocols = ipv4 修改完成后运行postifx service postfix start
```

# 设置域名DNS
添加一天A记录指向mail.xxx.com
```
记录类型：A ， 主机记录：mail ，记录值：（127.x.x.x）填写你服务器IP
```

# 发送邮件
使用mail组件进行发送邮件
```
检查mail rpm -qa | grep mail 如果没安装就运行该命令 yum -y install mailx 发送邮件到QQ或163邮箱测试效果 echo "content" | mail -s "title" ewomail@163.com 将(ewomail@163.com)改成你要发送的邮件地址
```



