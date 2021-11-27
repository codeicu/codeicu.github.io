---
title: ConcurrentHashMap的锁分段技术
date: 2018-07-17 21:24:34
tags: 面试
---
1.哈希算法: 是一种将任意内容的输入转换成相同长度输出的加密方式，其输出被称为哈希值。 
2. 哈希表: 根据设定的哈希函数H(key)和处理冲突方法将一组关键字映象到一个有限的地址区间上，并以关键字在地址区间中的象作为记录在表中的存储位置，这种表称为哈希表或散列，所得存储位置称为哈希地址或散列地址。
# 线程不安全的HashMap
因为多线程环境下，使用Hashmap进行put操作会引起死循环，导致CPU利用率接近100%，所以在并发情况下不能使用HashMap。
# 效率低下的HashTable容器
HashTable容器使用synchronized来保证线程安全，但在线程竞争激烈的情况下HashTable的效率非常低下。因为当一个线程访问HashTable的同步方法时，其他线程访问HashTable的同步方法时，可能会进入阻塞或轮询状态。如线程1使用put进行添加元素，线程2不但不能使用put方法添加元素，并且也不能使用get方法来获取元素，所以竞争越激烈效率越低。
# ConcurrentHashMap的锁分段技术
HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。
# 实现原理
<p><img alt="" src="http://static.oschina.net/uploads/space/2016/0317/154925_NLV0_2243330.png"></p>
从图中可以看到，ConcurrentHashMap内部分为很多个Segment，每一个Segment拥有一把锁，然后每个Segment（继承ReentrantLock）
```
static final class Segment<K,V> extends ReentrantLock implements Serializable
```
这样对每个segment中的数据需要同步操作的话都是使用每个segment容器对象自身的锁来实现。只有对全局需要改变时锁定的是所有的segment。

Segment下面包含很多个HashEntry列表数组。对于一个key，需要经过三次（为什么要hash三次下文会详细讲解）hash操作，才能最终定位这个元素的位置，这三次hash分别为：

对于一个key，先进行一次hash操作，得到hash值h1，也即h1 = hash1(key)；
将得到的h1的高几位进行第二次hash，得到hash值h2，也即h2 = hash2(h1高几位)，通过h2能够确定该元素的放在哪个Segment；
将得到的h1进行第三次hash，得到hash值h3，也即h3 = hash3(h1)，通过h3能够确定该元素放置在哪个HashEntry。

ConcurrentHashMap中主要实体类就是三个：ConcurrentHashMap（整个Hash表）,Segment（桶），HashEntry（节点），对应上面的图可以看出之间的关系

在JDK1.6中，HashEntry中的next指针也定义为final，并且每次插入将新添加节点作为链的头节点（同HashMap实现），而且每次删除一个节点时，会将删除节点之前的所有节点 拷贝一份组成一个新的链，而将当前节点的上一个节点的next指向当前节点的下一个节点，从而在删除以后有两条链存在，因而可以保证即使在同一条链中，有一个线程在删除，而另一个线程在遍历，它们都能工作良好，因为遍历的线程能继续使用原有的链。因而这种实现是一种更加细粒度的happens-before关系，即如果遍历线程在删除线程结束后开始，则它能看到删除后的变化，如果它发生在删除线程正在执行中间，则它会使用原有的链，而不会等到删除线程结束后再执行，即看不到删除线程的影响。如果这不符合你的需求，还是乖乖的用Hashtable或HashMap的synchronized版本，Collections.synchronizedMap()做的包装。

不变模式（immutable）是多线程安全里最简单的一种保障方式。因为你拿他没有办法，想改变它也没有机会。不变模式主要通过final关键字来限定的。在JMM中final关键字还有特殊的语义。Final域使得确保初始化安全性（initialization safety）成为可能，初始化安全性让不可变形对象不需要同步就能自由地被访问和共享。

# 传入的参数initialCapacity,locdFactor,concurrencyLevel这三个
+ initialCapacity表示新创建的这个ConcurrentHashMap的初始容量，也就是上面的结构图中的Entry数量。默认值为static final int DEFAULT_INITIAL_CAPACITY = 16;
+ loadFactor表示负载因子，就是当ConcurrentHashMap中的元素个数大于loadFactor * 最大容量时就需要rehash，扩容。默认值为static final float DEFAULT_LOAD_FACTOR = 0.75f;
+ concurrencyLevel表示并发级别，这个值用来确定Segment的个数，Segment的个数是大于等于concurrencyLevel的第一个2的n次方的数。比如，如果concurrencyLevel为12，13，14，15，16这些数，则Segment的数目为16(2的4次方)。默认值为static final int DEFAULT_CONCURRENCY_LEVEL = 16;。理想情况下ConcurrentHashMap的真正的并发访问量能够达到concurrencyLevel，因为有concurrencyLevel个Segment，假如有concurrencyLevel个线程需要访问Map，并且需要访问的数据都恰好分别落在不同的Segment中，则这些线程能够无竞争地自由访问（因为他们不需要竞争同一把锁），达到同时访问的效果。这也是为什么这个参数起名为“并发级别”的原因。

# 转载

<div class="content" id="articleContent">
                                                    <div class="ad-wrap">
                                                        <p style="margin:0 0 10px 0;"><a data-traceid="blog_detail_above_text_link_1" data-tracepid="blog_detail_above_text_link" style="color:#A00;font-weight:bold;" href="http://clickc.admaster.com.cn/c/a109734,b2619652,c3159,i0,m101,8a1,8b2,h" target="_blank">华为云4核8G，高性能云服务器，免费试用
&gt;&gt;&gt; </a>&nbsp;&nbsp;<img src="https://www.oschina.net/img/hot3.png" style="max-height: 32px; max-width: 32px;" align="absmiddle"></p>
                                        </div>
                                                <p>曾经在&nbsp;<span style="font-size:12.5px"><a href="http://my.oschina.net/hosee/blog/607677" target="_blank" rel="nofollow">[高并发Java 五] JDK并发包1</a>&nbsp;中提到过ConcurrentHashMap，只是简单的提到了下ConcurrentHashMap的优点，以及大概的实现原理。</span></p> 
<p><span style="font-size:12.5px">而本文则重点介绍ConcurrentHashMap实现的细节。</span></p> 
<p><span style="font-size:12.5px">HashMap就不介绍了，具体请查看<a href="http://my.oschina.net/hosee/blog/618953" target="_blank" rel="nofollow">JDK7与JDK8中HashMap的实现</a></span></p> 
<p><span style="font-size:12.5px">HashTable是一个线程安全的类，它使用</span>synchronized来锁住整张Hash表来实现线程安全，即每次锁住整张表让线程独占。ConcurrentHashMap允许多个修改操作并发进行，其关键在于使用了<span style="color:#E53333">锁分离</span>技术。它使用了多个锁来控制对hash表的不同部分进行的修改。ConcurrentHashMap内部使用段(Segment)来表示这些不同的部分，每个段其实就是一个小的Hashtable，它们有自己的锁。只要多个修改操作发生在不同的段上，它们就可以并发进行。</p> 
<p>有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。这里“按顺序”是很重要的，否则极有可能出现死锁，在ConcurrentHashMap内部，段数组是final的，并且其成员变量实际上也是final的，但是，仅仅是将数组声明为final的并不保证数组成员也是final的，这需要实现上的保证。这可以确保不会出现死锁，因为获得锁的顺序是固定的。</p> 
<span id="OSC_h2_1"></span>
<h2 id="h2_1">1. 实现原理</h2> 
<p>ConcurrentHashMap使用分段锁技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问，能够实现真正的并发访问。如下图是ConcurrentHashMap的内部结构图：</p> 
<p>&nbsp;</p> 
<p><img alt="" src="http://static.oschina.net/uploads/space/2016/0317/154925_NLV0_2243330.png"></p> 
<p style="text-align:justify">从图中可以看到，ConcurrentHashMap内部分为很多个Segment，每一个Segment拥有一把锁，然后每个Segment（继承<span style="color:#E53333">ReentrantLock</span>）</p> 
<pre><code class="hljs java"><span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Segment</span>&lt;<span class="hljs-title">K</span>,<span class="hljs-title">V</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">ReentrantLock</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span></span></code></pre> 
<p style="text-align:justify">Segment继承了ReentrantLock，表明每个segment都可以当做一个锁。（ReentrantLock前文已经提到，不了解的话就把当做synchronized的替代者吧）这样对每个segment中的数据需要同步操作的话都是使用每个segment容器对象自身的锁来实现。只有对全局需要改变时锁定的是所有的segment。</p> 
<p style="text-align:justify"><span style="font-size:13.3333px">Segment</span>下面包含很多个HashEntry列表数组。对于一个key，需要经过<span style="color:#E53333">三次</span>（为什么要hash三次下文会详细讲解）hash操作，才能最终定位这个元素的位置，这三次hash分别为：</p> 
<ol> 
 <li>对于一个key，先进行一次hash操作，得到hash值h1，也即h1 = hash1(key)；</li> 
 <li>将得到的h1的高几位进行第二次hash，得到hash值h2，也即h2 = hash2(h1高几位)，通过h2能够确定该元素的放在哪个Segment；</li> 
 <li>将得到的h1进行第三次hash，得到hash值h3，也即h3 = hash3(h1)，通过h3能够确定该元素放置在哪个HashEntry。</li> 
</ol> 
<p style="text-align:justify">ConcurrentHashMap中主要实体类就是三个：ConcurrentHashMap（整个Hash表）,Segment（桶），HashEntry（节点），对应上面的图可以看出之间的关系</p> 
<pre><code class="hljs php"><span class="hljs-comment">/** 
* The segments, each of which is a specialized hash table 
*/</span>  
<span class="hljs-keyword">final</span> Segment&lt;K,V&gt;[] segments;</code></pre> 
<p>不变(Immutable)和易变(Volatile)ConcurrentHashMap完全允许多个读操作并发进行，读操作并不需要加锁。如果使用传统的技术，如HashMap中的实现，如果允许可以在hash链的中间添加或删除元素，读操作不加锁将得到不一致的数据。ConcurrentHashMap实现技术是保证HashEntry几乎是不可变的。HashEntry代表每个hash链中的一个节点，其结构如下所示：</p> 
<pre><code class="hljs java"><span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HashEntry</span>&lt;<span class="hljs-title">K</span>,<span class="hljs-title">V</span>&gt; </span>{  
     <span class="hljs-keyword">final</span> K key;  
     <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> hash;  
     <span class="hljs-keyword">volatile</span> V value;  
     <span class="hljs-keyword">volatile</span> HashEntry&lt;K,V&gt; next;  
 }</code></pre> 
<p>在JDK 1.6中，HashEntry中的next指针也定义为final，并且每次插入将新添加节点作为链的头节点（同HashMap实现），而且每次删除一个节点时，会将删除节点之前的所有节点 <span style="color:#E53333">拷贝一份组成一个新的链</span>，而将当前节点的上一个节点的next指向当前节点的下一个节点，从而在删除以后 <span style="color:#E53333">有两条链存在</span>，因而可以保证即使在同一条链中，有一个线程在删除，而另一个线程在遍历，它们都能工作良好，因为遍历的线程能继续使用原有的链。因而这种实现是一种更加细粒度的happens-before关系，即如果遍历线程在删除线程结束后开始，则它能看到删除后的变化，如果它发生在删除线程正在执行中间，则它会使用原有的链，而不会等到删除线程结束后再执行，即看不到删除线程的影响。如果这不符合你的需求，还是乖乖的用Hashtable或HashMap的synchronized版本，Collections.synchronizedMap()做的包装。</p> 
<p style="text-align:justify">而HashMap中的Entry只有key是final的</p> 
<pre><code class="hljs ruby">static <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Entry</span>&lt;K,<span class="hljs-title">V</span>&gt; <span class="hljs-title">implements</span> <span class="hljs-title">Map</span>.<span class="hljs-title">Entry</span>&lt;K,<span class="hljs-title">V</span>&gt; {</span>
        final K key;
        V value;
        Entry&lt;K,V&gt; <span class="hljs-keyword">next</span>;
        int hash;</code></pre> 
<p><strong>不变</strong> <strong>模式（immutable）</strong>是多线程安全里最简单的一种保障方式。因为你拿他没有办法，想改变它也没有机会。<br> 不变模式主要通过final关键字来限定的。在JMM中final关键字还有特殊的语义。Final域使得确保初始化安全性（initialization safety）成为可能，初始化安全性让不可变形对象不需要同步就能自由地被访问和共享。</p> 
<span id="OSC_h3_2"></span>
<h3 id="h3_2">1.1&nbsp;初始化</h3> 
<p>先看看ConcurrentHashMap的初始化做了哪些事情，构造函数的源码如下：</p> 
<pre><code class="hljs java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ConcurrentHashMap</span><span class="hljs-params">(<span class="hljs-keyword">int</span> initialCapacity,
                             <span class="hljs-keyword">float</span> loadFactor, <span class="hljs-keyword">int</span> concurrencyLevel)</span> </span>{
        <span class="hljs-keyword">if</span> (!(loadFactor &gt; <span class="hljs-number">0</span>) || initialCapacity &lt; <span class="hljs-number">0</span> || concurrencyLevel &lt;= <span class="hljs-number">0</span>)
            <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException();
        <span class="hljs-keyword">if</span> (concurrencyLevel &gt; MAX_SEGMENTS)
            concurrencyLevel = MAX_SEGMENTS;
        <span class="hljs-comment">// Find power-of-two sizes best matching arguments</span>
        <span class="hljs-keyword">int</span> sshift = <span class="hljs-number">0</span>;
        <span class="hljs-keyword">int</span> ssize = <span class="hljs-number">1</span>;
        <span class="hljs-keyword">while</span> (ssize &lt; concurrencyLevel) {
            ++sshift;
            ssize &lt;&lt;= <span class="hljs-number">1</span>;
        }
        <span class="hljs-keyword">this</span>.segmentShift = <span class="hljs-number">32</span> - sshift;
        <span class="hljs-keyword">this</span>.segmentMask = ssize - <span class="hljs-number">1</span>;
        <span class="hljs-keyword">if</span> (initialCapacity &gt; MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        <span class="hljs-keyword">int</span> c = initialCapacity / ssize;
        <span class="hljs-keyword">if</span> (c * ssize &lt; initialCapacity)
            ++c;
        <span class="hljs-keyword">int</span> cap = MIN_SEGMENT_TABLE_CAPACITY;
        <span class="hljs-keyword">while</span> (cap &lt; c)
            cap &lt;&lt;= <span class="hljs-number">1</span>;
        <span class="hljs-comment">// create segments and segments[0]</span>
        Segment&lt;K,V&gt; s0 =
            <span class="hljs-keyword">new</span> Segment&lt;K,V&gt;(loadFactor, (<span class="hljs-keyword">int</span>)(cap * loadFactor),
                             (HashEntry&lt;K,V&gt;[])<span class="hljs-keyword">new</span> HashEntry[cap]);
        Segment&lt;K,V&gt;[] ss = (Segment&lt;K,V&gt;[])<span class="hljs-keyword">new</span> Segment[ssize];
        UNSAFE.putOrderedObject(ss, SBASE, s0); <span class="hljs-comment">// ordered write of segments[0]</span>
        <span class="hljs-keyword">this</span>.segments = ss;
    }</code></pre> 
<p style="text-align:justify">传入的参数有initialCapacity，loadFactor，concurrencyLevel这三个。</p> 
<ul> 
 <li>initialCapacity表示新创建的这个ConcurrentHashMap的初始容量，也就是上面的结构图中的Entry数量。默认值为static final int DEFAULT_INITIAL_CAPACITY = 16;</li> 
 <li>loadFactor表示负载因子，就是当ConcurrentHashMap中的元素个数大于loadFactor * 最大容量时就需要rehash，扩容。默认值为static final float DEFAULT_LOAD_FACTOR = 0.75f;</li> 
 <li>concurrencyLevel表示并发级别，这个值用来确定Segment的个数，Segment的个数是大于等于concurrencyLevel的第一个2的n次方的数。比如，如果concurrencyLevel为12，13，14，15，16这些数，则Segment的数目为16(2的4次方)。默认值为static final int DEFAULT_CONCURRENCY_LEVEL = 16;。理想情况下ConcurrentHashMap的真正的并发访问量能够达到concurrencyLevel，因为有concurrencyLevel个Segment，假如有concurrencyLevel个线程需要访问Map，并且需要访问的数据都恰好分别落在不同的Segment中，则这些线程能够无竞争地自由访问（因为他们不需要竞争同一把锁），达到同时访问的效果。这也是为什么这个参数起名为“并发级别”的原因。</li> 
</ul> 
<p style="text-align:justify">初始化的一些动作：</p> 
<ol> 
 <li>验证参数的合法性，如果不合法，直接抛出异常。</li> 
 <li>concurrencyLevel也就是Segment的个数不能超过规定的最大Segment的个数，默认值为static final int MAX_SEGMENTS = 1 &lt;&lt; 16;，如果超过这个值，设置为这个值。</li> 
 <li>然后使用循环找到大于等于concurrencyLevel的第一个2的n次方的数ssize，这个数就是Segment数组的大小，并记录一共向左按位移动的次数sshift，并令segmentShift = 32 - sshift，并且segmentMask的值等于ssize - 1，segmentMask的各个二进制位都为1，目的是之后可以通过key的hash值与这个值做&amp;运算确定Segment的索引。</li> 
 <li>检查给的容量值是否大于允许的最大容量值，如果大于该值，设置为该值。最大容量值为static final int MAXIMUM_CAPACITY = 1 &lt;&lt; 30;。</li> 
 <li>然后计算每个Segment平均应该放置多少个元素，这个值c是向上取整的值。比如初始容量为15，Segment个数为4，则每个Segment平均需要放置4个元素。</li> 
 <li>最后创建一个Segment实例，将其当做Segment数组的第一个元素。</li> 
</ol> 
<span id="OSC_h3_3"></span>
<h3 id="h3_3">1.2&nbsp;put操作</h3> 
<p>put操作的源码如下：</p> 
<pre><code class="hljs cs"><span class="hljs-function"><span class="hljs-keyword">public</span> V <span class="hljs-title">put</span>(<span class="hljs-params">K key, V <span class="hljs-keyword">value</span></span>) </span>{
      Segment&lt;K,V&gt; s;
      <span class="hljs-keyword">if</span> (<span class="hljs-keyword">value</span> == <span class="hljs-literal">null</span>)
          <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> NullPointerException();
      <span class="hljs-keyword">int</span> hash = hash(key);
      <span class="hljs-keyword">int</span> j = (hash &gt;&gt;&gt; segmentShift) &amp; segmentMask;
      <span class="hljs-keyword">if</span> ((s = (Segment&lt;K,V&gt;)UNSAFE.getObject          <span class="hljs-comment">// nonvolatile; recheck</span>
           (segments, (j &lt;&lt; SSHIFT) + SBASE)) == <span class="hljs-literal">null</span>) <span class="hljs-comment">//  in ensureSegment</span>
          s = ensureSegment(j);
      <span class="hljs-keyword">return</span> s.put(key, hash, <span class="hljs-keyword">value</span>, <span class="hljs-literal">false</span>);
  }</code></pre> 
<p style="text-align:justify">操作步骤如下：</p> 
<ol> 
 <li>判断value是否为null，如果为null，直接抛出异常。</li> 
 <li>key通过一次hash运算得到一个hash值。(这个hash运算下文详说)</li> 
 <li>将得到hash值向右按位移动segmentShift位，然后再与segmentMask做&amp;运算得到segment的索引j。<br> 在初始化的时候我们说过segmentShift的值等于32-sshift，例如concurrencyLevel等于16，则sshift等于4，则segmentShift为28。hash值是一个32位的整数，将其向右移动28位就变成这个样子：<br> 0000 0000 0000 0000 0000 0000 0000 xxxx，然后再用这个值与segmentMask做&amp;运算，也就是取最后四位的值。这个值确定Segment的索引。</li> 
 <li>使用Unsafe的方式从Segment数组中获取该索引对应的Segment对象。</li> 
 <li>向这个Segment对象中put值，这个put操作也基本是一样的步骤（通过&amp;运算获取HashEntry的索引，然后set）。</li> 
</ol> 
<pre><code class="hljs cs"><span class="hljs-function">final V <span class="hljs-title">put</span>(<span class="hljs-params">K key, <span class="hljs-keyword">int</span> hash, V <span class="hljs-keyword">value</span>, boolean onlyIfAbsent</span>) </span>{
            HashEntry&lt;K,V&gt; node = tryLock() ? <span class="hljs-literal">null</span> :
                scanAndLockForPut(key, hash, <span class="hljs-keyword">value</span>);
            V oldValue;
            <span class="hljs-keyword">try</span> {
                HashEntry&lt;K,V&gt;[] tab = table;
                <span class="hljs-keyword">int</span> index = (tab.length - <span class="hljs-number">1</span>) &amp; hash;
                HashEntry&lt;K,V&gt; first = entryAt(tab, index);
                <span class="hljs-keyword">for</span> (HashEntry&lt;K,V&gt; e = first;;) {
                    <span class="hljs-keyword">if</span> (e != <span class="hljs-literal">null</span>) {
                        K k;
                        <span class="hljs-keyword">if</span> ((k = e.key) == key ||
                            (e.hash == hash &amp;&amp; key.<span class="hljs-keyword">equals</span>(k))) {
                            oldValue = e.<span class="hljs-keyword">value</span>;
                            <span class="hljs-keyword">if</span> (!onlyIfAbsent) {
                                e.<span class="hljs-keyword">value</span> = <span class="hljs-keyword">value</span>;
                                ++modCount;
                            }
                            <span class="hljs-keyword">break</span>;
                        }
                        e = e.next;
                    }
                    <span class="hljs-keyword">else</span> {
                        <span class="hljs-keyword">if</span> (node != <span class="hljs-literal">null</span>)
                            node.setNext(first);
                        <span class="hljs-keyword">else</span>
                            node = <span class="hljs-keyword">new</span> HashEntry&lt;K,V&gt;(hash, key, <span class="hljs-keyword">value</span>, first);
                        <span class="hljs-keyword">int</span> c = count + <span class="hljs-number">1</span>;
                        <span class="hljs-keyword">if</span> (c &gt; threshold &amp;&amp; tab.length &lt; MAXIMUM_CAPACITY)
                            rehash(node);
                        <span class="hljs-keyword">else</span>
                            setEntryAt(tab, index, node);
                        ++modCount;
                        count = c;
                        oldValue = <span class="hljs-literal">null</span>;
                        <span class="hljs-keyword">break</span>;
                    }
                }
            } <span class="hljs-keyword">finally</span> {
                unlock();
            }
            <span class="hljs-keyword">return</span> oldValue;
        }</code></pre> 
<p>put操作是要加锁的。</p> 
<span id="OSC_h3_4"></span>
<h3 id="h3_4">1.3 get操作</h3> 
<p style="text-align:justify">get操作的源码如下：</p> 
<pre><code class="hljs cs"><span class="hljs-function"><span class="hljs-keyword">public</span> V <span class="hljs-title">get</span>(<span class="hljs-params">Object key</span>) </span>{
        Segment&lt;K,V&gt; s; <span class="hljs-comment">// manually integrate access methods to reduce overhead</span>
        HashEntry&lt;K,V&gt;[] tab;
        <span class="hljs-keyword">int</span> h = hash(key);
        <span class="hljs-keyword">long</span> u = (((h &gt;&gt;&gt; segmentShift) &amp; segmentMask) &lt;&lt; SSHIFT) + SBASE;
        <span class="hljs-keyword">if</span> ((s = (Segment&lt;K,V&gt;)UNSAFE.getObjectVolatile(segments, u)) != <span class="hljs-literal">null</span> &amp;&amp;
            (tab = s.table) != <span class="hljs-literal">null</span>) {
            <span class="hljs-keyword">for</span> (HashEntry&lt;K,V&gt; e = (HashEntry&lt;K,V&gt;) UNSAFE.getObjectVolatile
                     (tab, ((<span class="hljs-keyword">long</span>)(((tab.length - <span class="hljs-number">1</span>) &amp; h)) &lt;&lt; TSHIFT) + TBASE);
                 e != <span class="hljs-literal">null</span>; e = e.next) {
                K k;
                <span class="hljs-keyword">if</span> ((k = e.key) == key || (e.hash == h &amp;&amp; key.<span class="hljs-keyword">equals</span>(k)))
                    <span class="hljs-keyword">return</span> e.<span class="hljs-keyword">value</span>;
            }
        }
        <span class="hljs-keyword">return</span> <span class="hljs-literal">null</span>;
    }</code></pre> 
<p style="text-align:justify">操作步骤为：</p> 
<ol> 
 <li>和put操作一样，先通过key进行两次hash确定应该去哪个Segment中取数据。</li> 
 <li>使用Unsafe获取对应的Segment，然后再进行一次&amp;运算得到HashEntry链表的位置，然后从链表头开始遍历整个链表（因为Hash可能会有碰撞，所以用一个链表保存），如果找到对应的key，则返回对应的value值，如果链表遍历完都没有找到对应的key，则说明Map中不包含该key，返回null。</li> 
</ol> 
<p style="text-align:justify">值得注意的是，<span style="color:#E53333">get操作是不需要加锁的（如果value为null，会调用</span><span style="color:#FF0000">readValueUnderLock，只有这个步骤会加锁</span><span style="color:#E53333">）<span style="color:#000000">，通过前面提到的volatile和final来确保数据安全。</span></span></p> 
<span id="OSC_h3_5"></span>
<h3 id="h3_5">1.4&nbsp;size操作</h3> 
<p>size操作与put和get操作最大的区别在于，size操作需要遍历所有的Segment才能算出整个Map的大小，而put和get都只关心一个Segment。假设我们当前遍历的Segment为SA，那么在遍历SA过程中其他的Segment比如SB可能会被修改，于是这一次运算出来的size值可能并不是Map当前的真正大小。所以一个比较简单的办法就是计算Map大小的时候所有的Segment都Lock住，不能更新(包含put，remove等等)数据，计算完之后再Unlock。这是普通人能够想到的方案，但是牛逼的作者还有一个更好的Idea：<span style="color:#E53333">先给3次机会</span>，不lock所有的Segment，遍历所有Segment，累加各个Segment的大小得到整个Map的大小，如果某相邻的两次计算获取的所有Segment的更新的次数（每个Segment都有一个modCount变量，这个变量在Segment中的Entry被修改时会加一，通过这个值可以得到每个Segment的更新操作的次数）是一样的，说明计算过程中没有更新操作，则直接返回这个值。如果这三次不加锁的计算过程中Map的更新次数有变化，则之后的计算先对所有的Segment加锁，再遍历所有Segment计算Map大小，最后再解锁所有Segment。源代码如下：</p> 
<pre><code class="hljs java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">size</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-comment">// Try a few times to get accurate count. On failure due to</span>
        <span class="hljs-comment">// continuous async changes in table, resort to locking.</span>
        <span class="hljs-keyword">final</span> Segment&lt;K,V&gt;[] segments = <span class="hljs-keyword">this</span>.segments;
        <span class="hljs-keyword">int</span> size;
        <span class="hljs-keyword">boolean</span> overflow; <span class="hljs-comment">// true if size overflows 32 bits</span>
        <span class="hljs-keyword">long</span> sum;         <span class="hljs-comment">// sum of modCounts</span>
        <span class="hljs-keyword">long</span> last = <span class="hljs-number">0L</span>;   <span class="hljs-comment">// previous sum</span>
        <span class="hljs-keyword">int</span> retries = -<span class="hljs-number">1</span>; <span class="hljs-comment">// first iteration isn't retry</span>
        <span class="hljs-keyword">try</span> {
            <span class="hljs-keyword">for</span> (;;) {
                <span class="hljs-keyword">if</span> (retries++ == RETRIES_BEFORE_LOCK) {
                    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>; j &lt; segments.length; ++j)
                        ensureSegment(j).lock(); <span class="hljs-comment">// force creation</span>
                }
                sum = <span class="hljs-number">0L</span>;
                size = <span class="hljs-number">0</span>;
                overflow = <span class="hljs-keyword">false</span>;
                <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>; j &lt; segments.length; ++j) {
                    Segment&lt;K,V&gt; seg = segmentAt(segments, j);
                    <span class="hljs-keyword">if</span> (seg != <span class="hljs-keyword">null</span>) {
                        sum += seg.modCount;
                        <span class="hljs-keyword">int</span> c = seg.count;
                        <span class="hljs-keyword">if</span> (c &lt; <span class="hljs-number">0</span> || (size += c) &lt; <span class="hljs-number">0</span>)
                            overflow = <span class="hljs-keyword">true</span>;
                    }
                }
                <span class="hljs-keyword">if</span> (sum == last)
                    <span class="hljs-keyword">break</span>;
                last = sum;
            }
        } <span class="hljs-keyword">finally</span> {
            <span class="hljs-keyword">if</span> (retries &gt; RETRIES_BEFORE_LOCK) {
                <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>; j &lt; segments.length; ++j)
                    segmentAt(segments, j).unlock();
            }
        }
        <span class="hljs-keyword">return</span> overflow ? Integer.MAX_VALUE : size;
    }</code></pre> 
<p style="text-align:justify">举个例子：</p> 
<blockquote> 
 <p>一个Map有4个Segment，标记为S1，S2，S3，S4，现在我们要获取Map的size。计算过程是这样的：第一次计算，不对S1，S2，S3，S4加锁，遍历所有的Segment，假设每个Segment的大小分别为1，2，3，4，更新操作次数分别为：2，2，3，1，则这次计算可以得到Map的总大小为1+2+3+4=10，总共更新操作次数为2+2+3+1=8；第二次计算，不对S1,S2,S3,S4加锁，遍历所有Segment，假设这次每个Segment的大小变成了2，2，3，4，更新次数分别为3，2，3，1，因为两次计算得到的Map更新次数不一致(第一次是8，第二次是9)则可以断定这段时间Map数据被更新，则此时应该再试一次；第三次计算，不对S1，S2，S3，S4加锁，遍历所有Segment，假设每个Segment的更新操作次数还是为3，2，3，1，则因为第二次计算和第三次计算得到的Map的更新操作的次数是一致的，就能说明第二次计算和第三次计算这段时间内Map数据没有被更新，此时可以直接返回第三次计算得到的Map的大小。最坏的情况：第三次计算得到的数据更新次数和第二次也不一样，则只能先对所有Segment加锁再计算最后解锁。</p> 
</blockquote> 
<span id="OSC_h3_6"></span>
<h3 id="h3_6">1.5&nbsp;containsValue操作</h3> 
<p>containsValue操作采用了和size操作一样的想法:</p> 
<pre><code class="hljs cs"><span class="hljs-function"><span class="hljs-keyword">public</span> boolean <span class="hljs-title">containsValue</span>(<span class="hljs-params">Object <span class="hljs-keyword">value</span></span>) </span>{
        <span class="hljs-comment">// Same idea as size()</span>
        <span class="hljs-keyword">if</span> (<span class="hljs-keyword">value</span> == <span class="hljs-literal">null</span>)
            <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> NullPointerException();
        final Segment&lt;K,V&gt;[] segments = <span class="hljs-keyword">this</span>.segments;
        boolean found = <span class="hljs-literal">false</span>;
        <span class="hljs-keyword">long</span> last = <span class="hljs-number">0</span>;
        <span class="hljs-keyword">int</span> retries = <span class="hljs-number">-1</span>;
        <span class="hljs-keyword">try</span> {
            outer: <span class="hljs-keyword">for</span> (;;) {
                <span class="hljs-keyword">if</span> (retries++ == RETRIES_BEFORE_LOCK) {
                    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>; j &lt; segments.length; ++j)
                        ensureSegment(j).<span class="hljs-keyword">lock</span>(); <span class="hljs-comment">// force creation</span>
                }
                <span class="hljs-keyword">long</span> hashSum = <span class="hljs-number">0</span>L;
                <span class="hljs-keyword">int</span> sum = <span class="hljs-number">0</span>;
                <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>; j &lt; segments.length; ++j) {
                    HashEntry&lt;K,V&gt;[] tab;
                    Segment&lt;K,V&gt; seg = segmentAt(segments, j);
                    <span class="hljs-keyword">if</span> (seg != <span class="hljs-literal">null</span> &amp;&amp; (tab = seg.table) != <span class="hljs-literal">null</span>) {
                        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span> ; i &lt; tab.length; i++) {
                            HashEntry&lt;K,V&gt; e;
                            <span class="hljs-keyword">for</span> (e = entryAt(tab, i); e != <span class="hljs-literal">null</span>; e = e.next) {
                                V v = e.<span class="hljs-keyword">value</span>;
                                <span class="hljs-keyword">if</span> (v != <span class="hljs-literal">null</span> &amp;&amp; <span class="hljs-keyword">value</span>.<span class="hljs-keyword">equals</span>(v)) {
                                    found = <span class="hljs-literal">true</span>;
                                    <span class="hljs-keyword">break</span> outer;
                                }
                            }
                        }
                        sum += seg.modCount;
                    }
                }
                <span class="hljs-keyword">if</span> (retries &gt; <span class="hljs-number">0</span> &amp;&amp; sum == last)
                    <span class="hljs-keyword">break</span>;
                last = sum;
            }
        } <span class="hljs-keyword">finally</span> {
            <span class="hljs-keyword">if</span> (retries &gt; RETRIES_BEFORE_LOCK) {
                <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> j = <span class="hljs-number">0</span>; j &lt; segments.length; ++j)
                    segmentAt(segments, j).unlock();
            }
        }
        <span class="hljs-keyword">return</span> found;
    }</code></pre> 
<span id="OSC_h2_7"></span>
<h2 id="h2_7">2.&nbsp;关于hash</h2> 
<p>看看hash的源代码：</p> 
<pre><code class="hljs java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> <span class="hljs-title">hash</span><span class="hljs-params">(Object k)</span> </span>{
        <span class="hljs-keyword">int</span> h = hashSeed;

        <span class="hljs-keyword">if</span> ((<span class="hljs-number">0</span> != h) &amp;&amp; (k <span class="hljs-keyword">instanceof</span> String)) {
            <span class="hljs-keyword">return</span> sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        <span class="hljs-comment">// Spread bits to regularize both segment and index locations,</span>
        <span class="hljs-comment">// using variant of single-word Wang/Jenkins hash.</span>
        h += (h &lt;&lt;  <span class="hljs-number">15</span>) ^ <span class="hljs-number">0xffffcd7d</span>;
        h ^= (h &gt;&gt;&gt; <span class="hljs-number">10</span>);
        h += (h &lt;&lt;   <span class="hljs-number">3</span>);
        h ^= (h &gt;&gt;&gt;  <span class="hljs-number">6</span>);
        h += (h &lt;&lt;   <span class="hljs-number">2</span>) + (h &lt;&lt; <span class="hljs-number">14</span>);
        <span class="hljs-keyword">return</span> h ^ (h &gt;&gt;&gt; <span class="hljs-number">16</span>);
    }</code></pre> 
<p>源码中的注释是这样的：</p> 
<blockquote>
 Applies a supplemental hash function to a given hashCode, which defends against poor quality hash functions. This is critical because ConcurrentHashMap uses power-of-two length hash tables, that otherwise encounter collisions for hashCodes that do not differ in lower or upper bits.
</blockquote> 
<p>这里用到了Wang/Jenkins hash算法的变种，主要的目的是为了减少哈希冲突，使元素能够均匀的分布在不同的Segment上，从而提高容器的存取效率。假如哈希的质量差到极点，那么所有的元素都在一个Segment中，不仅存取元素缓慢，分段锁也会失去意义。</p> 
<p>举个简单的例子：</p> 
<pre><code class="hljs css"><span class="hljs-selector-tag">System</span><span class="hljs-selector-class">.out</span><span class="hljs-selector-class">.println</span>(<span class="hljs-selector-tag">Integer</span><span class="hljs-selector-class">.parseInt</span>("0001111", 2) &amp; 15);
<span class="hljs-selector-tag">System</span><span class="hljs-selector-class">.out</span><span class="hljs-selector-class">.println</span>(<span class="hljs-selector-tag">Integer</span><span class="hljs-selector-class">.parseInt</span>("0011111", 2) &amp; 15);
<span class="hljs-selector-tag">System</span><span class="hljs-selector-class">.out</span><span class="hljs-selector-class">.println</span>(<span class="hljs-selector-tag">Integer</span><span class="hljs-selector-class">.parseInt</span>("0111111", 2) &amp; 15);
<span class="hljs-selector-tag">System</span><span class="hljs-selector-class">.out</span><span class="hljs-selector-class">.println</span>(<span class="hljs-selector-tag">Integer</span><span class="hljs-selector-class">.parseInt</span>("1111111", 2) &amp; 15);</code></pre> 
<p>这些数字得到的hash值都是一样的，全是15，所以如果不进行第一次预hash，发生冲突的几率还是很大的，但是如果我们先把上例中的二进制数字使用hash()函数先进行一次预hash，得到的结果是这样的：</p> 
<blockquote>
 0100｜0111｜0110｜0111｜1101｜1010｜0100｜1110 1111｜0111｜0100｜0011｜0000｜0001｜1011｜1000 0111｜0111｜0110｜1001｜0100｜0110｜0011｜1110 1000｜0011｜0000｜0000｜1100｜1000｜0001｜1010
</blockquote> 
<p>上面这个例子引用自:&nbsp; <a href="http://www.infoq.com/cn/articles/ConcurrentHashMap/" target="_blank" rel="nofollow">InfoQ</a></p> 
<p>可以看到每一位的数据都散开了，并且ConcurrentHashMap中是使用预hash值的高位参与运算的。比如之前说的先将hash值向右按位移动28位，再与15做&amp;运算，得到的结果都别为：4，15，7，8，没有冲突！</p> 
<span id="OSC_h2_8"></span>
<h2 id="h2_8">3.&nbsp;注意事项</h2> 
<ul> 
 <li>ConcurrentHashMap中的key和value值都不能为null，HashMap中key可以为null，HashTable中key不能为null。</li> 
 <li>ConcurrentHashMap是线程安全的类并不能保证使用了ConcurrentHashMap的操作都是线程安全的！</li> 
 <li><span style="font-size:13.3333px">ConcurrentHashMap的get操作不需要加锁，put操作需要加锁</span></li> 
</ul> 
<span id="OSC_h2_9"></span>
<h2 id="h2_9">Reference：</h2> 
<p>1. http://www.cnblogs.com/ITtangtang/p/3948786.html</p> 
<p>2.&nbsp;http://qifuguang.me/2015/09/10/[Java%E5%B9%B6%E5%8F%91%E5%8C%85%E5%AD%A6%E4%B9%A0%E5%85%AB]%E6%B7%B1%E5%BA%A6%E5%89%96%E6%9E%90ConcurrentHashMap/</p> 
<p>3.&nbsp;http://www.cnblogs.com/yydcdut/p/3959815.html</p>
                                                    <div class="ad-wrap">
                                                        <div data-traceid="blog_down_1" data-tracepid="blog_down" style="text-align:center">
    <!-- oschina-blog-728x90 -->
    <ins class="adsbygoogle" style="display:inline-block;width:728px;height:90px" data-ad-client="ca-pub-7090564139599510" data-ad-slot="5590362768"></ins>
    <script>
        (adsbygoogle = window.adsbygoogle || []).push({});
    </script>
    <script type="text/javascript">
        function googleAdJSAtOnload() {
            var element = document.createElement("script");
            element.src = "//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js";
            element.async = true;
            document.body.appendChild(element);
        }
        if (window.addEventListener) {
            window.addEventListener("load", googleAdJSAtOnload, false);
        } else if (window.attachEvent) {
            window.attachEvent("onload", googleAdJSAtOnload);
        } else {
            window.onload = googleAdJSAtOnload;
        }
    </script>
</div>
                                        </div>
                                            </div>



