---
layout: post
title:  "Springboot常用接口"
date:   2022-06-18 16:39:38 +0800
categories: springboot
---
# 1. Springboot常用接口

## 1.1 Aware接口

Spring IOC容器中 Bean是感知不到容器的存在，Aware(意识到的)接口就是帮助Bean感知到IOC容器的存在，即获取当前Bean对应的Spring的一些组件，如当前Bean对应的ApplicationContext等。

### 1.1.1 ApplicationContextAware 

```java

@Component
public class SpringContextUtil implements ApplicationContextAware {
    private static ApplicationContext applicationContext;

    public SpringContextUtil() {
    }

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    public static <T> T getBean(String name) throws BeansException {
        return applicationContext.getBean(name);
    }

    public static <T> T getBean(Class<?> clz) throws BeansException {
        return applicationContext.getBean(clz);
    }
}
```

### 1.1.2 EnvironmentAware 环境变量读取和属性对象获取

关于Environment的作用

> Environment：环境，Profile 和 PropertyResolver 的组合。
Environment 对象的作用是确定哪些配置文件（如果有）当前处于活动状态，以及默认情况下哪些配置文件（如果有）应处于活动状态。
properties 在几乎所有应用程序中都发挥着重要作用，并且有多种来源：属性文件，JVM 系统属性，系统环境变量，JNDI，
servlet 上下文参数，ad-hoc 属性对象，映射等。同时它继承 PropertyResolver 接口，
所以与属性相关的 Environment 对象其主要是为用于配置属性源和从中属性源中解析属性。

```java
@Component
 public class MyEnvironment implements EnvironmentAware {
    private Environment environment;
    @Override
    public void setEnvironment(Environment environment) {
           this.environment=environment;
    }
 }
 ```

 ### 1.1.3 其他Aware接口


- ApplicationEventPublisherAware (org.springframework.context)：事件发布
用法可参考：ApplicationEventPublisherAware事件发布详解
    
- ServletContextAware (org.springframework.web.context): ServletContext
ServletConfigAware (org.springframework.web.context): ServletConfig
用法可参考：ServletConfig与ServletContext接口API详解和使用

- MessageSourceAware (org.springframework.context): 国际化
用法可参考：Spring的国际化资源messageSource
    
- ResourceLoaderAware (org.springframework.context)：访问底层资源的加载器
用法参考：Spring使用ResourceLoader接口获取资源
    
- NotificationPublisherAware (org.springframework.jmx.export.notification)：JMX通知
Spring JMX之一：使用JMX管理Spring Bean： https://www.cnblogs.com/duanxz/p/3968308.html
Spring JMX之三：通知的处理及监听： https://www.cnblogs.com/duanxz/p/4036619.html

- BeanFactoryAware (org.springframework.beans.factory): 申明BeanFactory
Spring BeanFactory体系: https://juejin.cn/post/6844903881890070541

- EmbeddedValueResolverAware (org.springframework.context): 手动读取配置
使用方式可以参考：Spring中抽象类中使用EmbeddedValueResolverAware和@PostConstruct获取配置文件中的参数值
ImportAware (org.springframework.context.annotation):处理自定义注解
关于ImportAware的作用可以看： Spring的@Import注解与ImportAware接口

- BootstrapContextAware (org.springframework.jca.context): 资源适配器BootstrapContext,如JAC，CCI
springcloud情操陶冶-bootstrapContext(二): https://blog.csdn.net/weixin_30725467/article/details/96008648
    
- LoadTimeWeaverAware (org.springframework.context.weaving)：加载Spring Bean时植入第三方模块，如AspectJ
说说在 Spring AOP 中如何实现类加载期织入（LTW）: https://juejin.cn/post/6844903664469950478
   
- BeanClassLoaderAware (org.springframework.beans.factory): 加载Spring Bean的类加载器
深入理解Java类加载器(ClassLoader): https://blog.csdn.net/javazejian/article/details/73413292

- BeanNameAware (org.springframework.beans.factory): 申明Spring Bean的名字

- ApplicationContextAware (org.springframework.context)：申明ApplicationContext


## 1.2 ApplicationEvent ApplicationListener 自定义事件，及监听

如果容器中存在ApplicationListener的Bean，当ApplicationContext调用publishEvent方法时，对应的Bean会被触发。

|内置事件|描述|
| - | - |
|ContextRefreshedEvent    |ApplicationContext 被初始化或刷新时，该事件被触发。这也可以在 ConfigurableApplicationContext接口中使用 refresh() 方法来发生。此处的初始化是指：所有的Bean被成功装载，后处理Bean被检测并激活，所有Singleton Bean 被预实例化，ApplicationContext容器已就绪可用|
|ContextStartedEvent        |当使用 ConfigurableApplicationContext （ApplicationContext子接口）接口中的 start() 方法启动 ApplicationContext 时，该事件被发布。你可以调查你的数据库，或者你可以在接受到这个事件后重启任何停止的应用程序。|
|ContextStoppedEvent |当使用 ConfigurableApplicationContext 接口中的 stop() 停止 ApplicationContext 时，发布这个事件。你可以在接受到这个事件后做必要的清理的工作。|
|ContextClosedEvent       |当使用 ConfigurableApplicationContext 接口中的 close() 方法关闭 ApplicationContext 时，该事件被发布。一个已关闭的上下文到达生命周期末端；它不能被刷新或重启。|
|RequestHandledEvent|这是一个 web-specific 事件，告诉所有 bean HTTP 请求已经被服务。只能应用于使用DispatcherServlet的Web应用。在使用Spring作为前端的MVC控制器时，当Spring处理用户请求结束后，系统会自动触发该事件。|

同样,事件可以自定义, 监听也可以自定义

```java
@Component
public class BrianListener implements ApplicationListener<ContextRefreshedEvent> {

    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println("容器 init Bean数量:" + event.getApplicationContext().getBeanDefinitionCount());
    }

}
```

## 1.3 InitializingBean接口 

InitializingBean 为bean提供了初始化方法的方式，它只包括afterPropertiesSet方法，凡是实现该接口的类，在初始化bean的时候会执行该方法。效果等同于bean的init-method属性的使用或者@PostContsuct注解的使用。

三种方式的执行顺序：@PostContsuct -> 执行InitializingBean接口中定义的方法 -> 执行init-method属性指定的方法。

```java
@Component
public class TestInitializingBean implements InitializingBean{
    @Override
    public void afterPropertiesSet() throws Exception {
       //bean 初始化后执行
    }
}
```

## 1.4 BeanPostProcessor接口

实现BeanPostProcessor接口的类即为Bean后置处理器，Spring加载机制会在所有Bean初始化的时候遍历调用每个Bean后置处理器。BeanPostProcessor 为每个bean实例化时提供个性化的修改，做些包装等。

> 其顺序为：Bean实例化 -> 依赖注入 -> Bean后置处理器 (postProcessBeforeInitialization)  -> @PostConstruct  ->  InitializingBean实现类 -> 执行init-method属性指定的方法 -> Bean后置处理器 (postProcessAfterInitialization) 


```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    /**
     * 在bean实例化之前回调
     */
    public Object postProcessBeforeInitialization(Object bean, String beanName)throws BeansException {
        System.out.println("postProcessBeforeInitialization: " + beanName);
        return bean;
    }

    /**
     * 在bean实例化之后回调
     */
    public Object postProcessAfterInitialization(Object bean, String beanName)throws BeansException {
        System.out.println("postProcessAfterInitialization: " + beanName);
        return bean;
    }
}
```

## 1.5 BeanFactoryPostProcessor接口

bean工厂的bean属性处理容器，说通俗一些就是可以管理我们的bean工厂内所有的beandefinition（未实例化）数据，可以随心所欲的修改属性

```java
@Component
@Slf4j
public class BrianBeanFactoryPostProcessor implements BeanFactoryPostProcessor { 

　　public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException { 
　　　　BeanDefinition bd = beanFactory.getBeanDefinition("brianBean"); 
　　　　MutablePropertyValues pv = bd.getPropertyValues(); 

　　　　if (pv.contains("kawa")) { 
　　　　　　pv.addPropertyValue("kawa", "修改属性kawa的值"); 
　　　　} 
　　}
}
```

## 1.6 CommandLineRunner 和 ApplicationRunner接口

> CommandLineRunner 及 ApplicationRunner 都可实现在项目启动后执行操作别ApplicationRunner会封装命令行参数，可以很方便地获取到命令行参数和参数值

```java
@Component
@Slf4j
@Order(value = 2)
public class InitServiceConfiguration implements CommandLineRunner {
    @Autowired
    private ApplicationContext applicationContext;

    @Override
    public void run(String... args) throws Exception {
        Map<String, AbstractInitService> serviceMap = applicationContext.getBeansOfType(AbstractInitService.class);
        if (serviceMap == null) {
            return;
        }
        ExecutorService fixedThreadPool = Executors.newFixedThreadPool(3);
        for (Map.Entry<String, AbstractInitService> entry : serviceMap.entrySet()) {
            log.info("初始化:{}", entry.getKey());
            fixedThreadPool.execute(entry.getValue());
        }
        fixedThreadPool.shutdown();
    }
}
```

# 2. SpringBoot常用工具类

 - 字符串工具类 org.springframework.util.StringUtils
 - 对象序列化工具类 org.springframework.util.SerializationUtils
- 数字处理工具类 org.springframework.util.NumberUtils
- org.springframework.util.FileCopyUtils
- MD5加密 工具类org.springframework.util.DigestUtils
- 用于处理表达资源字符串前缀描述资源的工具 ResourceUtils
- org.springframework.core.io.support.PropertiesLoaderUtils 加载Properties资源工具类,和Resource结合
- org.springframework.core.NestedExceptionUtils
-  xml工具类 org.springframework.util.xml.*
```
org.springframework.util.xml.AbstractStaxContentHandler

org.springframework.util.xml.AbstractStaxXMLReader

org.springframework.util.xml.AbstractXMLReader

org.springframework.util.xml.AbstractXMLStreamReader

org.springframework.util.xml.DomUtils

org.springframework.util.xml.StaxUtils

org.springframework.util.xml.TransformerUtils
```
-  web工具类
```
org.springframework.web.util.CookieGenerator

org.springframework.web.util.HtmlCharacterEntityDecoder

org.springframework.web.util.HtmlCharacterEntityReferences

org.springframework.web.util.HtmlUtils
org.springframework.web.util.JavaScriptUtils

org.springframework.web.util.HttpUrlTemplate  这个类用于用字符串模板构建url, 它会自动处理url里的汉字及其它相关的编码. 在读取别人提供的url资源时, 应该经常用

org.springframework.web.util.Log4jConfigListener  用listener的方式来配制log4j在web环境下的初始化

org.springframework.web.util.UriTemplate

org.springframework.web.util.UriUtils  处理uri里特殊字符的编码

org.springframework.web.util.WebUtils
org.springframework.web.bind.ServletRequestUtils

介绍Spring WebUtils 和 ServletRequestUtils:  https://blog.csdn.net/neweastsun/article/details/80873994
```

- 其它工具集

```
org.springframework.util.xml.AntPathMatcherant   风格的处理

org.springframework.util.xml.AntPathStringMatcher

org.springframework.util.xml.Assert    断言,在我们的参数判断时应该经常用

org.springframework.util.xml.CachingMapDecorator

org.springframework.util.xml.ClassUtils   用于Class的处理

org.springframework.util.xml.CollectionUtils   用于处理集合的工具

org.springframework.util.xml.CommonsLogWriter

org.springframework.util.xml.CompositeIterator

org.springframework.util.xml.ConcurrencyThrottleSupport

org.springframework.util.xml.CustomizableThreadCreator

org.springframework.util.xml.DefaultPropertiesPersister

org.springframework.util.xml.LinkedCaseInsensitiveMap key值不区分大小写的LinkedMap

org.springframework.util.xml.LinkedMultiValueMap   一个key可以存放多个值的LinkedMap

org.springframework.util.xml.Log4jConfigurer   一个log4j的启动加载指定配制文件的工具类

org.springframework.util.xml.ObjectUtils   有很多处理null object的方法. 如nullSafeHashCode, nullSafeEquals, isArray, containsElement, addObjectToArray, 等有用的方法

org.springframework.util.xml.PatternMatchUtils  spring里用于处理简单的匹配. 如 Spring's typical "xxx", "xxx" and "xxx" pattern styles

org.springframework.util.xml.PropertyPlaceholderHelper  用于处理占位符的替换

org.springframework.util.xml.ReflectionUtils   反映常用工具方法. 有 findField, setField, getField, findMethod, invokeMethod等有用的方法

org.springframework.util.xml.StopWatch  一个很好的用于记录执行时间的工具类, 且可以用于任务分阶段的测试时间. 最后支持一个很好看的打印格式. 这个类应该经常用

org.springframework.util.xml.SystemPropertyUtils

org.springframework.util.xml.TypeUtils   用于类型相容的判断. isAssignable

org.springframework.util.xml.WeakReferenceMonitor  弱引用的监控
```

# 3. springBoot 常用注解

```

@Component可配合CommandLineRunner使用，在程序启动后执行一些基础任务。

@JsonBackReference解决嵌套外链问题。
解决Json 无线递归问题注解@JsonBackReference该注解 加在GET SET方法头上： https://blog.csdn.net/weixin_45029961/article/details/109026387

@RepositoryRestResourcepublic配合spring-boot-starter-data-rest使用。

@EnableAutoConfiguration：Spring Boot自动配置（auto-configuration）：尝试根据你添加的jar依赖自动配置你的Spring应用。
例如，如果你的classpath下存在HSQLDB，并且你没有手动配置任何数据库连接beans，那么我们将自动配置一个内存型（in-memory）数据库，
你可以将@EnableAutoConfiguration或者@SpringBootApplication注解添加到一个@Configuration类上来选择自动配置。
如果发现应用了你不想要的特定自动配置类，你可以使用@EnableAutoConfiguration注解的排除属性来禁用它们。

@ComponentScan：表示将该类自动发现扫描组件。个人理解相当于，如果扫描到有@Component、@Controller、@Service等这些注解的类，
并注册为Bean，可以自动收集所有的Spring组件，包括@Configuration类。我们经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入。
可以自动收集所有的Spring组件，包括@Configuration类。我们经常使用@ComponentScan注解搜索beans，并结合@Autowired注解导入。
如果没有配置的话，Spring Boot会扫描启动类所在包下以及子包下的使用了@Service,@Repository等注解的类。

@Import：用来导入其他配置类。

@ImportResource：用来加载xml配置文件。

@Inject：等价于默认的@Autowired，只是没有required属性；

@Qualifier：当有多个同一类型的Bean时，可以用@Qualifier(“name”)来指定。与@Autowired配合使用。@Qualifier限定描述符除了能根据名字进行注入，但能进行更细粒度的控制如何选择候选者，具体使用方式如下：

@Autowired 
@Qualifier(value = “demoInfoService”) 
private DemoInfoService demoInfoService;
@Resource(name=”name”,type=”type”)：没有括号内内容的话，默认byName。与@Autowired干类似的事。

```

## 3.2 JPA注解

```
@Entity：@Table(name=”“)：表明这是一个实体类。一般用于jpa这两个注解一般一块使用，但是如果表名和实体类名相同的话，@Table可以省略

@MappedSuperClass:用在确定是父类的entity上。父类的属性子类可以继承。

@NoRepositoryBean:一般用作父类的repository，有这个注解，spring不会去实例化该repository。

@Column：如果字段名与列名相同，则可以省略。

@Id：表示该属性为主键。

@GeneratedValue(strategy = GenerationType.SEQUENCE,generator = “repair_seq”)：
表示主键生成策略是sequence（可以为Auto、IDENTITY、native等，Auto表示可在多个数据库间切换），指定sequence的名字是repair_seq。

@SequenceGeneretor(name = “repair_seq”, sequenceName = “seq_repair”, allocationSize = 1)：
name为sequence的名称，以便使用，sequenceName为数据库的sequence名称，两个名称可以一致。

@Transient：表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性。如果一个属性并非数据库表的字段映射,
就务必将其标示为@Transient,否则,ORM框架默认其注解为@Basic。@Basic(fetch=FetchType.LAZY)：标记可以指定实体属性的加载方式

@JsonIgnore：作用是json序列化时将Java bean中的一些属性忽略掉,序列化和反序列化都受影响。

@JoinColumn（name=”loginId”）:一对一：本表中指向另一个表的外键。一对多：另一个表指向本表的外键。

@OneToOne、@OneToMany、@ManyToOne：对应hibernate配置文件中的一对一，一对多，多对一。
```
##  3.3 springMVC相关注解
```
@RequestMapping：@RequestMapping(“/path”)表示该控制器处理所有“/path”的UR L请求。RequestMapping是一个用来处理请求地址映射的注解，可用于类或方法上。
用于类上，表示类中的所有响应请求的方法都是以该地址作为父路径。

该注解有六个属性：
params:指定request中必须包含某些参数值是，才让该方法处理。
headers:指定request中必须包含某些指定的header值，才能让该方法处理请求。
value:指定请求的实际地址，指定的地址可以是URI Template 模式
method:指定请求的method类型， GET、POST、PUT、DELETE等
consumes:指定处理请求的提交内容类型（Content-Type），如application/json,text/html;
produces:指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回
```

## 3.4 异常处理注解

```
@ControllerAdvice：包含@Component。可以被扫描到。统一处理异常。
@ExceptionHandler（Exception.class）：用在方法上面表示遇到这个异常就执行以下方法。
用法可参考： https://blog.csdn.net/kinginblue/article/details/70186586
```