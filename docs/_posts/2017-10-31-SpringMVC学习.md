---
layout: post
title: 2017.10.31 SpringMVC学习
date: 2017-10-31 07:55:58
tags: Java Framework
---
常见的处理器映射
```
BeanNameUrlHandlerMapping
SimpleUrlHandlerMapping
ControllerClassNameHandlerMapping
SpringMVC加载是什么时候?
第一次调用时创建
默认哪个handlerMapping起作用?
按照xml配置顺序来处理,将所有controller链接,在Springmvc容器加载时进行初始化.调用时,在这个map找到第一个匹配
```
终于配好了controller命令控制器(简单的封装)!!!见下
```
MyCommandController类:
public class MyCommandController extends
AbstractCommandController {
    //初始化构造器,将command类型设置为person;类
    public MyCommandController(){
        this.setCommandClass(Person.class);
    }
    @Override
    protected ModelAndView handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, BindException e) throws Exception {
        Person p = (Person) o;
        System.out.println(p);
        return null;
    }
}
springmvc-servlet.xml中
    <bean class="controller.MyCommandController"></bean>
    <bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"></bean>

web.xml中
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:springmvc-servlet.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.action</url-pattern>
    </servlet-mapping>
```
SimpleFormController
有些细节知道之后,配置也就不难了

MyFormController类中:
```
public class MyFormController extends SimpleFormController{
    public MyFormController(){
        this.setCommandClass(Person.class);
    }
    //form表用post方式提交会默认调用doSubmitAction
    protected void doSubmitAction(Object command) throws Exception{
        Person p = (Person)command;
        super.doSubmitAction(command);
    }
}

```
在springmvc-servlet.xml中
```
    //需要配置两个属性,表单页面和成功后的跳转页面
    <bean class="controller.MyFormController">
        <property name="formView" value="/person/jPersonCreate"></property>
        <property name="successView" value="/index"></property>
    </bean>
    <bean class="org.springframework.web.servlet.mvc.support.ControllerClassNameHandlerMapping"></bean>

//内部资源解析器
    <bean id="jspViewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/pages"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
```
Wizard向导表单控制器
填简历时,要填各种数据,一个表单来填不好填写,分开填写

MyWizardController
```
public class MyWizardController extends AbstractWizardFormController {

    public MyWizardController(){
        this.setCommandClass(Person.class);
        this.setCommandName("person");//配置它可以获得person数据
    }
    @Override
    protected ModelAndView processCancel(HttpServletRequest request, HttpServletResponse response, Object command, BindException errors) throws Exception {
        return new ModelAndView("/wizard/jPersonInfo");
    }

    @Override
    protected ModelAndView processFinish(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, BindException e) throws Exception {
        Person p = (Person) o;
        System.out.println(p);
        return null;
    }
}

```
springmvc-servlet.xml
```
    <bean class="controller.MyWizardController">
        <property name="pages">
            <list>
                <value>/wizard/jPersonEdu</value>
                <value>/wizard/jPersonInfo</value>
                <value>/wizard/jPersonJob</value>
            </list>
        </property>
    </bean>
```
jPersonEdu
```
<tr>
    <td>name</td>
    <td><input type="text" name="name" value="${person.name}"/></td>
</tr>
<tr>
    <td>age</td>
    <td><input type="text" name="age"/></td>
</tr>
<tr>
    <td><input type="submit" name="_target1" value="下一页"></td>
    <td><input type="submit" name="_cancel" value="取消"></td>
    <td></td>
</tr>
```
基于注解方式实现springmvc

JDK1.8和Tomcat以及SpringMVC3.2版本不兼容,要用4.0的jar包

MyHomeController
```
@Controller
public class MyHomeController {
    @RequestMapping("/myhome.action")
    public String goHome(HttpServletRequest request){
        System.out.println(request.getRequestURI());
        return "pages/index.jsp";
    }
}

```
springmvc-servlet.xml
```
    <context:component-scan base-package="controller"></context:component-scan>
```
web.xml没有变化
