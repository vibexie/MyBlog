---
title: springMVC整合RabbitMQ
date: 2016-08-19 22:54:36
tags: springMVC
---
### 配置Maven
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit</artifactId>
        <version>1.6.1.RELEASE</version>
    </dependency>
</dependencies>
```
官方地址: [https://projects.spring.io/spring-amqp/](https://projects.spring.io/spring-amqp/)

### 谁来发送消息？
我们需要一个Service通过RabbitTemplate来发送消息。
``` java
package com.vibexie.server.service;

import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class RabbitService {
	@Autowired RabbitTemplate rabbitTemplate;
	
	/**
	 * 发送消息
	 * @param queue
	 * @param msg
	 */
	public void sendMessage(String queue, String msg) {
		rabbitTemplate.convertAndSend(queue, msg);
	}
}
```
<!-- more -->
在你需要发送消息的类中：
``` java
@Autowired RabbitService rabbitService;
rabbitService.sendMessage("oneQueue", "来自RabbitService的消息");
```
即可发出消息了。


### 谁来接收消息？
我们需要为每个队列写一个监听器，通过监听的方式接收消息，下面以两个队列为例，分别为queueOne和queueTwo。
* QueueOneListener.java
``` java
package com.vibexie.server.rabbit;
import org.springframework.stereotype.Service;

public class QueueOneListener {
	public void onReceive(byte[] msg) throws Exception {
		// TODO Auto-generated method stub
		System.out.println("-----QueueOne收到消息:" + new String(msg).toString());
	}
}
```
* QueueTwoListener.java
``` java
package com.vibexie.server.rabbit;
import org.springframework.stereotype.Service;

public class QueueOneListener {
	public void onReceive(byte[] msg) throws Exception {
		// TODO Auto-generated method stub
		System.out.println("-----QueueTwo收到消息:" + new String(msg).toString());
	}
}
```

### Spring整合
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" 
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:rabbit="http://www.springframework.org/schema/rabbit"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
	                    http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
	                    http://www.springframework.org/schema/rabbit http://www.springframework.org/schema/rabbit/spring-rabbit-1.5.xsd" >
	
	<!-- 连接工厂 -->
	<rabbit:connection-factory id="connectionFactory" host="" username="" password="" cache-mode="CONNECTION" connection-cache-size="25" />
	
	<!-- Rabbit模板 -->
    <rabbit:template id="rabbitTemplate" connection-factory="connectionFactory"/>
	   
	<!-- 注册QueueOne队列 -->
	<rabbit:queue name="queueOne" durable="true" />
	<!-- 注册QueueOne队列监听处理器 -->
	<bean id="queueOneListener" class="com.vibexie.server.rabbit.QueueOneListener" />
	
	<!-- 注册QueueTwo队列 -->
	<rabbit:queue name="queueTwo" durable="true" />
	<!-- 注册QueueTwo队列监听处理器 -->
	<bean id="queueTwoListener" class="com.vibexie.server.rabbit.QueueTwoListener" />
	
	<!-- 设置监听器 -->
	<rabbit:listener-container connection-factory="connectionFactory"
	    acknowledge="auto">  <!-- acknowledge可为auto,none,manual三种模式-->
	
        <!-- queues是队列名称，可填多个，用逗号隔开， method是ref指定的Bean调用Invoke方法执行的方法名称 -->
        <rabbit:listener queues="queueOne" method="onReceive" ref="queueOneListener" />
        <rabbit:listener queues="queueTwo" method="onReceive" ref="queueTwoListener" />
	</rabbit:listener-container>
</beans>
```
### 特别说明
对于Rabbit的acknowledge mode，有三种，即auto，none，manual，也就是自动确认，不确认，手动确认三种模式。如果你的业务需要保证消息接收过来都保证需要处理，那就不推荐none模式了，同时，在spring整合下，也没法使用manual的方式手动去确认消息，所以，此时最好的方式就是auto的方式了，只要在QueueOneListener的onReceive方法中没有抛异常，就自动确认消费成功，否则确认消费失败。
概况：
1. none：接收消息的速度最快，但是没有ack机制
2. manual：抱歉，spring整合下没法手动确认
3. auto：推荐模式，但是相对于none，接收消息的速度满了很多。
