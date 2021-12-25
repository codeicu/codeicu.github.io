---
layout: post
title:  "Grafana配置数据库及SpringBoot"
date:   2021-11-29 21:25:38 +0800
categories: Linux
---

### 安装docker

```shell
sudo yum install -y yum-uti
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io
# 开启docker
sudo systemctl start docker
# 测试docker
sudo docker run hello-world

```

### 安装Prometheus, Grafana

```shell
docker pull prom/prometheus
mkdir /opt/prometheus
cd /opt/prometheus/
# 详细配置见下方代码
vim prometheus.yml 

docker run -d -p 9090:9090 -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
mkdir /opt/grafana-storage
chmod 777 -R /opt/grafana-storage
docker run -d -p 3000:3000 --name=grafana -v /opt/grafana-storage:/var/lib/grafana grafana/grafana
```

### 安装node-exporter

```shell
docker pull prom/node-exporter
# node-exporter监控linux运行参数
docker run -d -p 9100:9100 -v "/proc:/host/proc:ro" -v "/sys:/host/sys:ro" -v "/:/rootfs:ro" --net="host" prom/node-exporter
```

### 安装mysqld-exporter

```shell
#确保时区是东八区
date -R 
docker pull prom/mysqld-exporter
docker rm -f mysqld-exporter

docker run -d --name mysqld-exporter -v /etc/timezone:/etc/timezone -v /etc/localtime:/etc/localtime -p 9104:9104 -e DATA_SOURCE_NAME="root:xxx@(192.168.159.131:3306)/" --restart=unless-stopped prom/mysqld-exporter

docker logs -f --tail 10 mysqld-exporter 
```

### 安装redis_exporter
```shell
docker run -d --name redis_exporter -p 9121:9121 oliver006/redis_exporter --redis.password 'xxx'
```

### 安装SpringBoot监控

增加pom依赖

```xml
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>io.micrometer</groupId>
      <artifactId>micrometer-registry-prometheus</artifactId>
      <version>1.8.0</version>
    </dependency>
```

修改配置文件

```properties
spring.application.name=springboot_prometheus
management.endpoints.web.exposure.include=*
management.metrics.tags.application=${spring.application.name}
```

访问http://localhost:8080/actuator/prometheus验证是否可以访问metrics

最后在Prometheus中添加spring应用的接口地址:

```yaml
# my global config
global:
scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
# scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
alertmanagers:
- static_configs:
- targets:
# - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
# - "first_rules.yml"
# - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
- job_name: 'prometheus'
static_configs:
- targets: ['127.0.0.1:9090']
###以下内容为SpringBoot应用配置
- job_name: 'springboot_prometheus'
scrape_interval: 5s
metrics_path: '/actuator/prometheus'
static_configs:
- targets: ['127.0.0.1:8080']

```

## 配置

### 在Grafana页面添加mysql/docker/linux Dashboard

- mysql: 6239
- linux: 1860
- redis: 763
- springboot: 4701

### prometheus.yml 详细配置
```
global:
  scrape_interval:     60s
  evaluation_interval: 60s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: prometheus

  - job_name: linux
    static_configs:
      - targets: ['192.168.159.131:9100']
        labels:
          instance: localhost

  - job_name: mysql
    static_configs:
      - targets: ['192.168.159.131:3306']
        labels:
          instance: localhost

  - job_name: 'redis_exporter_targets'
    static_configs:
      - targets:
        - redis://192.168.159.131:7000
        - redis://192.168.159.131:7001
        - redis://192.168.159.131:7002
        - redis://192.168.159.131:7003
        - redis://192.168.159.131:7004
        - redis://192.168.159.131:7005
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.159.131:9121

  - job_name: 'springboot 215'
    scrape_interval: 5s
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['192.168.159.131:8301']
```

## 说明

### Prometheus的存储方式

Prometheus默认使用基于磁盘的时间序列数据库, 非常高效.

默认文件储存时间为2周. 修改方法: 

```sh
vim /etc/sysconfig/prometheus 
STORAGE_RETENTION="--storage.tsdb.retention.time=30d"

# 修改存储时间为30天
docker rm -f prom
docker run -dit --restart always \
--name prom -p 7000:9090 \
-v /root/prometheus/datastore:/prometheus -v /root/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
prom/prometheus \
--storage.tsdb.retention.time=30d \
--config.file=/etc/prometheus/prometheus.yml \
--storage.tsdb.path=/prometheus \
--web.console.libraries=/usr/share/prometheus/console_libraries \
--web.console.templates=/usr/share/prometheus/consoles
```

查看docker中的存储文件

```shell
# 进入prometheus的docker容器
docker exec -it bb65de4ebdce /bin/sh
# 查看prometheus目录文件
/prometheus $ ls -l
total 36
drwxr-xr-x 3 nobody nobody 4096 Nov 29 05:56 01FNN48VHDNZESZRGD8NQ00584
drwxr-xr-x 3 nobody nobody 4096 Nov 29 07:00 01FNN7WJ329FRPPZ8BD2G7CKWT
drwxr-xr-x 2 nobody nobody 4096 Nov 29 07:00 chunks_head
-rw-r--r-- 1 nobody nobody 0 Nov 29 06:26 lock
-rw-r--r-- 1 nobody nobody 20001 Nov 29 06:49 queries.active
drwxr-xr-x 3 nobody nobody 4096 Nov 29 07:00 wal 
```
