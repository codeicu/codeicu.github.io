---
layout: post
title:  "Docker部署Jenkins"
date:   2021-12-25 23:48:38 +0800
categories: Jenkins
---

[TOC]

## Jenkins安装

### Docker常用命令
```shell
docker system df 查看磁盘占用
docker system df 清除磁盘
du -sh 目录占磁盘大小
docker start $(docker ps -a -q)  启动所有容器
```
### Docker安装Jenkins

```shell
docker pull jenkins/jenkins

sudo docker run --memory 1.5G  --name jenkins-gradle1 \
-p 7010:8080 -p 50000:50000 -p 50001:22 -u root -d \
-e TZ=Asia/Shanghai \
--env JAVA_OPTS="-Xms2048m -Xmx4096m"  \
--restart=always \
-v /var/run/docker.sock:/var/run/docker.sock   \
-v /opt/maven/apache-maven:/opt/maven/apache-maven  \
-v /usr/bin/docker:/usr/bin/docker  \
-v /opt/gradle:/opt/gradle \
-v jenkins-data:/var/jenkins_home  \
-v /opt/java/jdk1.8:/opt/java/jdk1.8 \
-v /opt/node:/opt/node \
-v /etc/localtime:/etc/localtime \
-v /usr/lib64/libltdl.so.7:/usr/lib/x86_64-linux-gnu/libltdl.so.7 \
jenkins/jenkins
```

### 修改镜像源

```shell
# 查看首次登录密码
# a1aa20f8a51f49b594603a4e9868191c
cat /var/jenkins_home/secrets/initialAdminPassword

# 修改镜像源
sed -i 's/www.google.com/www.baidu.com/g' /var/jenkins_home/updates/default.json
sed -i 's/updates.jenkins-ci.org\/download/mirrors.tuna.tsinghua.edu.cn\/jenkins/g' /var/jenkins_home/updates/default.json

```

### 下载插件

```sh
maven
gradle
git
subversion
```

### Docker运行时增加Volume映射

```shell
# 通过commit来复制一个同样的容器, 然后基于该容器 增加映射命令
docker commit xxxxId  newContainerName
docker stop xxxxId
docker run -v /var/xx:/var/xx ... newContainerName
```

### Jenkins批量增加用户

```shell
# 下载jar
curl -O http://192.168.159.131:7010/jnlpJars/jenkins-cli.jar

echo 'jenkins.model.Jenkins.instance.securityRealm.createAccount("user1", "password123")' | java -jar jenkins-cli.jar -auth admin:admin -s http://192.168.159.131:7010/ groovy =
```

### 通过chinese-to-pinyin生成用户名

```shell
# 安装chinese-to-pinyin
npm install chinese-to-pinyin --save

# 脚本
var pinyin = require("chinese-to-pinyin")
var a = '张三/n李四'
var arr = a.split("\n")
var s = "echo 'jenkins.model.Jenkins.instance.securityRealm.createAccount(\"username\", \"123456\")' | java -jar jenkins-cli.jar -auth admin:admin -s http://192.168.159.131:7010/ groovy ="
arr.forEach(e => {
    var name = pinyin(e, {removeTone: true,removeSpace: true})
    var x = s.replace("username",name)
    console.log(x)
});

```



### 加入Prometheus监控

```shell
#查看发行版: 
cat /etc/os-release
# Jenkins安装插件
Prometheus metrics plugin

# 增加prometheus配置
  - job_name: jenkins
    metrics_path: /prometheus
    scrape_interval: 10s
    scrape_timeout: 10s
    static_configs:
      - targets: ['192.168.159.131:7010']
# 使用Grafana Dashboard
ID: 9964
```

## Ansible实现自动部署

### 安装Ansible

```shell
apt-get install vim
# 增加镜像源
echo '
deb https://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb https://mirrors.aliyun.com/debian-security/ bullseye-security main
deb-src https://mirrors.aliyun.com/debian-security/ bullseye-security main
deb https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib' >  /etc/apt/sources.list

apt-get update

# 安装 ssh-keygen
apt-get install ssh -y
# 启动sshd
/etc/init.d/ssh start

#ssh设置
#在docker中jenkins下安装
# 生成密钥
ssh-keygen
# 将本地密钥传输到远程
ssh-copy-id -p 45685   root@10.10.184.215

apt-get install ansible -y
# 测试
ansible all -m ping
ansible-galaxy --version
# 安装role
ansible-galaxy install geerlingguy.jenkins
# 设置主机
vi /etc/ansible/hosts

```

### Ansible playbook重启远端服务

```shell
- hosts: 10.10.184.215
  vars:
    today_date: "{{ lookup('pipe', 'date +\"%Y%m%d\"') }}"
  tasks:
    - name: copyfile
      copy:
        src: /opt/dccd-admin/dccd-admin-server.jar
        dest: /opt/dccd-admin/dccd-admin-server{{today_date}}.jar
        remote_src: yes

          #- name: chdir
          # shell:
          # chdir: /opt/dccd-admin

    - name: restart dccd-admin
      command:
        cmd: ./remote_start_app.sh restart
      register: restart_dccd
      args:
        chdir: /opt/dccd-admin

    - name: debug_info
      debug:
        var: restart_dccd
```

## Node环境安装

```shell
# 下载匹配版本
http://nodejs.cn/download/

tar -xvf   node-v16-linux-x64.tar.xz   
mv node-v16-linux-x64  node
# 建立软连接
ln -s /opt/node/bin/npm /usr/bin/npm
ln -s /opt/node/bin/node /usr/bin/node

# 检验
node -v
```

## Gradle环境安装
```shell
# 安装路径
/opt/gradle/gradle-7.3.2

# 下载链接
wget https://services.gradle.org/distributions/gradle-7.3.2-all.zip
unzip gradle-7.3.2-all.zip

```

## Sonar安装
```
docker run -d --name sonarqube -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true -p 9000:9000 sonarqube:latest
```
## Docker迁移目录

因为Docker占空间较大, 原磁盘已经不够了, 从/var迁移到/opt下
```shell
# 停止docker服务
systemctl stop docker
systemctl stop docker.socket

# 备份 /var/lib/docker目录下面的文件到 /opt/var/lib/ 
rsync -avz /var/lib/docker /opt/var/lib/

# 配置 /etc/systemd/system/docker.service.d/devicemapper.conf。查看 devicemapper.conf 是否存在。如果不存在，就新建。
sudo mkdir -p /etc/systemd/system/docker.service.d/
sudo vi /etc/systemd/system/docker.service.d/devicemapper.conf

#  在 devicemapper.conf 写入：（同步的时候把父文件夹一并同步过来，实际上的目录应在 /opt/var/lib/docker ）
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --graph=/opt/var/lib/docker

# 重新加载 docker
systemctl daemon-reload
systemctl restart docker
systemctl enable docker

docker info

# 删除 /var/lib/docker/ 目录中的文件

```

## 当前存在的问题
```
多个jenkins如何共用 mvn,gradle,node_modules
```