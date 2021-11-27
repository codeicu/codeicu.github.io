---
layout: post
title:  properties的使用
date: 2017-10-20 07:55:58
tags: Java
---
```
InputStream in = JdbcPool.class.getClassLoader().getResourceAsStream("pic/test/db.properties");
Properties prop = new Properties();
try{
    prop.load(in);
    String driver = prop.getProperty("driver");
    String url = prop.getProperty("url");
    String username = prop.getProperty("username");
    String password = prop.getProperty("password");
    }catch(...){
        ...
    }
```
往properties里增加值
```
OutputStream fos = new FileOutputStream("C:\\Users\\Administrator\\Desktop\\db.properties");
properties.setProperty(String.valueOf(i++),m.group(0));
properties.store(fos,"Update '"+i + "' value");

```

只有符合切入点,才让通知和目标方法结合在一起
目标对象:
AOP代理:
切面:
织入:形成代理对象的方法的过程就是织入
通知:切面中的方法
切入点:
连接点:
