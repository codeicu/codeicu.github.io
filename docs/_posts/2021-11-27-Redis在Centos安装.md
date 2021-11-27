---
layout: post
title:  "在Centos安装Redis"
date:   2021-11-27 18:25:38 +0800
categories: Redis
---

1、安装gcc套装：

yum install cpp
yum install binutils
yum install glibc
yum install glibc-kernheaders
yum install glibc-common
yum install glibc-devel
yum install gcc
yum install make
2、升级gcc

yum -y install centos-release-scl

yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

scl enable devtoolset-9 bash

3、设置永久升级：

echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile

4、安装redis：