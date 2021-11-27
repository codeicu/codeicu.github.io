---
title: 为什么阿里巴巴禁止把SimpleDateFormat定义为static类型
date: 2018-12-05 11:10:52
tags: 技巧
---
# SimpleDateFormat线程安全性
JDK文档中明确表明:
- Date formats are not synchronized.It is recommended to create separate format instances for each thread.If multiple threads access a format concurrently, it must be synchronized externally.

# 原因:
￼SimpleDateFormat中的format方法在执行过程中，会使用一个成员变量calendar来保存时间。这其实就是问题的关键。
由于我们在声明SimpleDateFormat的时候，使用的是static定义的。那么这个SimpleDateFormat就是一个共享变量，随之，SimpleDateFormat中的calendar也就可以被多个线程访问到。
假设线程1刚刚执行完calendar.setTime把时间设置成2018-11-11，还没等执行完，线程2又执行了calendar.setTime把时间改成了2018-12-12。这时候线程1继续往下执行，拿到的calendar.getTime得到的时间就是线程2改过之后的。
# 解决办法
1. 使用局部变量
```
for (int i = 0; i < 100; i++) {
   //获取当前时间
   Calendar calendar = Calendar.getInstance();
   int finalI = i;
   pool.execute(() -> {
       // SimpleDateFormat声明成局部变量
   SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
       //时间增加
       calendar.add(Calendar.DATE, finalI);
       //通过simpleDateFormat把时间转换成字符串
       String dateString = simpleDateFormat.format(calendar.getTime());
       //把字符串放入Set中
       dates.add(dateString);
       //countDown
       countDownLatch.countDown();
   });
}
```
2. 加同步锁
```
for (int i = 0; i < 100; i++) {
   //获取当前时间
   Calendar calendar = Calendar.getInstance();
   int finalI = i;
   pool.execute(() -> {
       //加锁
       synchronized (simpleDateFormat) {
           //时间增加
           calendar.add(Calendar.DATE, finalI);
           //通过simpleDateFormat把时间转换成字符串
           String dateString = simpleDateFormat.format(calendar.getTime());
           //把字符串放入Set中
           dates.add(dateString);
           //countDown
           countDownLatch.countDown();
       }
   });
}
```

3. 使用ThreadLocal
```
private static ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
   @Override
   protected SimpleDateFormat initialValue() {
       return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
   }
};

```
4. 使用DateTimeFormatter

就像官方文档中说的，这个类 simple beautiful strong immutable thread-safe。
```
//解析日期
String dateStr= "2016年10月25日";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日");
LocalDate date= LocalDate.parse(dateStr, formatter);

//日期转换为字符串
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy年MM月dd日 hh:mm a");
String nowStr = now .format(format);
System.out.println(nowStr);
```
本文摘录学习自[Hollis](https://mp.weixin.qq.com/s/i2t0uYxbVeqRKGTc6qurag)

