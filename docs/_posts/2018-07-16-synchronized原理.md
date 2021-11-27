---
layout: post
title: synchronized原理
date: 2018-07-16 22:20:28
tags: 面试
---
# monitorenter
Each object is associated with a monitor.A monitor is locked if and only if it has an owner.The thread that executes monitorenter attemps to gain ownership of the monitor associated with objectref,as follows:
1. If the entry count of the monitor associated with objectref is zero, the thread enters the monitor and sets its entry count to one.The thread is then the owner of the monitor.
2. If the thread already owns the monitor associated with objectref, it reenters the monitor,incrementing its entry count.
3. If another thread already owns the monitor associated with objectref, the thread blocks until the monitor's entry count is zreo, then tries again to gain ownership.
# monitorexit
The thread that executes monitorexit must be the owner of the monitor associated with the instance referenced by objecref.
The thread decrements the entry count of the monitor associated with objectref. If as a result the value of the entry count is zero, the thread exits the monitor and is no longer its owner. Other threads that are blocking to enter the monitor are allowed to attempt to do so. 
# 加锁方式
三种方式: 实例方法, 静态方法, 代码块  
由于静态成员不属于任何实例对象, 是类成员, 所以通过Class对象锁可以控制静态成员的并发操作.  


# 理解对象头与Monitor
对象在堆内存中布局包括: 对象头+实例数据+对齐补充  
对象头是synchronized的锁对象的基础, 包括Mark Word+ Class Metadata Address(指向对象的类元数据)
- 无锁状态: 25bit HashCode + 4bit 对象分代年龄 + 1bit 是否偏向锁 + 2bit 锁标志位 00
- 轻量级锁: 指向栈中锁记录的指针 + 锁标志位 00
- 重量级锁: 指向互斥量(重量级锁)的指针 + 锁标志位 10
- GC标志: 空 + 锁标志位 11
- 偏向锁: 线程ID + Epoch + 对象分代年龄 + 是偏向锁 + 锁标志位 01

重量级锁指针指向的对象为monitor对象的起始地址. 在Java虚拟机中, monitor是由ObjectMonitor实现的, 位于ObjectMonitor.hpp文件中.

ObjectMonitor有两个队列, _WaitSet和_EntryList, 即保存进入队列与等待队列.

通过一个monitor的对象来完成synchronized,其实wait,notify 等方法也依赖于monitor对象.(所以只有在同步代码块里才能调用wait/notify等方法)

# 对于方法的同步
方法的同步并没有通过指令monitorenter和monitorexit来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。

#偏向锁
1. 读取Mark Word判断是否可偏向
2. 如果可偏向, CAS将线程ID写入Mark Word
3. 如果成功, 执行同步代码
4. 如果失败, 等待全局安全点, 将偏向状态改为0, 验证已获取锁的线程是否存活, 如果存活则升级为轻量级锁, 否则重新加锁.

# 轻量级锁
执行同步代码块之前，JVM会在线程的栈帧中创建一个存储锁记录的空间（Lock Record），并将Mark Word拷贝复制到锁记录中（因为已经脱离了原始的Mark Word，官方以displaced 作为前缀，即Displaced Mark Word（置换标记字））。然后尝试通过CAS将Mark Word中的锁记录的指针，指向创建的Lock Record。如果成功表示获取锁状态成功，如果失败，则进入自旋获取锁状态。
# wait与notify
所谓等待唤醒机制本篇主要指的是notify/notifyAll和wait方法，在使用这3个方法时，必须处于synchronized代码块或者synchronized方法中，否则就会抛出IllegalMonitorStateException异常，这是因为调用这几个方法前必须拿到当前对象的监视器monitor对象，也就是说notify/notifyAll和wait方法依赖于monitor对象，在前面的分析中，我们知道monitor 存在于对象头的Mark Word 中(存储monitor引用指针)，而synchronized关键字可以获取 monitor ，这也就是为什么notify/notifyAll和wait方法必须在synchronized代码块或者synchronized方法调用的原因。

可以[参考]这篇文章(https://blog.csdn.net/javazejian/article/details/72828483)
