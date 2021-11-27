---
layout: post
title: "AbstractQueuedSynchronizer"
date: 2020-06-18T14:02:34+08:00
draft: false
---

The wait queue is a variant of a "CLH" lock queue. 关于CLH的背景可以参考[这里](https://blog.csdn.net/claram/article/details/83828768). 简单来说是适用于SML(对称多处理器)架构下的锁. 基于单链表,高性能,公平的自旋锁. CLH queues need a dummy header node to get start. But we don't create them on construction, because it would be wasted effort if there is never contention(争吵).

```
static final class Node {
	/** Marker to indicate a node is waiting in shared mode */
	static final Node SHARED = new Node();
	/** Marker to indicate a node is waiting in exclusive mode */
	static final Node EXCLUSIVE = null;

	/** waitStatus value to indicate thread has cancelled */
	static final int CANCELLED =  1;
	/** waitStatus value to indicate successor's thread needs unparking */
	static final int SIGNAL    = -1;
	/** waitStatus value to indicate thread is waiting on condition */
	static final int CONDITION = -2;
	/**
	 * waitStatus value to indicate the next acquireShared should
	 * unconditionally propagate
	 */
	static final int PROPAGATE = -3;
}


	private Node enq(final Node node) {
        for (;;) {
            Node t = tail;
            if (t == null) { // Must initialize
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
				//如果不用CAS, 可能会添加两次尾节点
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
	
	private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        //好像也没fast多少?enq只是多了初始化的判断而已
		Node pred = tail;
        if (pred != null) {
            node.prev = pred;
            if (compareAndSetTail(pred, node)) {
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }
	
	//Head是dummy
	private void setHead(Node node) {
        head = node;
        node.thread = null;
        node.prev = null;
    }
	
	//唤醒后置节点, 因为在cas设置tail=node与pre.next=node之间，可能会有其他线程进来
	//所以出现了问题，从尾部向前遍历是一定能遍历到所有的节点。看[这里解释](https://www.zhihu.com/question/50724462/answer/123776334)
	private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation(预期) of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
		 //
        int ws = node.waitStatus;
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
            LockSupport.unpark(s.thread);
    }
	
	private void doReleaseShared() {
        /*
         * Ensure that a release propagates, even if there are other
         * in-progress acquires/releases.  This proceeds in the usual
         * way of trying to unparkSuccessor of head if it needs
         * signal. But if it does not, status is set to PROPAGATE to
         * ensure that upon release, propagation continues.
         * Additionally, we must loop in case a new node is added
         * while we are doing this. Also, unlike other uses of
         * unparkSuccessor, we need to know if CAS to reset status
         * fails, if so rechecking.
         */
        for (;;) {
            Node h = head;
            if (h != null && h != tail) {
                int ws = h.waitStatus;
                if (ws == Node.SIGNAL) {
				//将自身节点改为0, 成功后唤醒后置节点
                    if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
                        continue;            // loop to recheck cases
                    //唤醒后置节点, 此时head可能会变化
					unparkSuccessor(h);
                }
                else if (ws == 0 &&
                         !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
                    continue;                // loop on failed CAS
            }
            if (h == head)                   // loop if head changed
                break;
        }
    }
	
	//只有在pred.waitStatus已经等于Node.SIGNAL时才会返回true
	private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
        int ws = pred.waitStatus;
        if (ws == Node.SIGNAL)
            /*
             * This node has already set status asking a release
             * to signal it, so it can safely park.
             */
            return true;
        if (ws > 0) {
            /*
             * Predecessor was cancelled. Skip over predecessors and
             * indicate retry.
             */
            do {
                node.prev = pred = pred.prev;
            } while (pred.waitStatus > 0);
            pred.next = node;
        } else {
            /*
             * waitStatus must be 0 or PROPAGATE.  Indicate that we
             * need a signal, but don't park yet.  Caller will need to
             * retry to make sure it cannot acquire before parking.
             */
            compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
        }
        return false;
    }
	
	//Acquires in exclusive uninterruptible mode for thread already in queue.
	//Used by condition wait methods as well as acquire.
	final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false;
            for (;;) {
                final Node p = node.predecessor();
				//p为head 且获取到了锁, 就将head 移除
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return interrupted;
                }
				//判断是否需要park, 如果需要,就park, 并返回是否被interrupted
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    interrupted = true;
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
	
	
	
```
