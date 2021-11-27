---
layout: post
title: "Redis+Mysql+Docker搭建"
date: 2020-08-07T10:59:13+08:00
draft: false
tags: Java
categories:
author: Zink 
---
# 使用Docker搭建Mysql服务
1. 拉取官方镜像
```
docker pull mysql:5.7
```
2. 启动mysql
```
sudo docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```
- –name：容器名，此处命名为mysql
- -e：配置信息，此处配置mysql的root用户的登陆密码
- -p：端口映射，此处映射 主机3306端口 到 容器的3306端口
3. 连接mysql
```
sudo docker exec -it mysql bash
mysql -uroot -p123456
```
4. 重启
```
stemctl daemon-reload
systemctl restart docker.service
```
参考链接: ![使用Docker搭建MySQL服务](https://www.cnblogs.com/sablier/p/11605606.html)
