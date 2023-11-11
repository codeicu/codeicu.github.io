---
layout: post
title: "安装虚拟机,开启远程Docker,部署Eureka集群"
date: 2021-01-02T20:41:07+08:00
draft: false
tags: 
categories:
author: Zink
---
- 因为用到集群,所以装个虚拟机来模拟集群

## 安装VMtools
```
   点击安装vmtools
   mkdir /mnt/cdrom 
   mount /dev/cdrom /mnt/cdrom  
   cd /mnt/cdrom
   tar -zxvf  VMwareTools-8.3.2-257589.tar.gz  -C /tmp/
   cd  /tmp/vm**
   ./vmware-install.pl
   umount  /dev/cdrom
```
如果遇到错误:

> Searching for a valid kernel header path… The path “” is not valid

然后
```
	yum install kernel-headers-$(uname -r) 
	yum install kernel-devel-$(uname -r) 
	reboot (注意：内核改变必须重启)
```
最后发现然并卵,非图形界面不能用vmtools复制粘贴, 我..........
曲线救国, 通过ssh来使用虚拟机...

## 网络不能用的解决办法 

```
cd /etc/sysconfig/network-scripts/
vi ifcfg-ens33， 将ONBOOT=no改成ONBOOT=yes
service network restart， 重启网卡，亲测有效
```

## 更换yum为国内源:
```
cat >> /etc/yum.repos.d/sjtu.repo <<EOF
[base]
name=CentOS-$releasever - Base
baseurl=http://ftp.sjtu.edu.cn/centos/7.9.2009/os/x86_64/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[updates]
name=CentOS-$releasever - Updates
baseurl=http://ftp.sjtu.edu.cn/centos/7.9.2009/updates/x86_64/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

[extras]
name=CentOS-$releasever - Extras
baseurl=http://ftp.sjtu.edu.cn/centos/7.9.2009/extras/x86_64/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
EOF

yum clean all
yum update
```

# 安装docker
```
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sudo yum install docker-ce docker-ce-cli containerd.io

systemctl start docker

systemctl stop docker

cat >> /etc/docker/daemon.json <<EOF
{ 
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}    
EOF

systemctl start docker

```
# docker安装软件
```
docker run -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d --restart=always mysql:5.7
docker run -itd --name redis-test -p 6379:6379 -d --restart=always redis
docker run -itd --name mongo -p 27017:27017 -d --restart=always mongo
docker run -d --hostname my-rabbit --name rabbit -v /data/rabbitmq:/var/lib/rabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:3-management
docker run -d --name zookeeper -p 2181:2181 -v /etc/localtime:/etc/localtime wurstmeister/zookeeper
docker run -d --restart=always --log-driver json-file --log-opt max-size=100m --log-opt max-file=2 --name kafka -p 9092:9092 -e KAFKA_BROKER_ID=0 -e KAFKA_ZOOKEEPER_CONNECT=192.168.36.129:2181/kafka -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.36.129:9092 -e KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 -v /etc/localtime:/etc/localtime wurstmeister/kafka
docker run --name zipkin -d -p 9411:9411 openzipkin/zipkin
```

# 开启远程Docker
1. 修改虚拟机IP租用时长, 以免虚拟机频繁更换ip
 ![](/assets/安装虚拟机,开启远程Docker,部署Eureka集群/1.png)
2. 修改Docker配置,支持远程调用
```
vim /lib/systemd/system/docker.service
#修改ExecStart这行
ExecStart=/usr/bin/dockerd  -H tcp://0.0.0.0:2375  -H unix:///var/run/docker.sock

#重新加载配置文件
systemctl daemon-reload    

#重启服务
systemctl restart docker.service 

#查看端口是否开启
netstat -nlpt

#直接curl看是否生效
curl http://127.0.0.1:2375/info

#关闭防火墙
firewall-cmd --zone=public --add-port=2375/tcp --permanent
firewall-cmd --reload
```
3. Intellij IDEA安装Docker插件
```
API url设置为:
tcp://x:2375
```
4. 额外的坑
```
1. Idea需要以管理员方式打开, 否则权限不足, 无法读取Dockerfile

2. springboot如果以jar运行, 需要加入插件
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <executable>true</executable>
            </configuration>
        </plugin>
    </plugins>
</build>

3. 在package阶段运行docker打包发布命令
<plugin>
    <groupId>com.spotify</groupId>
    <artifactId>docker-maven-plugin</artifactId>
    <version>1.0.0</version>

    <!--将插件绑定在某个phase执行-->
    <executions>
        <execution>
            <id>build-image</id>
            <!--将插件绑定在package这个phase上。也就是说，用户只需执行mvn package ，就会自动执行mvn docker:build-->
            <phase>package</phase>
            <goals>
                <goal>build</goal>
            </goals>
        </execution>
    </executions>

    <configuration>
        <!--指定生成的镜像名-->
        <imageName>zink/${project.artifactId}</imageName>
        <!--指定标签-->
        <imageTags>
            <imageTag>latest</imageTag>
        </imageTags>
        <!-- 指定 Dockerfile 路径-->
        <dockerDirectory>${project.basedir}</dockerDirectory>

        <!--指定远程 docker api地址-->
        <dockerHost>http://x:2375</dockerHost>

        <!-- 这里是复制 jar 包到 docker 容器指定目录配置 -->
        <resources>
            <resource>
                <targetPath>/</targetPath>
                <!--jar 包所在的路径  此处配置的 即对应 target 目录-->
                <directory>${project.build.directory}</directory>
                <!-- 需要包含的 jar包 ，这里对应的是 Dockerfile中添加的文件名　-->
                <include>${project.build.finalName}.jar</include>
            </resource>
        </resources>
    </configuration>
</plugin>
```

4. 修改/etc/hosts, 方便集群

5. dockerfile带参数:
```
FROM java:8
COPY target/server-0.0.1-SNAPSHOT.jar app.jar
ENV PARAMS=""
ENTRYPOINT ["java", "-jar", "/app.jar", "$PARAMS"]
```
6. 运行docker集群
```
docker run --name l1 -dp 8763:8763 zink/server -e "--spring.profiles.active=server1" --restart=always
docker run --name l2 -dp 8763:8763 zink/server -e "--spring.profiles.active=server2" --restart=always
docker run --name l3 -dp 8763:8763 zink/server -e "--spring.profiles.active=server3" --restart=always

```

7. 创建bridge network 参考[Spring Cloud Eureka 集群部署 (以及本地Docker部署)](https://www.cnblogs.com/maximo/archive/2004/01/13/13731731.html)
```
在docker中运行eureka server集群, 容器之间是无法通过服务名相互访问的, 
   所以需要创建自己的一个bridge network，然后将eureka服务连接这个network中.
   
# 创建一个自定义的 bridge network，指定网段的时候注意，别和其它网卡的网段冲突
docker network create --subnet=172.19.0.0/16 mynetwork

#查看创建的network信息
docker network inspect mynetwork

#删除网卡命令
#docker network rm mynetwork

# 将eureka服务加入mynetwork网络中
# 注意!!!l1,l2,l3必须为docker的name, 因为bridge是通过name进行通信的

docker network connect mynetwork l1
docker network connect mynetwork l2
docker network connect mynetwork l3

#再次查看network的信息，你会看到每个eureka服务在 mynetwork 中分配的IP信息
docker network inspect mynetwork
```   

# 大功告成!
![](/assets/安装虚拟机,开启远程Docker,部署Eureka集群/2.png)

用时2h+, 从写Dockerfile,到部署集群, 尝试了很多次, 最终成达成目标得益于硬件软件环境配置的好:
1. 电脑cpu风扇的异响通过加点食用油解决了
2. idea的docker插件/Maven插件,方便一键部署
3. 周末把七牛云/Blog的环境设置好,直接可以来写blog
4. 新买的小米键盘好用, 特别是把page键改成home键后, 用的非常顺手
