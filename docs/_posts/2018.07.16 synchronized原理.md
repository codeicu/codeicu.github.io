---
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
# Synchronized的实现原理
通过一个monitor的对象来完成synchronized,其实wait,notify 等方法也依赖于monitor对象.(所以只有在同步代码块里才能调用wait/notify等方法)
# 对于方法的同步
方法的同步并没有通过指令monitorenter和monitorexit来完成（理论上其实也可以通过这两条指令来实现），不过相对于普通方法，其常量池中多了ACC_SYNCHRONIZED标示符。JVM就是根据该标示符来实现方法的同步的：当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先获取monitor，获取成功之后才能执行方法体，方法执行完后再释放monitor。在方法执行期间，其他任何线程都无法再获得同一个monitor对象。 其实本质上没有区别，只是方法的同步是一种隐式的方式来实现，无需通过字节码来完成。
