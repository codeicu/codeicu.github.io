---
layout: post
title: "协程. Redis跳表. Kafka在网络波动时怎么保证数据被消费"
date: 2021-01-08T23:41:07+08:00
draft: false
tags: 
categories:
author: Zink
---
# 协程

## 协程简介
1. 协程，又叫微线程，Coroutine。
1. 子程序，或者函数，相互调用是在栈上实现的，一个线程就是一个子程序。
3. 协程看上去像子程序，但运行中可以中断，执行其他子程序，但AB并没有相互调用。  
4. 子程序的切换由自身控制，而非线程控制。  
5. 协程最大的优势是没有线程的切换，效率极高。和多线程比，线程越多优势越明显。  
6. 协程第二大优势是不需要锁机制。因为只有一个线程，所以没有资源冲突。  
7. 协程是一个线程，如何利用CPU多核优势呢？最简单的方式是多进程+协程

## 简单例子
传统的生产者、消费者是两个线程，通过锁控制队列和等待。  
如果改用协程，生产者生产完消息后，通过yield跳转到消费者执行，
完毕后再跳回生产者继续生产，效率极高。

```
import time

def consumer():
    r = ''
    while True:
        n = yield r
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        time.sleep(1)
        r = '200 OK'

def produce(c):
    c.next()
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()

if __name__=='__main__':
    c = consumer()
    produce(c)
```

# Redis跳表

参考[这里](https://blog.csdn.net/whereisherofrom/article/details/84997418)
,图很清晰，讲解到位。

# Kafka

## 消费者丢失数据
Consumer端丢失数据主要体现在：拉取了消息，并提交了消费位移，但是在消息处理结束之前突然发生了宕机等故障。消费者重生后，会从之前已提交的位移的下一个位置重新开始消费，之前未处理完成的消息不会再次处理，即相当于消费者丢失了消息。
解决Consumer端丢失消息的方法也很简单：将位移提交的时机改为消息处理完成后，确认消费完成了一批消息再提交相应的位移。这样做，即使处理消息的过程中发生了异常，由于没有提交位移，下次消费时还会从上次的位移处重新拉取消息，不会发生消息丢失的情况。
具体的实现方法为，Consumer在消费消息时，关闭自动提交位移，由应用程序手动提交位移。
## Broker端丢失数据  
Broker端丢失数据主要有以下几种情况：
原来的Broker宕机了，却选举了一个落后Leader太多的Broker成为新的Leader，那么落后的这些消息就都丢失了，可以禁止这些“unclean”的Broker竞选成为Leader；
Kafka使用页缓存机制，将消息写入页缓存而非直接持久化至磁盘，将刷盘工作交由操作系统来调度，以此来保证高效率和高吞吐量。如果某一部分消息还在内存页中，未持久化至磁盘，此时Broker宕机，重启后则这部分消息丢失，使用多副本机制可以避免Broker端丢失消息；
## 避免消息丢失的最佳实践
不使用producer.send(msg)，而使用带回调的producer.send(msg, callback)方法；
设置acks = all。acks参数是Producer的一个参数，代表了对消息“已提交”的定义。如果设置成all，则表示所有的Broker副本都要接收到消息，才算消息“已提交”，是最高等级的“已提交”标准；
设置retries为一个较大的值，retries表示Producer发送消息失败后的重试次数，如果发生了网络抖动等瞬时故障，可以通过重试机制重新发送消息，避免消息丢失；
设置unclean.leader.election.enable = false。这是一个Broker端参数，表示哪些Broker有资格竞选为分区的Leader。如果一个落后Leader太多的Follower所在Broker成为了新的Leader，则必然会导致消息的丢失，故将该参数设置为false，即不允许这种情况的发生；
设置replication.factor >= 3。Broker端参数，表示每个分区的副本数大于等于3，使用冗余的机制来防止消息丢失；
设置min.insync.replicas > 1。Broker端参数，控制的是消息至少被写入多少个副本蔡栓是“已提交”，将该参数设置成大于1可以提升消息持久性；
确保replication.factor > min.insync.replicas。若两者相等，则如果有一个副本挂了，整个分区就无法正常工作了。推荐设置为：replication.factor = min.insync.replicas + 1；
确保消息消费完再提交位移，将Consumer端参数enable.auto.commit设置为fasle，关闭位移自动提交，使用手动提交位移的形式。
