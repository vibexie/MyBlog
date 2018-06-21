---
title: SpringMVC整合Quartz
date: 2016-08-16 20:41:02
tags: springMVC
---
#### 对于在JavaWeb项目中使用Timer的看法
Timer这个类用来做定时任务看起来无可厚非，具体的用法也是非常简单。但是在JavaWeb项目中也用Timer来做定时任务是否可行呢？我们先来看看使用这种方式的方案(监听Application的启动):
``` java
@Service
public class StartupListener implements ApplicationListener<ContextRefreshedEvent> {
	
	
	@Override
	public void onApplicationEvent(ContextRefreshedEvent arg0) {
		// TODO Auto-generated method stub
		if (arg0.getApplicationContext().getParent() == null) {
			//启动
			Logger.getLogger(StartupListener.class).info("服务器启动了......");
			init();
		}
	}
	
	/**
	 * 服务器相关的初始化操作
	 */
	private void init() {
	}
}
```
<!-- more -->
我们可以在上面的init方法中启动定时器Timer;
但...
在生产环境可别这么做哦，Tomcat具备热启动功能，当tomcat重新启动的时候，init会再次调用，所以，timer会增多。简单的说就是tomcat热启动一次，timer会增加一次，这样前前后后的timer都在跑，想必这偏离了你的需求吧。
所以，不要在JavaWeb项目中使用Timer。

#### 最优方案，集成Quartz
* 下载Quartz
对于Maven项目:
``` xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.2.1</version>
  </dependency>
  <dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.2.1</version>
  </dependency>
```
或者直接下载jar，将所有jar导入你的项目。 [下载链接点这里](http://www.quartz-scheduler.org/downloads/)

* 新建一个类来跑定时任务
MyQuartz.java
``` java
/**
 * 定时器
 * @author vibexie
 * 
 */
public class MyQuartz {
	@Autowired StatisticsService statisticsService;
	
	private Logger logger = Logger.getLogger(MyQuartz.class);
	
	//每秒执行的任务
	public void doPer1Second() {
		logger.info("执行1秒定时任务......");
		//注意操作需要加try catch
		try {
			statisticsService.resetEventsCount();
		} catch (Exception e) {
		}
	}
	
	//每10秒执行的任务
	public void doPer10Seconds() {
		logger.info("执行10秒定时任务......");
		//注意操作需要加try catch
		try {
			statisticsService.save2DB();
		} catch (Exception e) {
		}
	}
	
	//每天早上8点执行的任务
	public void do8Clock() {
		logger.info("上午8点定时任务......");
		statisticsService.sendEmail();
	}
}
```
* 添加quartz配置文件，放在项目src目录下
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
  
  <!-- 添加调度的任务bean 配置对应的class-->
  <bean id="myQuartz" class="com.vibexie.server.quartz.MyQuartz" />
  
  <!--配置调度具体执行的方法-->
  <bean id="doPer1Second"
    class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <property name="targetObject" ref="myQuartz" />
    <property name="targetMethod" value="doPer1Second" />
    <property name="concurrent" value="false" />
  </bean>
  <!--配置调度执行的触发的时间-->
  <bean id="doPer1SecondTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="doPer1Second" />
    <property name="cronExpression">
      <!-- 每秒执行 -->
      <value>* * * * * ?</value>
    </property>
  </bean>
  
  <bean id="doPer10Seconds"
    class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <property name="targetObject" ref="myQuartz" />
    <property name="targetMethod" value="doPer10Seconds" />
    <property name="concurrent" value="false" />
  </bean>
  <bean id="doPer10SecondsTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="doPer10Seconds" />
    <property name="cronExpression">
      <!-- 每10秒执行 -->
      <value>0/10 * * * * ?</value>
    </property>
  </bean>
  
  <bean id="do8Clock"
    class="org.springframework.scheduling.quartz.MethodInvokingJobDetailFactoryBean">
    <property name="targetObject" ref="hybQuartz" />
    <property name="targetMethod" value="do8Clock" />
    <property name="concurrent" value="false" />
  </bean>
  <bean id="do8ClockTrigger" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
    <property name="jobDetail" ref="do8Clock" />
    <property name="cronExpression">
      <!-- 每天8点执行 -->
      <value>0 0 8 * * ?"</value>
    </property>
  </bean>
  
  <!-- quartz的调度工厂 调度工厂只能有一个，多个调度任务在list中添加 -->
  <bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="triggers">
      <list>
        <!-- 所有的调度列表-->
        <ref local="doPer1SecondTrigger" />
        <ref local="doPer10SecondsTrigger" />
        <ref local="do8ClockTrigger" />
      </list>
    </property>
  </bean>
</beans>
```

* 在web.xml中载入quartz.xml
``` xml
<servlet>
    <servlet-name>springDispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:ssm.xml, classpath:quartz.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```
#### 遇到的问题
如果你使用的Spring版本为Spring 3.1+；启动web项目时有以下报错:
org.springframework.scheduling.quartz.CronTriggerBean

解决方案如下:
将类名替换下就好
JobDetailBean => JobDetailFactoryBean
CronTriggerBean => CronTriggerFactoryBean

