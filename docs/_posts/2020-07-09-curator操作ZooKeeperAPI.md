---
layout: post
title: "curator操作ZooKeeperAPI"
date: 2020-07-09T18:58:22+08:00
---
## pom坐标
```
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-client</artifactId>
            <version>5.1.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>5.1.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>5.1.0</version>
        </dependency>
```
## 代码
```java
package ink.cyz;

import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.api.BackgroundCallback;
import org.apache.curator.framework.api.CuratorEvent;
import org.apache.curator.framework.recipes.cache.*;
import org.apache.curator.framework.recipes.locks.InterProcessLock;
import org.apache.curator.framework.recipes.locks.InterProcessMultiLock;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.framework.recipes.locks.InterProcessReadWriteLock;
import org.apache.curator.retry.RetryOneTime;
import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.ZooDefs;
import org.apache.zookeeper.data.ACL;
import org.apache.zookeeper.data.Id;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.util.ArrayList;
import java.util.List;

public class CuratorConnection {
    CuratorFramework client;
    @Before
    public  void Before() {
        client = CuratorFrameworkFactory.builder()
                .connectString("localhost:2182,localhost:2183,localhost:2184")
                .sessionTimeoutMs(5000)
                .retryPolicy(new RetryOneTime(3000))
                .namespace("create")
                .build();
        client.start();
        System.out.println(client.isStarted());
    }
    @After
    public void after(){
        client.close();
    }
    @Test
    public void createWatcher1() throws Exception {
        client.create().withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .forPath("/watcher1","node1".getBytes());
        System.out.println("watcher1 created");
    }

    @Test
    public void create1() throws Exception {
        client.create().withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .forPath("/node1","node1".getBytes());
        System.out.println("node1 created");
    }
    @Test
    public void create2() throws Exception {
        List<ACL> list = new ArrayList<>();
        Id id = new Id("ip","127.0.0.1");
        list.add(new ACL(ZooDefs.Perms.ALL,id));
        client.create().withMode(CreateMode.PERSISTENT).withACL(list).forPath("/node2","node2".getBytes());
        System.out.println("node2 created");
    }
    @Test
    public void create3() throws Exception{
        //recursively create node
        client.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .forPath("/node3/node31","node31".getBytes());
        System.out.println("node31 created");
    }

    @Test
    public void create4() throws Exception{
        client.create().creatingParentsIfNeeded().withMode(CreateMode.PERSISTENT)
                .withACL(ZooDefs.Ids.OPEN_ACL_UNSAFE)
                .inBackground(new BackgroundCallback() {
                    @Override
                    public void processResult(CuratorFramework curatorFramework, CuratorEvent curatorEvent) throws Exception {
                        System.out.println(curatorEvent.getType());
                        System.out.println(curatorEvent.getPath());
                    }
                }).forPath("/node4/node41","node41".getBytes());
        Thread.sleep(5000);
    }

    @Test
    public void watcher1() throws Exception {
        final NodeCache nodeCache = new NodeCache(client, "/watcher1");
        nodeCache.start();
        nodeCache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                System.out.println(nodeCache.getPath());
                System.out.println(nodeCache.getCurrentData());
            }
        });
        Thread.sleep(100000);
        nodeCache.close();
    }

    @Test
    public void watcher2() throws Exception {
        final PathChildrenCache nodeCache = new PathChildrenCache(client, "/watcher1",true);
        nodeCache.start();
        nodeCache.getListenable().addListener(new PathChildrenCacheListener() {
            @Override
            public void childEvent(CuratorFramework client, PathChildrenCacheEvent event) throws Exception {
                System.out.println(event.getData());
                System.out.println(event.getType());
            }
        });
        Thread.sleep(100000);
        nodeCache.close();
    }

    @Test
    //开启事务处理
    public void tra1() throws Exception{
        client.inTransaction().create()
        .forPath("/node1","node1".getBytes())
                .and().
        setData().forPath("/node2","node2".getBytes())
                .and().commit();
    }

    //分布式锁
    //排他锁
    @Test
    public void lock1() throws Exception {
        InterProcessLock interProcessLock = new InterProcessMutex(client,"/lock1");
        System.out.println("等待获取锁对象");
        interProcessLock.acquire();
        for (int i = 0; i < 10; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        System.out.println("等待释放锁对象");
        interProcessLock.release();
    }

    @Test
    public void lock2() throws Exception {
        InterProcessReadWriteLock lock = new InterProcessReadWriteLock(client,"/lock1");
        InterProcessLock readLock = lock.readLock();
        readLock.acquire();
        for (int i = 0; i < 10; i++) {
            Thread.sleep(3000);
            System.out.println(i);
        }
        readLock.release();
    }
}

```
## ZooKeeper的应用

以下内容参考[ZooKeeper的三种典型应用场景](https://www.cnblogs.com/jian0110/p/10650396.html)

> 节点类型和Watcher监听机制，是解决所有应用场景问题的出发点和落脚点。


ZooKeeper是典型的pub/sub模式的分布式数据管理与协调框架, 开发人员可以使用它进行分布式数据的发布与订阅. 另外, 其丰富的数据节点类型(ephemeral, persistent, ephemeral-sequential, persistent-sequential)可以交叉使用, 配合Watcher事件通知机制, 可以用应用于分布式都会涉及的一些核心功能:
- 数据发布/订阅
- Master选举
- 命名服务
- 分布式协调/通知
- 集群管理
- 分布式锁
- 分布式队列


1. 数据发布/订阅
```java
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.cache.NodeCache;
import org.apache.curator.framework.recipes.cache.NodeCacheListener;
import org.apache.curator.retry.RetryOneTime;
import org.apache.zookeeper.CreateMode;

import java.util.concurrent.CountDownLatch;

public class Sub {
    private static final String ADDRESS = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
    private static CountDownLatch countDownLatch = new CountDownLatch(4);
    private static final String PATH = "/configs";
    private static String config = "jdbc_connection";
    private static void init() throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(ADDRESS)
                .sessionTimeoutMs(5000)
                .retryPolicy(new RetryOneTime(2000))
                .build();
        client.start();
        if (client.checkExists().forPath(PATH) == null){
            client.create().creatingParentsIfNeeded()
                    .withMode(CreateMode.EPHEMERAL)
                    .forPath(PATH, config.getBytes());
        }
    }
    private static void publish(CuratorFramework client, String znode) throws Exception {
        client.setData().forPath(PATH, "configuration".getBytes());
        countDownLatch.await();
    }
    private static void subscribe(CuratorFramework clinet, String znoe) throws Exception{
        final NodeCache cache = new NodeCache(clinet,PATH);
        cache.start(true);
        countDownLatch.countDown();
        cache.getListenable().addListener(new NodeCacheListener() {
            @Override
            public void nodeChanged() throws Exception {
                System.out.println("configuration changed");
            }
        });
    }
}

```
2. Master选举
>  在一些读写分离的应用场景中，客户端写请求往往是由Master处理的，而另一些场景中，Master则常常负责处理一些复杂的逻辑，并将处理结果同步给集群中其它系统单元。比如一个广告投放系统后台与ZooKeeper交互，广告ID通常都是经过一系列海量数据处理中计算得到（非常消耗I/O和CPU资源的过程），那就可以只让集群中一台机器处理数据得到计算结果，之后就可以共享给整个集群中的其它所有客户端机器。
> 利用ZooKeeper的特性：利用ZooKeeper的强一致性，即能够很好地保证分布式高并发情况下节点的创建一定能够保证全局唯一性，ZooKeeper将会保证客户端无法重复创建一个已经存在的数据节点，也就是说如果多个客户端请求创建同一个节点，那么最终一定只有一个客户端请求能够创建成功，这个客户端就是Master，而其它客户端注在该节点上注册子节点Wacther，用于监控当前Master是否存活，如果当前Master挂了，那么其余客户端立马重新进行Master选举。
> 竞争成为Master角色之后，创建的子节点都是临时顺序节点，比如：_c_862cf0ce-6712-4aef-a91d-fc4c1044d104-lock-0000000001，并且序号是递增的。需要注意的是这里有"lock"单词，这说明ZooKeeper这一特性，也可以运用于分布式锁。
```java
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.leader.LeaderSelector;
import org.apache.curator.framework.recipes.leader.LeaderSelectorListenerAdapter;
import org.apache.curator.retry.RetryOneTime;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.atomic.AtomicInteger;

public class Master {
    private static final String ADDRESS = "127.0.0.1:2182,127.0.0.1:2183,127.0.0.1:2184";
    private static CountDownLatch countDownLatch = new CountDownLatch(4);
    private static final String PATH = "/configs";
    private static String config = "jdbc_connection";

    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            service.submit(()->{
                masterSelect(finalI +"");
            });
        }
    }
    public static void masterSelect(final String znode) {
        AtomicInteger leaderCount = new AtomicInteger(1);
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(ADDRESS)
                .sessionTimeoutMs(5000)
                .retryPolicy(new RetryOneTime(5000))
                .build();
        client.start();
        LeaderSelector selector = new LeaderSelector(client, PATH, new LeaderSelectorListenerAdapter() {
            @Override
            public void takeLeadership(CuratorFramework client) throws Exception {
                leaderCount.getAndIncrement();
                System.out.println("znode: " + znode + "第" + countDownLatch.getCount() + "次成为leader");
            }
        });
        // autoRequeue自动重新排队：使得上一次选举为master的节点还有可能再次成为master
        selector.autoRequeue();
        selector.start();
    }
}

```

3. 分布式锁
>  对于排他锁：ZooKeeper通过数据节点表示一个锁，例如/exclusive_lock/lock节点就可以定义一个锁，所有客户端都会调用create()接口，试图在/exclusive_lock下创建lock子节点，但是ZooKeeper的强一致性会保证所有客户端最终只有一个客户创建成功。也就可以认为获得了锁，其它线程Watcher监听子节点变化（等待释放锁，竞争获取资源）。
>  对于共享锁：ZooKeeper同样可以通过数据节点表示一个锁，类似于/shared_lock/[Hostname]-请求类型（读/写）-序号的临时节点，比如/shared_lock/192.168.0.1-R-0000000000

```java
package com.lijian.zookeeper.demo;

import org.apache.curator.RetryPolicy;
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.framework.recipes.locks.InterProcessMutex;
import org.apache.curator.retry.ExponentialBackoffRetry;

import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ZooKeeper_Lock {
    private static final String ADDRESS = "xxx.xxx.xxx.xxx:2181";
    private static final int SESSION_TIMEOUT = 5000;
    private static final String LOCK_PATH = "/lock_path";
    private static final int CLIENT_COUNT = 10;

    private static RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
    private static int resource = 0;

    public static void main(String[] args){
        ExecutorService service = Executors.newFixedThreadPool(CLIENT_COUNT);
        for (int i = 0; i < CLIENT_COUNT; i++) {
            final String index = String.valueOf(i);
            service.submit(() -> {
                distributedLock(index);
            });
        }
    }

    private static void distributedLock(final String znode) {
        CuratorFramework client = CuratorFrameworkFactory.builder()
                .connectString(ADDRESS)
                .sessionTimeoutMs(SESSION_TIMEOUT)
                .retryPolicy(retryPolicy)
                .build();
        client.start();
        final InterProcessMutex lock = new InterProcessMutex(client, LOCK_PATH);
        try {
//            lock.acquire();
            System.out.println("客户端节点[" + znode + "]获取lock");
            System.out.println("客户端节点[" + znode + "]读取的资源为：" + String.valueOf(resource));
            resource ++;
//            lock.release();
            System.out.println("客户端节点[" + znode + "]释放lock");

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
