---
title: linux任务jobs详解
date: 2018-05-10 21:22:26
tags: 技巧
---
# 为什么用到jobs
关于linux的前台后台,一直很迷惑,趁着qqbot的需求,好好的学习一波!主要参考[这里](https://blog.csdn.net/q_l_s/article/details/44117969)以及[这里](https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/)~
# jobs相关命令
1. & 最常被用到,用在命令最后, 让进程在后台运行
2. jobs 查看后台运行的进程
3. fg %n 让后台运行的进程n到前台来
4. bg %n 让进程n到后台去
5. ctrl+z 将前台的命令放到后台暂停执行
6. ctrl+c 强制中断程序的执行/当执行输入某些命令时，使用ctrl+c会自动进入新命令输入。
7. Ctrl-s 暂停屏幕输出【锁住终端】
8. Ctrl-q 恢复屏幕输出【解锁终端】
9. Ctrl-o Discard output
10. Ctrl-l 清屏，【是字母L的小写】等同于Clear
11. Ctrl+a 切换到命令行开始
12. Ctrl+e 切换到命令行末尾
13. Ctrl+u 清除剪切光标之前的内容
14. Ctrl+k 清除剪切光标及光标之后的内容
15. Ctrl+y 在光标处粘贴剪切的内容
16. ps -ef|grep qqbot 查找qqbot的进程号
17. ctrl z>bg %1 将当前运行的进程放在后台
