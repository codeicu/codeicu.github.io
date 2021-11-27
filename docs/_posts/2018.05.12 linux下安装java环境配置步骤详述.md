---
title: linux下安装java环境配置步骤详述
date: 2018-05-12 16:45:37
tags: 技巧
---
# 被linux搞了
linux默认可以运行java命令, 但不能用javac命令, 而且找不到java在哪里, 搞不懂.索性直接删除了原java环境,根据[这篇文章](https://www.cnblogs.com/zeze/p/5902124.html)成功搭建了java环境.
## 下载jdk
官网下载后,通过ftp传送到了服务器. 然后移动到java目录下.
```
cp /home/wwwroot/jdk-8u60-linux-x64.tar.gz /etc/java/
```
## 解压
tar -zxvf jdk-8u60-linux-x64.tar.gz
## 安装完毕为他建立一个链接以节省目录长度
ln -s /etc/java/jdk-8u60/ /etc/jdk
## 编辑配置文件,配置环境变量
```
vim /etc/profile

通过$进入最后一行
添加如下内容
JAVA_HOME=/etc/jdk
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH
```
## 执行命令
```
source /etc/profile
```
## 查看安装情况
直接javac 测试











