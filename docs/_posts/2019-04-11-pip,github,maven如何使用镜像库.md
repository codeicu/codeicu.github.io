---
layout: post
title: pip,github,maven如何使用镜像库
date: 2019-04-11 22:17:52
tags: 技巧
---
# pip如何使用镜像库

Linux下，修改 ~/.pip/pip.conf (没有就创建一个文件夹及文件。文件夹要加“.”，表示是隐藏文件夹)

windows下，直接在user目录中创建一个pip目录，如：C:\Users\xx\pip，新建文件pip.ini。内容同上。
```
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple

[install]
trusted-host=mirrors.aliyun.com

```
# github如何加快下载速度

很简单

首先在gitee上搜索,然后在clone gitee中的库

如果没有,就以gitee为中介,先转到gitee上再clone

# maven如何使用镜像库

非常简单,只需要将settings.xml替换为以下内容:
```
<?xml version="1.0" encoding="UTF-8" ?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
    <localRepository>C:\Users\Administrator\.m2\repository</localRepository>
    <mirrors>
        <mirror>
            <id>alimaven-central</id>
            <mirrorOf>central</mirrorOf>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
        </mirror>
    </mirrors>

    <profiles>
        <profile>
            <id>jdk18</id>
            <activation>
                <jdk>1.8</jdk>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <maven.compiler.source>1.8</maven.compiler.source>
                <maven.compiler.target>1.8</maven.compiler.target>
                <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
            </properties>
        </profile>
    </profiles>
</settings>
 
```
