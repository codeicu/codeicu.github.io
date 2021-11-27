---
layout: post
title: 2017.10.30 Spring,Struts学习
date: 2017-10-29 07:55:58
tags: Java Framework
---
使用AspectJ的XML配置声明式事务
```
1. 配置声明式事务管理器
创建DataSourceTranscationManager对象

<!--第一步,配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<!--第二步 配置事务的增强-->
<tx:advice id="txadvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--设置匹配规则-->
        <tx:method name="account*" propagation="REQUIRED"/>
        <tx:method name="insert*"/>
    </tx:attributes>
</tx:advice>
<!--第三步 配置切面-->
<aop:config>
    <aop:pointcut id="pointcut1" expression="execution(* com.Spring_tx.OrdersService.*(..))"></aop:pointcut>
    <aop:advisor advice-ref="txadvice" pointcut-ref="pointcut1"></aop:advisor>
</aop:config>
```
```
注解方式实现:
<!--第一步,配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"></property>
</bean>
<!--第二步-->
    <tx:annotation-driven transaction-manager="transactionManager"></tx:annotation-driven>
<!--第三步-->
找到@Transactional,里面所有方法加事务管理

```
Hibernate框架
```
ORM思想
实体类与数据库映射
Hibernate与Spring整合,两个核心配置文件名称和位置没有固定要求
hibernate.cfg.xml
```
Hibernate入门学习
```
1.让实体类和表一一对应 使用配置文件方式

2. 不需要操作表,只需要操作表对应的实体类对象就可以了

3. hibernate封装的对象session

4. 创建实体类对象
User user = new User();
user.setUsername("lucy");
session.save(user);
```
```
1.hibernate要求实体类中有一个属性唯一的值
2.使用hibernate时不需要手动创建表,会自动建表
3.配置实体类和数据库表一一对应关系(使用配置文件实现映射)
配置名称和位置没有固定要求
建议:实体类所在包里创建,名称:Hbm.xml
配置xml首先要引入约束,schema,dtd
引入约束
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
```
然后:hibernate的映射
```
<hibernate-mapping>
    <!--class
    name:全路径
    table:数据库表的名称-->
    <class name="cn.itcast.entity.User" table="t_user">
        <!--2.配置要求实体类有一个属性唯一值-->
        <!--id:实体类id属性名称,column:表字段名称-->
        <id name="uid" column="uid">
            <!--设置增长策略-->
            <!--native就是自动增长-->
            <generator class="native"></generator>
        </id>
        <!--配置其他属性和表字段对应-->
        <property name="username" column="username"></property>
        <property name="password" column="password"></property>
        <property name="address" column="address"></property>
    </class>
</hibernate-mapping>
```
Spring的约束是screma
4.创建hibernate的核心配置文件
```
核心配置文件格式xml
期名称和位置是固定的
1.位置:src下面
名称:hibernate.cfg.xml
2.引入dtd约束
<?xml version="1.0" encoding="utf-8" ?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
3.hibernate操作中只加载核心配置文件,其他配置文件不会加载

```
```
1.配置数据库信息
2.配置hibernate信息
3.把映射文件放到配置文件中
<hibernate-configuration>
    <session-factory>
        <!--第一部分,必须的-->
        <property name="hibernate.connection.driver_class" >com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/hibernate</property>
        <property name="connection.username">root</property>
        <property name="connection.password">admin</property>
        <!--第二部分,可选-->
        <!--底层输出sql-->
        <property name="hibernate.show_sql">true</property>
        <!--底层输出sql语句格式-->
        <property name="hibernate.format_sql">true</property>
        <!--hibernate帮忙创建表-->
        <!--update如果有表就更新，没表就创建-->
        <property name="hibernate.hbm2ddl.auto">update</property>
        <!--配置数据库的方言-->
        <!--让hibernate识别不同数据库的语句-->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQLDialect</property>
        <mapping resource="cn/itcast/entity/User.hbm.xml"></mapping>
    </session-factory>
</hibernate-configuration>

```
hibernate操作的步骤
```
1.加载核心配置文件
2.创建SessionFactory对象
3.使用SessionFactory创建Session对象(与HTTPSession没有任何关系)
4.开启事务
5.写我们具体的逻辑crud
6.提交事务
7.关闭资源/连接

        Configuration cfg = new Configuration();
        cfg.configure();
        //2. 创建SessionFactory
        //在这个过程中,会找到映射文件,把表也创建出来
        SessionFactory sessionFactory = cfg.buildSessionFactory();
        //3. 创建session对象
        //如何理解:类似于Connection conn = DriverManager.getConnection
        Session session = sessionFactory.openSession();
        //4. 开启事务
        Transaction transaction = session.beginTransaction();
        //5. 写具体逻辑
        User user = new User(x);
        user.setUid(123456);
        user.setUsername("小王");
        user.setPassword("123");
        user.setAddress("美国");
        session.save(user);
        //6. 提交事务
        transaction.commit();
        //7. 关闭事务
        session.close();
        sessionFactory.close();
```
hibernate配置文件详解
```
1. hibernate映射文件名称位置不固定
2.关于映射配置文件
内部中写的都是实体类相关的
- 1. class的name属性值是实体类的全路径
- 2. id标签和property的name都是实体类属性名称
3.id标签好property标签,column属性可以省略,表的字段自动与实体类属性名称一致

```
hibernate中核心api的使用
```
创建SessionFactory特别耗费资源
所以在hibernate操作中,建议一个项目创建一个sessionFactory
具体实现:
1.写工具类,写静态代码实现
public class HibernateUtils {
    static Configuration cfg = null;
    static SessionFactory sessionFactory = null;
    static {
        //加载核心配置文件
        cfg = new Configuration();
        cfg.configure();
        SessionFactory sessionFactory = cfg.buildSessionFactory();
    }
    public static SessionFactory getSessionFactory(){
        return sessionFactory;
    }
}

```
---
Session(重点)
1. 类似于jdbc中的connection
2. 调用session里面的不同方法实现crud操作
- 1.添加 save方法
- 2 修改 update方法
- 3. 删除方法
- 4. 根据id查询get方法
3. 一个项目一般只创建一个SessionFactory.session对象单线程对象,不能共用,只能自己用
---
Transaction
Transaction tx = session.beginTransaction();
tx.commit();
tx.rollback();
事务概念
- 1.原子性:一种操作一种失败都失败
- 2.一致性:操作之前之后要一致
- 3.隔离性:多个事务同时操作不影响
- 4.持久性:提交数据库后持久性
---
```
实体类的编写规则
实体类不建议使用基本数据类型,而是包装类
因为包装类Interger 可以为null,可以为0;
实体类也叫做持久化类

```
```
hibernate主键生成策略
increment:每次增量为1
identity:数据库支持自动自动增长的类型.适用于代理主键
sequence:条件是数据库支持序列,如oracle
native:根据底层数据库对自动生成标识符
uuid:32位16进制字符串 column="uuid"
```
---
```
对实体类crud的操作
1.添加
调用sesson save方法
sesson.save(user);
2.根据id查询
User user = session.get(User.class,1);
3.修改
先查询,再修改
User user = session.get(User.class,1);
user.setUserName("newName");
session.update(user);
4.删除
User user = session.get(User.class,1);
session.delete(user);
```
实体类对象的状态
```
瞬态值:
持久态:
托管态:
```
Hibernate的一级缓存
```
1. 把数据存在数据库里,数据库本身是文件系统,可以直接用流方式操作文件效率不高
2. 把数据放在内存中,提高效率

Hibernate的缓存特点:
1.Hibernate的一级缓存
- 1 默认打开
- 2 一级缓存有使用范围,从session创建到session关闭的范围.
- 3 一级缓存中存储数据必须是持久态数据
2. Hibernate的二级缓存已经没人使用,有一种替代技术,redis
- 1 默认不打开,需要配置
- 2 使用范围是整个web范围,SessionFactory范围.一个项目只有这一个对象
```
验证方式:
```
1.根据uid=1查询
2.再根据uid=1查询
验证第二次没有发送sql语句(第二次不查数据库而查一级缓存)
```
一级缓存的特性:
```
持久态会自动更新数据库
User user = session.get(User.class,7)
把user放在一级缓存快照区
user.setUsernamme("hanmeimei")
修改的是持久态对象的值
同时修改一级缓存的内容,但不一定修改一级缓存对应的快照区内容.
最后提交事务,会:
比较一级缓存和快照区内容是否相同,如果不同会把一级缓存区存放在数据库中
```
---
事务代码规范
```
1.不考虑隔离产生的问题
-1 脏读
-2 不可重复度
-3 虚读
解决方法:
设置事务隔离级别. 默认级别
-1 mysql默认隔离级别是repeatable read
<property name="hibernate.connection.isolation">4</property>
try{
    sessionFactory=HibernateUtils.getSessionFactory();
    session = sessionFactory.openSession();
    tx = session.beginTransaction();
    ...
    tx.commit();
}catch(){
    session.rollback();
    }
finaly{
    
}


```
hibernate绑定session
```
配置与本地线程绑定的session
获取本地线程绑定session,关闭session报错


```
struts2 重点:
```
1.Action操作
- 1.创建继承ActionSupport
- 2.配置action的访问路径,创建struts.xml的配置文件,其固定在src下面
- 3.配置多个action访问方法
- 4.在action获取表单提交数据
-- 1.获取request对象
模型驱动(重点)
--6.配置过滤器
2.值栈
-1. 向值栈中放置数据
- set方法,push方法,定义变量,生成get方法


3.拦截器
- 1. aop和责任链模式
- 2. 自定义拦截器
继承MethodFilterInterceptor
重写方法
配置拦截器和action的关联

```
