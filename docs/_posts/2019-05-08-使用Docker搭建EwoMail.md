---
layout: post
title: 使用Docker搭建EwoMail
date: 2019-05-08 11:25:49
tags: Linux
---
# Docker安装EwoMail
在计算机领域, 我一直都有两个心底的梦想. 虽然很小, 很过时, 很基础, 但这也是一直让我不断前进的动力:

---
- 建立自己的网站,并且能被百度检索到
- 使用自己后缀的邮箱
---

为了这两个小想法, 我学习了Java SE, Java EE,Hiberanate, Springboot, Mybatis, Vue, 算法, 计算机网络, Linux......终于, 今天搞定了自己的邮箱服务器,可以稳定的使用和访问了! 回想历程, 还是颇为心酸.比如, 最先在不理解邮件收发过程时,只建立了SMTP服务器,只能发送邮件,不能接收.后来搭建dovecot却一直不能收件,再后来安装Ewomail,却要使用全新的服务器环境,考虑再三也没有重置服务器,而是选择了不太熟悉的docker来承载EwoMail,最后学会端口映射,Apache修改端口,nginx转发等等,才最终将域名和服务器对接成功,在服务器中互不干扰的发布不同的网站服务. 


以上的每一个知识,内容都可以很深奥很底层,虽然我并没有掌握多么牢固,很容易过段时间就忘却脑后,但曾经一步一个脚印的尝试,搜索,学习,修改,经历很多次碰壁后,才摸索出正确的道路,理解其中具体的含义,这些历程却能一直刻印在我内心中,让我以后更加强大.

写下此篇的目的,除了感慨一下完成了幼时所向往的两大计算机任务外,还想详细的记录下搭建Docker中遇到的坑,给自己和他人提供一定帮助.

## 安装Docker中的EwoMail

我的docker还不是很熟悉,所以直接安装网上的标准流程来:
```
docker search EwoMail
docker pull bestwu/ewomail
docker run  -d -h mail.cyz.ink --restart=always -p 25:25 -p 109:109 -p 110:110 -p 143:143 -p 465:465 -p 587:587 -p 993:993 -p 995:995  -p 8081:8081 -p 8080:8080 -p 8082:8082  -v `pwd`/mysql/:/ewomail/mysql/data/ -v `pwd`/vmail/:/ewomail/mail/ -v `pwd`/ssl/certs/:/etc/ssl/certs/ -v `pwd`/ssl/private/:/etc/ssl/private/ -v `pwd`/rainloop:/ewomail/www/rainloop/data -v `pwd`/ssl/dkim/:/ewomail/dkim/ --name ewomail bestwu/ewomail
```
其中-p 8080:8081 是将docker的端口映射到服务器的端口,这样外网才能访问docker中的内容. -v是添加数据卷, --name是给docker起个名字, 后面直接docker start ewomail即可.

此时需要注意,如果docker使用了80端口映射,就容易与服务器中的nginx配置的80端口冲突,导致docker无法成功运行.

解决方法就是,修改Apache中的conf,比如该例子的设置步骤为:

```
- 先关闭nginx服务(ps -ef|grep nginx,kill #pid) 
- 发布docker 
- 进入docker的命令行(docker exec -it ewomail /bin/bash)
- 编辑conf文件: /ewomail/apache/conf/httpd.conf 
- 修改80端口为8080
- 编辑conf文件: /ewomail/apache/conf/extra/httpd.conf/ httpd-vhosts.conf
- 修改其中的端口为8081,8082(注意需在docker命令行中增加8080-8082的端口映射)
- 重启apacheservice httpd restart
- 修改服务器中的nginx,将邮件的服务端,客户端域名分别映射到8080:8082端口
- 运行nginx/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
```
# EwoMail无法收发邮件

发送邮件几分钟后,邮箱显示10024端口 Connection Refused. 解决办法为:
```

- 修改 /etc/postfix/main.cf 这个配置文件 
- 把含有10024端口的一行注释掉
- 重启postfix service postfix reload

```

设置完即可~

做完回头看看,其实并不难,但难点在于:
- 理解架构和运行原理
- 遇到问题找到合适的解决办法

# 推荐使用Docker + EwoMail
这套组合非常适合搭建服务器,既不会对服务器既有的服务造成影响,服务器的服务也不会对docker造成影响,相比之下,官网推荐的重置服务器,也是有点太生硬了.

# 题外话
在看官网教程时,如何配置Apache仅仅是一句话带过,看得我非常迷茫.查找过资料后才知道,这些属于Apache的教程,ewomail只是整合到了一起,故不会把内容说的那么详细.当别人没提到的时候,可能对其他人只是基本操作,自己不懂罢了.



