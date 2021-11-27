---
layout: post
title: "CountDownLatch源码阅读"
date: 2020-06-18T10:19:17+08:00
tags: 面试
---
> Latch: 插销 

A synchronization aid that allows one or more threads to wait until a set of operations being performed in other threads completes.

<p>A {@code CountDownLatch} is initialized with a given <em>count</em>.  The {@link #await await} methods block until the current count reaches  zero due to invocations(调用) of the {@link #countDown} method, after which  all waiting threads are released and any subsequent invocations of  {@link #await await} return immediately.  This is a one-shot phenomenon  -- the count cannot be reset.  If you need a version that resets the  count, consider using a {@link CyclicBarrier}.

<p>A {@code CountDownLatch} is a versatile(多才多艺) synchronization tool  and can be used for a number of purposes.  A  {@code CountDownLatch} initialized with a count of one serves as a  simple on/off latch, or gate: all threads invoking {@link #await await}  wait at the gate until it is opened by a thread invoking {@link  #countDown}.  A {@code CountDownLatch} initialized to <em>N</em>  can be used to make one thread wait until <em>N</em> threads have  completed some action, or some action has been completed N times.

源码中给了两个例子, 这里给形象化一些:
1. 8个运动员依次报数1-8, 听到8时同时出发
2. 8个运动员依次到达终点, 到达8个时触发结束.

CountDownLatch与ReentrantLock一样, 定义了自己的Sync.

该Sync只能减少, 不能增加, 减到0即可释放锁.
```java
    private static final class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = 4982264981922014374L;
		
        Sync(int count) {
            setState(count);
        }

        int getCount() {
            return getState();
        }
		
        protected int tryAcquireShared(int acquires) {
            return (getState() == 0) ? 1 : -1;
        }
		
        protected boolean tryReleaseShared(int releases) {
            // Decrement count; signal when transition to zero
            for (;;) {
                int c = getState();
                if (c == 0)
                    return false;
                int nextc = c-1;
                if (compareAndSetState(c, nextc))
                    return nextc == 0;
            }
        }
    }
```
CountDownLatch的几个重要方法
```java
	//等待, until CountDown到0
	public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(1);
    }
	//可以被打断地等待
    public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
    }
	//
	private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        failed = false;
                        return;
                    }
                }
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
带有等待时常的await方法与上面的await大同小异, 只是多设置了deadline, 首先尝试直接获取, 如果失败就在deadline前获取.
```java
	public boolean await(long timeout, TimeUnit unit)
        throws InterruptedException {
        return sync.tryAcquireSharedNanos(1, unit.toNanos(timeout));
    }

```
与ReentrantLock相似, 本身的概念不复杂, 复杂的是AQS的实现原理, 如setHeadAndPropagate 及shouldParkAfterFailedAcquire, parkAndCheckInterrupt.
这些分析就放在AQS中来看.
