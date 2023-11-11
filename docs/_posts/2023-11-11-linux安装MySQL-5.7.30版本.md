---
layout: post
title:  "linux安装MySQL-5.7.30版本"
date:   2023-11-11 13:39:38 +0800
categories: linux
---

1、下载 https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz

2、进入Linux系统进行安装

（1）把下载好的mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz包放到/home/tools/目录下

[root@VM-24-12-centos tools]# pwd
/home/tools
[root@VM-24-12-centos tools]# ll
total 644556
（2）安装Mysql数据库时首先要排查服务器内是否已安装mariadb，排查命令如下

[root@VM-24-12-centos ~]# rpm -qa|grep -i mariadb-libs
mariadb-libs-5.5.68-1.el7.x86_64

（3）如果存在需强制卸载，不卸载后面安装时会出问题

[root@VM-24-12-centos ~]# rpm -e mariadb-libs-5.5.68-1.el7.x86_64 --nodeps
[root@VM-24-12-centos ~]# rpm -qa|grep -i mariadb-libs
1
2
3、以上都准备就绪，进行正式安装Mysql

（1）解压二级制安装包

[root@VM-24-12-centos tools]# tar -zxvf mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz
[root@VM-24-12-centos tools]# ll
total 644560
drwxr-xr-x 9 root root      4096 Aug 24 09:47 mysql-5.7.30-linux-glibc2.12-x86_64
-rw-r--r-- 1 root root 660017902 Aug 22 16:01 mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz

（2）复制到/usr/local/目录下

[root@VM-24-12-centos tools]# cp mysql-5.7.30-linux-glibc2.12-x86_64 /usr/local/mysql -r

（3）添加系统mysql组和mysql用户

[root@VM-24-12-centos tools]# groupadd mysql
[root@VM-24-12-centos tools]# useradd -r -g mysql mysql

（4）进入/usr/local/mysql目录，创建data和logs目录

[root@VM-24-12-centos tools]# cd /usr/local/mysql/

（5）把data和logs目录权限分配给mysql用户

[root@VM-24-12-centos mysql]# chown mysql:mysql data/ logs/

（6）切换/ect目录创建my.cnf文件并进行配置

[root@VM-24-12-centos etc]# vim my.cnf
[mysqld]
# 端口
port = 3306
socket = /tmp/mysql.sock
pid-file = /usr/local/mysql/data/mysql.pid
# 设置服务端字节集
character-set-server = utf8
# 设置Mysql的安装目录
basedir = /usr/local/mysql
# 设置Mysql数据库数据存储目录
datadir = /usr/local/mysql/data
# 允许最大连接数
max_connections = 1000
# 创建新表时使用默认存储引擎
default-storage-engine = INNODB
# 设置错误日志
log-error = /usr/local/mysql/logs/error.log
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES

（7）切回/usr/local/mysql目录进行初始化

[root@VM-24-12-centos mysql]# ./bin/mysqld --initialize --user=mysql

（8）如果初始化出现“libaio.so”错误，需安装libaio依赖

# 错误信息
error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory

# 安装libaio依赖
[root@VM-24-12-centos mysql]# yum install -y libaio 或 apt-get install libaio1 -y

（9）查看日志是否初始化成功

[root@VM-24-12-centos mysql]# tail ./logs/error.log 
2022-08-24T02:06:45.022315Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2022-08-24T02:06:45.022376Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2022-08-24T02:06:45.022380Z 0 [Warning] 'NO_AUTO_CREATE_USER' sql mode was not set.
2022-08-24T02:06:45.332738Z 0 [Warning] InnoDB: New log files created, LSN=45790
2022-08-24T02:06:45.399016Z 0 [Warning] InnoDB: Creating foreign key constraint system tables.
2022-08-24T02:06:45.480359Z 0 [Warning] No existing UUID has been found, so we assume that this is the first time that this server has been started. Generating a new UUID: 678a7430-2351-11ed-9619-525400b79b9b.
2022-08-24T02:06:45.484165Z 0 [Warning] Gtid table is not ready to be used. Table 'mysql.gtid_executed' cannot be opened.
2022-08-24T02:06:46.475548Z 0 [Warning] CA certificate ca.pem is self signed.
2022-08-24T02:06:46.725491Z 1 [Note] A temporary password is generated for root@localhost: 0ayYRHdEOa;*
# 未出现[ERROR]错误并且root@localhost密码已显示出来此说明已安装成功

（10）启动Mysql服务

[root@VM-24-12-centos mysql]# service mysql start
Redirecting to /bin/systemctl start mysql.service
Failed to start mysql.service: Unit not found.
# 出现类似问题说明Mysql服务未添加到系统中，需手动添加
[root@VM-24-12-centos mysql]# cp support-files/mysql.server /etc/init.d/mysql
# 再次启动
[root@VM-24-12-centos mysql]# service mysql start
Starting MySQL. SUCCESS! 
# 出现“SUCCESS”表示已启动成功
# 查看Mysql进程
[root@VM-24-12-centos mysql]# ps -ef|grep mysql
root     14289     1  0 10:21 pts/1    00:00:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --datadir=/usr/local/mysql/data --pid-file=/usr/local/mysql/data/mysql.pid
mysql    14504 14289  1 10:21 pts/1    00:00:00 /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/usr/local/mysql/logs/error.log --pid-file=/usr/local/mysql/data/mysql.pid --socket=/tmp/mysql.sock --port=3306
root     14609 13867  0 10:22 pts/1    00:00:00 grep --color=auto mysql

（11）登录Mysql服务并重置root密码

[root@VM-24-12-centos mysql]# mysql -uroot -p
-bash: mysql: command not found
# 出现类似问题需把Mysql放到/user/bin目录下
[root@VM-24-12-centos mysql]# ln -s /usr/local/mysql/bin/mysql /usr/bin
# 再次登录
[root@VM-24-12-centos mysql]# mysql -uroot -p
Enter password: 0ayYRHdEOa;*
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.7.30

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
# 重置root密码
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected (0.00 sec)
# 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
# 退出
mysql> quit;
Bye

# 再次登录
[root@VM-24-12-centos mysql]# mysql -uroot -p
Enter password: 123456
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
# 查看用户
mysql> select host,user,authentication_string from mysql.user;

（12）root用户开启远程访问

mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)
# 查看用户
mysql> select host,user,authentication_string from mysql.user;
+-----------+---------------+-------------------------------------------+
| host      | user          | authentication_string                     |
+-----------+---------------+-------------------------------------------+
| localhost | root          | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
| localhost | mysql.session | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| localhost | mysql.sys     | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| %         | root          | *6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9 |
+-----------+---------------+-------------------------------------------+
4 rows in set (0.00 sec)
# 刷新权限
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
# 退出
mysql> quit;

（13）本地用Navicat连接验证远程访问是否设置成功