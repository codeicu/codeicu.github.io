---
layout: post
title: "zookeeper学习"
date: 2020-07-04T11:05:17+08:00
draft: false
---
# 安装与配置
1. 使用的服务器是WSL, 首先重置一下WSL
```java
wslconfig /u
```
2. 下载安装包: java, zookeeper
3. tar -xzvf 解压安装
4. 在.bash_profile中配置Java环境, 并. .bash_profile
```
JAVA_HOME=/home/zink/jdk14
export JAVA_HOME

PATH=$JAVA_HOME/bin:$PATH
export PATH
```
5. 在配置文件conf/zoo.cfg中修改zookeeper data存放位置
6. 启动: bin/zkServer.sh start
7. 检测状态: bin/zkServer.sh status

# zookeeper常用Shell命令
## 新增节点
```
create [-s] [-e] path data ,其中-s为有序节点, -e 临时节点
[-s] [-e] 可共存
```
## 更新节点
`set path val [-v version]`
## 删除节点
```
delete path [-v version] 只能删除叶子节点
rmr path : 递归删除
```
## 查看节点
`get path [-s] [-w]` : -w为watch, 监听 
`stat path` : 只看属性, 不看数据
## 查看子节点
```ls [-s] path```
## 监听器
```
get -w
ls -w
stat -w
```
## Zookeeper权限控制
```
getAcl path
setAcl path acl
addauth scheme auth

```
### World 授权模式
```
e.g. 
setAcl /node1 world:anyone:rwa
crwda : create, read, write, delete, admin
```
### IP授权模式
```
setAcl /node2 ip:127.0.0.1:cdrwa
```
### auth授权模式
```
//添加认证用户后即可访问(digest:摘要)
addauth digest username:password
setAcl /node3 auth:username:cdrwa
```
### 加密授权
```
digest:zink:QzClKsadasDKasdASjf=:cdrwa
```
### 多种模式授权
```
addauth digest zink:123456
setAcl /node5 ip:127.0.0.1:cdra,auth:zink:cdrwa,digest:zink:QzClKsadasDKasdASjf=:cdrwa
```
## Acl超级管理员
zookeeper的权限管理模式有一种叫super, 该模式提供超管,可以管理任何权限的节点
首先生成密码的密文
```
echo -n super:admin|openssl dgst -binary -sha1|openssl base64
```
然后打开zkServer.sh,在启动zookeeper的命令中添加如下:
```
-Dzookeeper.DigestAuthenticationProvider.superDigest=super:xQJmxLMiHGwaqBvst5y6rkB6HQs=
```
最后添加如下命令添加权限, 即可获取所有权限
```
addauth digest super:admin
```
# zookeeper javaAPI
znode是zookeeper集合的核心组件.
## 连接到ZooKeeper
```
ZooKeeper(String connectionString, int sessionTimeout, Watcher watcher)

try{
	//计数器对象
	CountDownLatch cdl = new CountDownLatch(1);
	/**
	 * 1. 端口
	 * 2. 超时时间, 毫秒
	 * 3. 监视器对象
	 */
	ZooKeeper zooKeeper = new ZooKeeper("localhost:2181", 2000, new Watcher() {
		@Override
		public void process(WatchedEvent event) {
			if (event.getState() == Event.KeeperState.SyncConnected){
				System.out.println("连接创建成功!");
				cdl.countDown();
			}
		}
	});
	cdl.await();
	System.out.println(zooKeeper.getSessionId());
	zooKeeper.close();
	}catch(Exception e){
	e.printStackTrace();
}
```
## 配置中心案例

```java
public class MyConfigCenter implements Watcher {
     String url ;
     String user ;
     String pass ;
    CountDownLatch cdl = new CountDownLatch(1);
    static ZooKeeper zooKeeper;

    static String IP = "127.0.0.1:2181";

    public MyConfigCenter() {
        try {
            zooKeeper = new ZooKeeper(IP, 2000, this);
            cdl.await();
            init();
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void init() throws KeeperException, InterruptedException, IOException {
        this.url = new String(zooKeeper.getData("/config/url",true,null));
        this.user = new String(zooKeeper.getData("/config/user",true,null));
        this.pass = new String(zooKeeper.getData("/config/pass",true,null));
        System.out.println("config changed");
    }

    public static void main(String[] args) throws InterruptedException, KeeperException {
        MyConfigCenter m = new MyConfigCenter();
        for (int i = 0; i < 10; i++) {
            Thread.sleep(1000);
            System.out.println(m.url);
        }
    }

    @SneakyThrows
    @Override
    public void process(WatchedEvent event) {
        if (event.getType() == Event.EventType.None){
            if (event.getState() == Event.KeeperState.SyncConnected){
                System.out.println("Created");
                cdl.countDown();
            }
            if (event.getState() == Event.KeeperState.Disconnected){
                System.out.println("disconnected");
            }
            if (event.getState() == Event.KeeperState.Expired){
                System.out.println("timeout");
            }
            if (event.getState() == Event.KeeperState.AuthFailed){
                System.out.println("AuthFailed");
            }
        }else {
            if (event.getType() == Event.EventType.NodeDataChanged){
                init();
                System.out.println("changed");
            }
        }
    }
}
```
## 生成分布式唯一ID
1. 连接zookeeper服务器
2. 指定路径生成有序节点
3. 其序列号即可作为分布式唯一ID

```java
public String getUniqueId(){
        String path = "";
        try {
            path = zooKeeper.create(defaultPath,new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.EPHEMERAL_SEQUENTIAL);
        } catch (KeeperException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return path.substring(9);
    }
```
## 生成分布式锁
