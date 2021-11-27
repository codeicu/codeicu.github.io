---
layout: post
title: "SpringBoot日志管理"
date: 2020-06-10T21:58:32+08:00
draft: false
tags: Spring
---
# 历史

1. 日志实现框架: JUL Logback Log4j Log4j2
2. 日志门面框架(抽象层): JCL slf4j, 避免代码改动影响用户使用

- JDK1.3前, 日志靠System.out/err.print, 存在巨大缺陷, 粒度不够细
- log4j出现, 于15年8月停止更新
- JDK1.4引入java.util.logging, JUL
- commons-logging出现, 解决日志实现框架的兼容问题
- log4j作者推出门面框架slf4j
- log4j作者推出logback
- log4j2对log4j重大升级, 修复bug, 极大提升性能

- 最佳组合: slf4j+logback, slf4j+log4j2

# 日志门面如何定位到日志实现

1. LoggerFactory.getLogger 
2. findPossibleStaticLoggerBinderPathSet
3. 获取StaticLoggerBinder所在Jar路径
4. 如果存在多个日志实现就打印提示
5. 使用StaticLoggerBinder获取工程再实现
```java
public static Logger getLogger(String name) {
        ILoggerFactory iLoggerFactory = getILoggerFactory();
        return iLoggerFactory.getLogger(name);
    }
	
public static ILoggerFactory getILoggerFactory() {
		//未初始化
        if (INITIALIZATION_STATE == UNINITIALIZED) {
            synchronized (LoggerFactory.class) {
                if (INITIALIZATION_STATE == UNINITIALIZED) {
				//on初始化
                    INITIALIZATION_STATE = ONGOING_INITIALIZATION;
					//实现寻址
                    performInitialization();
                }
            }
        }
        switch (INITIALIZATION_STATE) {
        case SUCCESSFUL_INITIALIZATION:
            return StaticLoggerBinder.getSingleton().getLoggerFactory();
        case NOP_FALLBACK_INITIALIZATION:
            return NOP_FALLBACK_FACTORY;
        case FAILED_INITIALIZATION:
            throw new IllegalStateException(UNSUCCESSFUL_INIT_MSG);
        case ONGOING_INITIALIZATION:
            // support re-entrant behavior.
            // See also http://jira.qos.ch/browse/SLF4J-97
            return SUBST_FACTORY;
        }
        throw new IllegalStateException("Unreachable code");
    }
	
	private final static void performInitialization() {
        bind();
        if (INITIALIZATION_STATE == SUCCESSFUL_INITIALIZATION) {
            versionSanityCheck();
        }
    }
	
	private final static void bind() {
        try {
            Set<URL> staticLoggerBinderPathSet = null;
            // skip check under android, see also
            // http://jira.qos.ch/browse/SLF4J-328
            if (!isAndroid()) {
				//找到可能的LoggerBinder
                staticLoggerBinderPathSet = findPossibleStaticLoggerBinderPathSet();
				//判断当前集合Size, 如果大于1就打印输出提示
                reportMultipleBindingAmbiguity(staticLoggerBinderPathSet);
            }
            // the next line does the binding
			//获得Factory并获取对象
            StaticLoggerBinder.getSingleton();
			//启动状态未成功
            INITIALIZATION_STATE = SUCCESSFUL_INITIALIZATION;
			//打印出最终使用的日志框架
            reportActualBinding(staticLoggerBinderPathSet);
            fixSubstituteLoggers();
            replayEvents();
            // release all resources in SUBST_FACTORY
            SUBST_FACTORY.clear();
        } catch (NoClassDefFoundError ncde) {
            String msg = ncde.getMessage();
            if (messageContainsOrgSlf4jImplStaticLoggerBinder(msg)) {
                INITIALIZATION_STATE = NOP_FALLBACK_INITIALIZATION;
                Util.report("Failed to load class \"org.slf4j.impl.StaticLoggerBinder\".");
                Util.report("Defaulting to no-operation (NOP) logger implementation");
                Util.report("See " + NO_STATICLOGGERBINDER_URL + " for further details.");
            } else {
                failedBinding(ncde);
                throw ncde;
            }
        } catch (java.lang.NoSuchMethodError nsme) {
            String msg = nsme.getMessage();
            if (msg != null && msg.contains("org.slf4j.impl.StaticLoggerBinder.getSingleton()")) {
                INITIALIZATION_STATE = FAILED_INITIALIZATION;
                Util.report("slf4j-api 1.6.x (or later) is incompatible with this binding.");
                Util.report("Your binding is version 1.5.5 or earlier.");
                Util.report("Upgrade your binding to version 1.6.x.");
            }
            throw nsme;
        } catch (Exception e) {
            failedBinding(e);
            throw new IllegalStateException("Unexpected initialization failure", e);
        }
    }
	
	
	static Set<URL> findPossibleStaticLoggerBinderPathSet() {
        // use Set instead of list in order to deal with bug #138
        // LinkedHashSet appropriate here because it preserves insertion order
        // during iteration
        Set<URL> staticLoggerBinderPathSet = new LinkedHashSet<URL>();
        try {
            ClassLoader loggerFactoryClassLoader = LoggerFactory.class.getClassLoader();
            Enumeration<URL> paths;
            if (loggerFactoryClassLoader == null) {
                paths = ClassLoader.getSystemResources(STATIC_LOGGER_BINDER_PATH);
            } else {
                paths = loggerFactoryClassLoader.getResources(STATIC_LOGGER_BINDER_PATH);
            }
			//多路径的话就添加多次
            while (paths.hasMoreElements()) {
                URL path = paths.nextElement();
                staticLoggerBinderPathSet.add(path);
            }
        } catch (IOException ioe) {
            Util.report("Error getting resources from path", ioe);
        }
        return staticLoggerBinderPathSet;
    }
```

# 日志使用

logger.debug("xyz {} is {}", i,j)
其他用法可以参考网上的配置.
