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
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" 
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  
  <localRepository>C:\Users\Administrator\.m2\repository</localRepository>

    <!-- 当maven需要输入值时，是否由用户输入，默认为true，当为false时，maven将根据配置信息进行填充 -->
    <interactiveMode>true</interactiveMode>

    <!-- 执行构建时，Maven是否连接网络进行artifact下载、部署等操作，
         默认为false，即默认联网
         当需要处于离线状态(offline)时，将此值改为true。
     -->
    <offline>false</offline>

    <!-- 当插件没有提供groupId时，则使用此处配置的groupId，相当于导入了配置此处配置的groupId的所有组件（使用时下载）
          默认包含两个groupId：org.apache.maven.plugins、org.codehaus.mojo
     -->
    <pluginGroups>
    </pluginGroups>

    <!-- 配置代理，用于多工作环境，通过proxy id即可实现环境切换 -->
    <proxies>
    </proxies>

    <!-- 配置访问远程服务器所需的用户信息，此处多为个人或公司私服的账号信息 -->
    <servers>

      <server>
        <id>maven-releases</id>
        <username>admin</username>
        <password>admin</password>
      </server>

      <server>
        <id>maven-snapshots</id>
        <username>admin</username>
        <password>admin</password>
      </server>
    </servers>

    <mirrors>
      <!-- 配置多个mirror，当mirrorOf的值相同时，当且仅当上一个远程仓库连接失败才会访问下一个远程仓库，
           连接成功后，即使没有获取想要的jar包，也不会访问下一个远程仓库
        -->
      <mirror>
        <!-- 唯一标识一个mirror -->
        <id>aliyun-maven-mirror</id>
        <!-- 指定该镜像代替的时那个仓库，例如central就表示代替官方的中央库，*表示所有仓库都是用该镜像，！表示该仓库除外
             <mirrorOf>*, ! central</mirrorOf> 表示所有的远程仓库 central除外，都使用该阿里云镜像
         -->
        <mirrorOf>central</mirrorOf>
        <name>aliyun Maven</name>
        <url>https://maven.aliyun.com/repository/public</url>
      </mirror>

    </mirrors>

    <profiles>

      <!-- settings文件种的profile一共包含5各元素，起作用分别如下：
              1. id：唯一标识一个profile
              2. activation：profile激活条件配置
                  另外两种激活方式:settings文件种的activeProfile元素（最后一个元素）指定porfile激活
                                命令行通过-P和逗号分隔的列表来激活，如mvn clean package -P profile-id
              3. properties：全局变量设置，一个常见用法就是在此设置jdk版本和编码方式，如下面id为jdk-1.8的profile
              4. repositories：构件远程仓库列表
              5. pluginRepositories：插件的远程仓库配置
       -->

      <!-- 配置maven的jdk版本 -->
      <profile>
        <id>jdk-1.8</id>
        <!-- 下面两个激活项任意一个满足都可激活 -->
        <activation>
          <!-- 该profile是否默认激活 -->
          <activeByDefault>true</activeByDefault>
          <!-- 通过jdk版本前缀来激活当前profile。
               此处当检测到使用的jdk版本是1.8.xxx，则当前profile被激活，!1.8表示激活所有不是以1.8开头的jdk版本
            -->
          <jdk>1.8</jdk>
        </activation>
        <properties>
          <maven.compiler.source>1.8</maven.compiler.source>
          <maven.compiler.target>1.8</maven.compiler.target>
          <maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
        </properties>
      </profile>

      <!-- 配置阿里云Maven -->
      <profile>
        <id>maven-aliyun</id>
        <repositories>
          <repository>
            <!-- 唯一标识远程仓库 -->
            <id>aliyun</id>
            <name>aliyun maven</name>
            <url>https://maven.aliyun.com/repository/public</url>
            <!-- 远程仓库里的发布版本设置 -->
            <releases>
              <!-- 是否使用远程仓库的发布版本 -->
              <enabled>true</enabled>
              <!-- 更新远程仓库发布版本的频率:always-一直,daily-每日（默认）,interval:X-X分钟,never-从不 -->
              <updatePolicy>daily</updatePolicy>
              <!-- maven验证构件检验文件失败时的处理方式:ignore-忽略,fail-失败,warn-警告 -->
              <checksumPolicy>warn</checksumPolicy>
            </releases>
            <!-- 远程仓库里的快照版本设置 -->
            <snapshots>
              <enabled>false</enabled>
              <updatePolicy>always</updatePolicy>
            </snapshots>
          </repository>
        </repositories>
      </profile>

      <!-- 配置私服Maven -->
    </profiles>

    <!-- 手动激活profile -->
    <activeProfiles>
      <!-- activeProfile的属性值就是上面profiles列表种profile的id，若不存在则忽视 -->

      <!-- jdk-1.8已经自动激活，故此处无需显示指定激活 -->
      <!--<activeProfile>jdk-1.8</activeProfile>-->
      <activeProfile>maven-aliyun</activeProfile>
      <activeProfile>maven-nexus</activeProfile>
    </activeProfiles>
  </settings>

 
```
