---
layout: post
title: 1.SpringBoot框架整体流程
date: 2020-05-27 11:00:59
tags: SpringBoot源码
categories: SpringBoot源码
---

SpringBoot的重要性无需多言, 底层代码也是老生常谈. 为了深入理解其中的逻辑, 最近一段时间跟着"跳跳虎"来学习, 并做一些总结.

1. Spring初始化分为两步: 框架初始化, 框架启动(包括自动化装配).
`new SpringApplication(primarySources).run(args);`

- 框架初始化包括:

```java
class SpringApplication{
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
//      配置资源加载器
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
//      配置主类,启动类
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
//      应用环境检测
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
//      系统初始化器
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
//      监听器
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
//      配置main所在类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
}
```

- 框架启动包括:

```java
class SpringApplication{
public ConfigurableApplicationContext run(String... args) {
//启动计时器
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
//无显示器环境配置
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
//发送start事件
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
//配置环境
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
//printBanner
			Banner printedBanner = printBanner(environment);
//创建应用上下文对象
			context = createApplicationContext();
//初始化失败分析器
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
//关联组件与上下文对象
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
//刷新上下文
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
//发送start事件
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
}
```

- 框架自动化装配包括

1. 收集配置文件中的配置工厂类
2. 加载组件工厂
3. 注册组件内定义bean
