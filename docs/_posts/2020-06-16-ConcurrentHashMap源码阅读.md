---
layout: post
title: "ConcurrentHashMap源码阅读"
date: 2020-06-16T10:55:20+08:00
draft: false
---

初始化容量不小于concurrencyLevel(estimated threads).
```java
public ConcurrentHashMap(int initialCapacity,
                             float loadFactor, int concurrencyLevel) {
        if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
            throw new IllegalArgumentException();
        if (initialCapacity < concurrencyLevel)   // Use at least as many bins
            initialCapacity = concurrencyLevel;   // as estimated threads
        long size = (long)(1.0 + (long)initialCapacity / loadFactor);
        int cap = (size >= (long)MAXIMUM_CAPACITY) ?
            MAXIMUM_CAPACITY : tableSizeFor((int)size);
        this.sizeCtl = cap;
    }
```

init方法

```java
private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
			//sizeCtl由volatile修饰, 如果为负, 意味其他线程正在扩容, 所以该线程请往后稍稍
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
			//CAS修改,四个参数为: this, valueOffset, expect, update);  
			//即通过(对象, 偏移量)来获取修改处位置, 通过expect, update进行修改 
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
						//求0.75n的高效算法
                        sc = n - (n >>> 2);
                    }
                } finally {
				//下次扩容的阈值
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```
treeifyBin方法
```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
		//如果容量小于64, 则只扩容, 不转化
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                tryPresize(n << 1);
				//如果此处有值
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                synchronized (b) {
				//双重确认
                    if (tabAt(tab, index) == b) {
                        TreeNode<K,V> hd = null, tl = null;\
						//从b开始, 依次转换
                        for (Node<K,V> e = b; e != null; e = e.next) {
                            TreeNode<K,V> p =
                                new TreeNode<K,V>(e.hash, e.key, e.val,
                                                  null, null);
                            if ((p.prev = tl) == null)
                                hd = p;
                            else
                                tl.next = p;
                            tl = p;
                        }
                        setTabAt(tab, index, new TreeBin<K,V>(hd));
                    }
                }
            }
        }
    }
```


get方法

```java
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        //如果tab有数据
        if ((tab = table) != null && (n = tab.length) > 0 &&
            //通过内存位移找到对应的e不为空
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
				//直接找到
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
			//find的实现有好几种, 与e的种类有关
			//eh=-1, 说明该节点是ForwardingNode, 正在迁移
			//eh=-2, 说明该节点是TreeBin, 会加锁读值
			//eh>=0, 说明该节点下挂链表, 直接遍历
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```
分别来看看Node, TreeBin, ForwardingNode节点的find方法
```java
/**
 * Virtualized support for map.get(); overridden in subclasses.
 */
	Node<K,V> find(int h, Object k) {
		Node<K,V> e = this;
		if (k != null) {
			do {
				K ek;
				if (e.hash == h &&
					((ek = e.key) == k || (ek != null && k.equals(ek))))
					return e;
			} while ((e = e.next) != null);
		}
		return null;
	}
```		
TreeBin的find方法
```java
final Node<K,V> find(int h, Object k) {
            if (k != null) {
                for (Node<K,V> e = first; e != null; ) {
                    int s; K ek;
					//如果lockState=Waiter 或者 Writer, 就使用链表形式
					//
                    if (((s = lockState) & (WAITER|WRITER)) != 0) {
                        if (e.hash == h &&
                            ((ek = e.key) == k || (ek != null && k.equals(ek))))
                            return e;
                        e = e.next;
                    }
					//WRITER: 001, WAITER: 010, READER: 100
					//将LOCKSTATE增加READER状态
                    else if (U.compareAndSwapInt(this, LOCKSTATE, s,
                                                 s + READER)) {
                        TreeNode<K,V> r, p;
                        try {
                            p = ((r = root) == null ? null :
                                 r.findTreeNode(h, k, null));
                        } finally {
                            Thread w;
							// 如果当前线程是最后一个读线程，且有写线程因为读锁而阻塞，则写线程，告诉它可以尝试获取写锁了
                            if (U.getAndAddInt(this, LOCKSTATE, -READER) ==
                                (READER|WAITER) && (w = waiter) != null)
                                LockSupport.unpark(w);
                        }
                        return p;
                    }
                }
            }
            return null;
        }
```

ForwardingNode节点的find方法
```java
//A node inserted at head of bins during transfer operations.
static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
			//父类的构造器, hash为-1, key,val,next均为null
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }
		//重写了find方法, h: hash, k: key
        Node<K,V> find(int h, Object k) {
            // loop to avoid arbitrarily(任意) deep recursion on forwarding nodes
			//多年不见的类似goto的语句
            outer: for (Node<K,V>[] tab = nextTable;;) {
                Node<K,V> e; int n;
				//经典的连环灵魂拷问: 你key是null么, 你tab是null么, 你e是null?
                if (k == null || tab == null || (n = tab.length) == 0 ||
                    (e = tabAt(tab, (n - 1) & h)) == null)
                    return null;
                for (;;) {
                    int eh; K ek;
					//开始在nextTable上找key, hash相等并且值相等(先判断hash,效率高, 如果hash不等, 值必然不等. 如果hash相等, 值大概率相等,需要验证)
					//值相等有两种情况, 一种是绝对相等, 一种是equeals
                    if ((eh = e.hash) == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
						//如果eh是-1或者-2
                    if (eh < 0) {
					//ForwardingNode接连ForwardingNode
                        if (e instanceof ForwardingNode) {
                            tab = ((ForwardingNode<K,V>)e).nextTable;
                            continue outer;
                        }
                        else
                            return e.find(h, k);
                    }
                    if ((e = e.next) == null)
                        return null;
                }
            }
        }
    }
```
put方法

```java
/** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
		//不能为null
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
		//目前在bin链上第几个
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
				//通过CAS更新值
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
			//MOVED: -1, hash for forwarding nodes
			//说明正在扩容
            else if ((fh = f.hash) == MOVED)
			//更新tab
                tab = helpTransfer(tab, f);
            else {
			//fh可能为-2, 或>=0, 即TreeBin或下挂链表
                V oldVal = null;
				//对该Node加锁, 其他线程无法改变该Node
                synchronized (f) {
					//双重确认
                    if (tabAt(tab, i) == f) {
						//如果是链表
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
									//key的hash匹配上的话
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
								//巧妙得将e前进一个Node
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
						//如果是TreeBin
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```	
Cmap的计数原理
```java
public int size() {
	long n = sumCount();
	return ((n < 0L) ? 0 :
			(n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
			(int)n);
}
	
final long sumCount() {
	CounterCell[] as = counterCells; CounterCell a;
	long sum = baseCount;
	if (as != null) {
		for (int i = 0; i < as.length; ++i) {
			if ((a = as[i]) != null)
				sum += a.value;
		}
	}
	return sum;
}

private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
		//如果as为null 或者 在basecount上更新失败
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
			//如果 1. as为空 2. 从as中随机取出的值为空 3. 修改as中随机取出的值失败,
			//就fullAddCount. ThreadLocalRandom.getProbe()方法可以避免性能问题及可能
			//产生相同的随机数的问题.
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            s = sumCount();
        }
		//判断是否扩容
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
			//s为元素总数, 
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
				//计算数组长度盖戳标记值
                int rs = resizeStamp(n);
				//sizeCtl: 0 初始值,意味还没数组初始化, -1: 线程开始初始化, -n: 有n个线程
                if (sc < 0) {
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                s = sumCount();
            }
        }
    }
```
counterCells的原理: 
为了统计总数, 构造了包含baseCount和counterCells的数据结构, sum = baseCount+sum of(counterCells), 执行add时, 先更新sum, 如果更新失败则随机更新counterCells中一个元素的值, 将CAS分散在counterCells数组中, 避免在高并发下同时CASbaseCount带来的性能问题.

关于transfer, Cmap定义了最小转移跨度为16,即每个线程至少要转移16个Node.  
但每个cpu能操作多少线程呢, 计算方法为:1个cpu核心数运行8个线程并发执行扩容.
```
private static final int MIN_TRANSFER_STRIDE = 16;
```

tryPresize如何扩容:
```java
/**  尝试对table数组进行扩容,tryPresize在putAll以及treeifyBin中调用
     * Tries to presize table to accommodate the given number of elements.
     *
     * @param size number of elements (doesn't need to be perfectly accurate)
     */
    private final void tryPresize(int size) {
        // 给定的容量若>=MAXIMUM_CAPACITY的一半，直接扩容到允许的最大值，否则调用函数扩容
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
        int sc;
        while ((sc = sizeCtl) >= 0) { //没有正在初始化或扩容，或者说表还没有被初始化
            Node<K,V>[] tab = table; int n;
            //CASE 1: table还未初始化，则先进行初始化
            if (tab == null || (n = tab.length) == 0) {
                n = (sc > c) ? sc : c; // 扩容阀值取较大者
                // 期间没有其他线程对表操作，则CAS将SIZECTL状态置为-1，表示正在进行初始化
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            table = nt;
                            sc = n - (n >>> 2); //无符号右移2位，此即0.75*n
                        }
                    } finally {
                        sizeCtl = sc; // 更新扩容阀值
                    }
                }
            }
            // CASE2: c <= sc说明已经被扩容过了；n >= MAXIMUM_CAPACITY说明table数组已达到最大容量
            else if (c <= sc || n >= MAXIMUM_CAPACITY)
                break;
            else if (tab == table) { // CASE3: 进行table扩容
                int rs = resizeStamp(n); // 根据容量n生成一个随机数，唯一标识本次扩容操作
                if (sc < 0) { // sc < 0 表明此时有别的线程正在进行扩容
                    Node<K,V>[] nt;
                    // 如果当前线程无法协助进行数据转移, 则退出
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0) //RESIZE_STAMP_SHIFT=16,MAX_RESIZERS=2^15-1
                        break;
                    // 协助数据转移, 把正在执行transfer任务的线程数加1
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // sc置为负数, 当前线程自身成为第一个执行transfer(数据转移)的线程
                // 这个CAS操作可以保证，仅有一个线程会执行扩容
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
            }
        }
    }
```


helpTransfer, 如果发现正在扩容, 就帮助扩容.
```java
//先回顾几个概念
RESIZE_STAMP_BITS = 16;
低5位|低15位
int resizeStamp(){return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));}
低15位均为1
最多有几个线程协助resize
int MAX_RESIZERS = (1 << (32 - RESIZE_STAMP_BITS)) - 1;
int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;

final Node<K,V>[] helpTransfer(Node<K,V>[] tab, Node<K,V> f) {
        Node<K,V>[] nextTab; int sc;
        if (tab != null && (f instanceof ForwardingNode) &&
            (nextTab = ((ForwardingNode<K,V>)f).nextTable) != null) {
			//rs: 扩容戳
            int rs = resizeStamp(tab.length);
            while (nextTab == nextTable && table == tab &&
			//sizeCtl: -1 初始化, -(1+n) n个线程在resize
                   (sc = sizeCtl) < 0) {
				   //sc高位生成戳和rs是否相同, 
				   //sc=rs+1 扩容结束
				   //sc=rs+MAX_RESIZERS 帮助线程达到最大
				   //transferIndex<=0 所有的transfer任务都被领取完了
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1)) {
                    transfer(tab, nextTab);
                    break;
                }
            }
            return nextTab;
        }
        return table;
    }
```
transfer
```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {//预处理
                int nextIndex, nextBound;
                if (--i >= bound || finishing)//一次transfer任务还没有执行完毕
                    advance = false;
                else if ((nextIndex = transferIndex) <= 0) {//transfer任务已经没有了
                    i = -1;
                    advance = false;
                }
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;//申请到自己的区间
                    i = nextIndex - 1;
                    advance = false;
                }
            }
			//扩容重叠处理, 但实际不会出现扩容重叠
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
			//Node为null, 不需要迁移,直接安放fwd
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
				//该任务被领取了
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
					//使用高低位
                        Node<K,V> ln, hn;
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
						//红黑树, 先使用链表遍历, 复制节点, 然后高低位
						//组成两个链表, 并判断是否需要红黑树变换, 最后放入Node中
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }
```	

[参考1](https://www.sohu.com/a/320372210_120176035)  
[参考2](https://zhuanlan.zhihu.com/p/89052559)  
[这个是大牛写的,非常详细](https://blog.csdn.net/u011392897/article/details/60479937)
