---
layout: post
title: "ReentrantLock源码阅读"
date: 2020-06-06T10:53:57+08:00
tags: 多线程
---

先看类注释: 

A reentrant mutual exclusion(互相排斥) Lock with the same basic behavior and semantics 
as the implicit monitor lock accessed using synchronized methods and statements, 
but with extended capabilities.

The constructor for this class accepts an optional fairness parameter. Programs using fair 
locks accessed by many threads may display lower overall throughput.(i.e., are slower; 
often much slower) than using the default setting, but have smaller variances(变化) in times
to obtain locks and gurarantee lack of starvation.

It's recommended practice to **always** immediately follow a call to lock with a try block.

```java
//Synchronizer同步器 providing all implementation mechanics
private final Sync sync;
```
递归分析Sync的继承结构:
```java
abstract static class Sync extends AbstractQueuedSynchronizer 
public abstract class AbstractQueuedSynchronizer extends AbstractOwnableSynchronizer 
public abstract class AbstractOwnableSynchronizer
```

AbstractOwnableSynchronizer:  
A synchronizer that may be exclusively owned by a thread. This class provides a basis for creating locks and related sychronizers that may entail(需要) a notion of ownership. The AbstractOwnableSynchronizer class itself does not manage or use this information.  
该接口定义了如下方法
```
AbstractOwnableSynchronizer
getExclusiveOwnerThread
setExclusiveOwnerThread
exclusiveOwnerThread
```
AbstractQueuedSynchronizer:
Provides a framework for implementing blocking locks and related synchronizers (semaphores旗语, events, etc)that rely on FIFO wait queues. This class is designed to be a sueful basis for most kinds of synchronizers that rely on a single atomic int value to represent state. 

AQS是一个抽象类, 定义了一套**多线程访问共享资源的同步器框架**, ReentrantLock/Semaphore/CountDownLatch等同步类都用到了它. AQS核心是维护了一个volatile in state和FIFO线程等待序列.
其最重要的成员变量为: 
```
private transient volatile Node head;
private transient volatile Node tail;
private volatile int state;// state 为0表示锁可用, state=1表示锁已被用
```
- 获取同步状态: state=0时, 线程A获取并置state为1. 此时B也来获取锁, 就将B的线程信息和等待状态构造出Node节点对象放入同步队列, 同时阻塞B线程(LockSupport.park())
- 释放同步状态: A释放锁时, 将state置为0, 并唤醒后置节点(LockSupport.unpark(B))

Sync继承了AQS, 并实现了获取锁/释放锁/一些判断的方法, 比较简单
```java
//因为没有队列控制先来后到, 所以是非公平锁, 谁抢到算谁的
final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
			//还没有上锁, 就将该锁设为自己独占
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }//如果是自己独占, 就"重入"
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
				//这里会有overflow的处理, 毕竟相加有可能会溢出, 即使一般情况不会发生
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```
接着有定义了两个Sync的子类, 首先是非公平锁
```java
static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
			//没通过CAS获取到锁, 就自旋
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```	
另一个是公平锁
```java
static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;
		//与非公平锁的acquire是相同的方法
        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
			//hasQueuedPredecessors: 当前线程是queue的head或者queue为空时返回false
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
    }
```	
ReentrantLock有两种lock方式, lock()及lockInterruptibly(), 简单来说前者不得到lock不放弃, 后者如果被打断, 就放弃获取锁. 我们来看看其实现:
```java
public void lock() {
//由公平锁/非公平锁有各自的实现
	sync.lock();
}


public void lockInterruptibly() throws InterruptedException {
	sync.acquireInterruptibly(1);
}

public final void acquireInterruptibly(int arg)
		throws InterruptedException {
	if (Thread.interrupted())
		throw new InterruptedException();
		//尝试获取一次锁, 万一没人竞争呢?锁直接就到手了
	if (!tryAcquire(arg))
		doAcquireInterruptibly(arg);
}
	
private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
				//该node的前任
                final Node p = node.predecessor();
				//如果前任是head, 说明排到自己了, 就尝试获取锁
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
				//简单来说, 当前不在队列的第二位, 就往前看看是否有node取消排队了
				//如果前方有node取消, 就往前进一位, 直到前方node不在取消状态.
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```
还有一种限定时间的lock
```java
	public boolean tryLock(long timeout, TimeUnit unit)
            throws InterruptedException {
        return sync.tryAcquireNanos(1, unit.toNanos(timeout));
    }
	
    private boolean doAcquireNanos(int arg, long nanosTimeout)
            throws InterruptedException {
        if (nanosTimeout <= 0L)
            return false;
        final long deadline = System.nanoTime() + nanosTimeout;
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return true;
                }
				//设定deadline, 超时即放弃
                nanosTimeout = deadline - System.nanoTime();
                if (nanosTimeout <= 0L)
                    return false;
                if (shouldParkAfterFailedAcquire(p, node) &&
				//预估时间, 如果时间不够自旋, 就放弃了
                    nanosTimeout > spinForTimeoutThreshold)
                    LockSupport.parkNanos(this, nanosTimeout);
                if (Thread.interrupted())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }	
```	
最后对比一下与Synchronized的区别
Synchronized: 不需要手动释放, 不可中断, 没有高级功能
ReentrantLock: 更多样化, 如时间限制的功能, 可以实现公平锁, 同步激烈时性能较好
ReentrantLock本身不难理解, 代码都很直观. 深入理解其原理需要理解其中的Sync成员. 而Sync是继承与AbstractQueuedSynchronizer, AbstractQueuedSynchronizer本身是多线程访问共享资源的同步器框架, 内容比较丰富, 我们接下来就深入看看这个类的实现方式.
