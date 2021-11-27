---
layout: post
title: BlockingQueue源码阅读
date: 2020-06-05 00:05:30
tags: BlockingQueue Java
categories: 
---

BlockingQueue继承了Queue, Queue继承了
```java
public interface BlockingQueue<E> extends Queue<E> 
public interface Collection<E> extends Iterable<E> 
public interface Iterable<T>
```
我们来递归的分析以上接口.

首先看Iterable, Iterable接口中最重要的方法是: 
```java
default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
```
划重点了
1. default是Java8引入的关键字, 打破了接口的限制, 可以在接口中包含方法体. 是为了修改接口又不影响已有的实现.
2. Consumer是一个函数接口,  定义: Represents an operation that accepts a single input argument and returns no result.

然后看Collection, 其方法如下, 包括了集合应有的各种方法.
```
add
addAll
clear
contains
containsAll
equals
hashCode
isEmpty
iterator
parallelStream
void remove
removeAll
removeIf
retainAll
size
spliterator
stream
toArray
```
Queue接口呢, 增加了如下方法, 有些方法虽然与Collection同名, 但返回值类型不同. Queue的定位是: A collection designed for holding elements prior to processing. Queues typically, but do not necessarily, order elements in a FIFO (first-in-first-out) manner.
```
add
element
offer
peek
poll
E remove
```

好了, 分析完BlockingQueue的父类们, 我们来看看这个类本身.
这个类在 package java.util.concurrent下.

BlockingQueue methods come in four forms, with different ways of handling operations that cannot be satisfied immediately, but may be satisfied at some point in the future:  one throws an exception, the second returns a special value (either  {@code null} or {@code false}, depending on the operation), the third  blocks the current thread indefinitely until the operation can succeed,  and the fourth blocks for only a given maximum time limit before giving up.   
总结如下: 

||ThrowsException|Specialvalue|Blocks|Timesout|
|---|---|---|---|---|
|Insert|add(e)|offer(e)|put(e)|offer(e,time)|
|Remove|remove|poll|take|poll(time)|
|Examine(考察)|element|peek|||

- BlockingQueue may be capacity bounded. At any given time it may have a remainingCapacity beyond which no additional elements can be put without blocking.
- BlockingQueue implementations are designed to be used primarily for producer-consumer queues, but additionally support the Collection interface. So, for example, it is possible to remove an arbitrary(任意) element from a queue using remove(x). However, such operations are in general not performed very efficiently, and are intended for only occasional use, such as when a queued message is cancelled.
- BlockingQueue implementations are **thread-safe**. All queuing methods achieve their effects atomically using internal locks or other form of concurrency control. 既然本来就是用来多线程来操作的, BlockingQueue的实现都该是线程安全的, 除了addAll, ContainsAll, RemoveAll这些.

BlockingQueue的方法如下
```
add
contains
drainTo
drainTo
offer
offer
poll
put
remainingCapacity
remove
take
```

看完了接口, 来分析一个实现:ArrayBlockingQueue, 为什么看这个呢, 因为凭直觉看, 这个比较简单...  

This is a classic bounded buffer, in which a fixed-sized array holds elements inserted by producers and extracted by consumers. Once created, the capacity cannot be changed. Attempts to put an element into a full queue will result in the operation blocking; attemps to take an element from an empty queue will similarly block.   

This class supports an optional fairness policy for ordering waiting producer and consumer threads. By default, this ordering is not guaranteed.
```java
public class ArrayBlockingQueue<E> extends AbstractQueue<E> implements BlockingQueue<E>, java.io.Serializable {
	final Object[] items; // the queued items
	int takeIndex; //items index for next take, poll, peek or remove
	int putIndex; //items index for next put, offer
	int count; // Number of elements
	
	//Concurrency control uses the classic two-condition algorithm found in any textbook.
	private final ReentrantLock lock;
	private final Condition notEmpty; // Condition for waiting takes
	private final Condition notFull; // Condition for waiting puts
	
	//Shared state for currently active iterators.
	transient Itrs itrs = null;
	
	//insert the specified element at the tail of this queue if it is possible to do so.
	//return true upon success and throwing an IllegalStateException if this queue is full.
	//Nothing special, 交给父类做
	public boolean add(E e){
		return super.add(e);
	}
	
	//return false if this queue is full, this method is generally preferable to method add.
	public boolean offer(E e) {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count == items.length)
                return false;
            else {
                enqueue(e);
                return true;
            }
        } finally {
            lock.unlock();
        }
    }
	
	//真正的加锁插入queue方法.
	private void enqueue(E x) {
        // assert lock.getHoldCount() == 1;
        // assert items[putIndex] == null;
        final Object[] items = this.items;
        items[putIndex] = x;
        if (++putIndex == items.length)
            putIndex = 0;
        count++;
		//If any threads are waiting on this condition then one is selected for waking up.
        notEmpty.signal();
    }
	enqueue(e);
	
	//waiting for space to become available if the queue is full
	public void put(E e) throws InterruptedException {
        checkNotNull(e);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly(); //Acquires the lock unless the current thread is interrupted
        try {
            while (count == items.length) //满了, 就等待
                notFull.await();
            enqueue(e);
        } finally {
            lock.unlock();
        }
    }
	
	public boolean offer(E e, long timeout, TimeUnit unit)
        throws InterruptedException {

        checkNotNull(e);
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == items.length) {
                if (nanos <= 0)
                    return false;
				//该方法比较复杂, 既要等待, 又要计时
                nanos = notFull.awaitNanos(nanos);
            }
            enqueue(e);
            return true;
        } finally {
            lock.unlock();
        }
    }
	
	public void remove() {
            // assert lock.getHoldCount() == 0;
            final ReentrantLock lock = ArrayBlockingQueue.this.lock;
            lock.lock();
            try {
                if (!isDetached())
                    incorporateDequeues(); // might update lastRet or detach
                final int lastRet = this.lastRet;
                this.lastRet = NONE;
                if (lastRet >= 0) {
                    if (!isDetached())
                        removeAt(lastRet);
                    else {
                        final E lastItem = this.lastItem;
                        // assert lastItem != null;
                        this.lastItem = null;
                        if (itemAt(lastRet) == lastItem)
                            removeAt(lastRet);
                    }
                } else if (lastRet == NONE)
                    throw new IllegalStateException();
                // else lastRet == REMOVED and the last returned element was
                // previously asynchronously removed via an operation other
                // than this.remove(), so nothing to do.

                if (cursor < 0 && nextIndex < 0)
                    detach();
            } finally {
                lock.unlock();
                // assert lastRet == NONE;
                // assert lastItem == null;
            }
        }
	
	//removal of interior elements in circular array based queues is 
	//an intrinsically(本质上) slow and disruptive(引起混乱) operation,
	//so should be undertaken only in exceptional circumstances, ideally 
	//only when the queue is known not to be accessible by other threads.
	public boolean remove(Object o) {
        if (o == null) return false;
        final Object[] items = this.items;
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            if (count > 0) {
                final int putIndex = this.putIndex;
                int i = takeIndex;
                do {
                    if (o.equals(items[i])) {
                        removeAt(i);
                        return true;
                    }
                    if (++i == items.length)
                        i = 0;
                } while (i != putIndex);
            }
            return false;
        } finally {
            lock.unlock();
        }
    }
	
	public E poll(long timeout, TimeUnit unit) throws InterruptedException {
        long nanos = unit.toNanos(timeout);
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0) {
                if (nanos <= 0)
                    return null;
                nanos = notEmpty.awaitNanos(nanos);
            }
            return dequeue();
        } finally {
            lock.unlock();
        }
    }
	
	//一次性取出多个元素
	public int drainTo(Collection<? super E> c, int maxElements) {
        checkNotNull(c);
        if (c == this)
            throw new IllegalArgumentException();
        if (maxElements <= 0)
            return 0;
        final Object[] items = this.items;
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            int n = Math.min(maxElements, count);
            int take = takeIndex;
            int i = 0;
            try {
                while (i < n) {
                    @SuppressWarnings("unchecked")
                    E x = (E) items[take];
                    c.add(x);
                    items[take] = null;
                    if (++take == items.length)
                        take = 0;
                    i++;
                }
                return n;
            } finally {
                // Restore invariants even if c.add() threw
                if (i > 0) {
                    count -= i;
                    takeIndex = take;
                    if (itrs != null) {
                        if (count == 0)
                            itrs.queueIsEmpty();
                        else if (i > take)
                            itrs.takeIndexWrapped();
                    }
                    for (; i > 0 && lock.hasWaiters(notFull); i--)
                        notFull.signal();
                }
            }
        } finally {
            lock.unlock();
        }
    }
}
```

ArrayBlockingQueue存取数据都是使用相同的锁, 所以两者无法同时运行. 但代码复杂度不高, 性能很好.
ArrayBlockingQueue加锁方式有两种: lock.lock() 及lock.lockInterruptibly()(put, offer, take, poll), 我们有必要研究一下两者的区别.
```java
//Acquires the lock if it is not held by another thread and returns immediately,
//setting the lock hold count to one.
//If the current thread already holds the lock then the hold count is incremented by one.
//If the lock is held by another thread, then the current thread becomes disabled 
//for thread schedulinbg purposes and lies dormant(休眠)until the lock has been acquired.
public void lock(){
	sync.lock();
}

//Acquires the lock unless the current thread is interrupted.
//If the lock is held by another thread then the current thread becames disabled 
//for thread schedulinbg purposes and lies dormant(休眠)until one of two things happens.
//1. the lock is accuired by the current thread.
//2. some other thread interrupts the current thread.
public void lockInterruptibly() throws InterruptedException {
	sync.acquireInterruptibly(1);
}
```
总结来说, lock与lockInterruptibly的区别在于: 获取不到锁时, lock一直等, 而lockInterruptibly
被其他线程interrupt就不等了.
这里写一个demo来测试一下:
```java
public static void main(String[] args) {
        ReentrantLock lock = new ReentrantLock(false);
        Thread t1 = new Thread(() -> {
            lock.lock();
            System.out.println("t1获取到锁, 并睡眠5s");
            sleep(5);
            lock.unlock();
        });

        Thread t2 = new Thread(() -> {
            try {
                lock.lockInterruptibly();
                System.out.println("t2获取到锁, 并睡眠5s");
                sleep(5);
                lock.unlock();
            } catch (InterruptedException e) {
                System.out.println("t2 is interrupted, stop acquiring lock");
            }
        });
        t1.start();
        t2.start();
        sleep(2);
        t2.interrupt();
    }

    private static void sleep(int sec) {
        try {
            Thread.sleep(sec*1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
```	
结果如下, 如果线程被打断, 就不在获取锁.
```
t1获取到锁, 并睡眠5s
t2 is interrupted, stop acquiring lock
```
如果在lock.lock()时interrupt该线程, 则不会改变lock(),只是该线程的interrupt flag变了.
所以为啥要用lock.lock()和lock.lockInterruptibly()呢? 所有阻塞获取/添加的值的操作, 都要用lockInterruptibly(),
以便该线程随时被打断, 放弃加锁.
从以上分析来看, BlockingQueue并不复杂(除了removeAt()方法), 主要建立在ReentranLock之上, 所以有必要接着深入ReentrantLock的源码一探究竟. 下篇见!
