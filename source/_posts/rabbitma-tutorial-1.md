---
title: RabbitMQ教程翻译（一）
date: 2016-08-10 01:41:40
tags: rabbitMQ
---
#### 简介
RabbitMQ是一个消息代理，它最重要的作用也非常简单：就是收发消息。你可以把他当作一个邮局，当你把信件投递到邮箱中时你可以肯定的是邮递员会将你的邮件投递到收件人那。在这个隐喻中，RabbitMQ就是邮箱，邮局和邮递员的角色。
RabbitMQ和邮局最主要的区别就是它不处理信件，而是接受，存储和发送二进制的数据。
RabbitMQ和消息传递的过程，我们规定几个术语:
1. Producing代表发送的意思。A程序发送消息那么A程序就是生产者（producer）。我们将生产者简写为P，如下图：
![P](http://www.rabbitmq.com/img/tutorials/producer.png)

<!-- more -->
2. 一个消息队列就相当于一个邮箱，它属于RabbitMQ中。和消息流在RabbitMQ和你的程序中一样，消息流也是属于消息队列的。一个队列里没有消息数量的限制，可以存储你所需要的海量消息，也就是说消息队列是一个无限大小的缓存池。很多生产者可以向同一个消息队列里发送消息，统一很消费者也可以尝试在同一个队列里获取消息。一个消息队列(queue)可以以下图表示:
![queue](http://www.rabbitmq.com/img/tutorials/queue.png)

3. 消费就是接受消息的意思了，消费者(Consumer)也就是那些等待接受并处理消息的程序，以下图表示:
![Consumer](http://www.rabbitmq.com/img/tutorials/consumer.png)

说明:生成者，消费者和消息代理并不一定是要在同一个系统中，的确，在很多项目中，这三者并不是在同一个系统中。

#### 基于RabbitMQ “Hello World” 的Java实现
在本教程的这一节中，我们写两个java小程序，一个生产者发送单个消息，同时一个消费者接受消息并且打印出来。我们会对java api的一些细节做一些说明。让我们集中精神在这个简单的小程序上开始我们的学习吧。
在下面的示意图中，"P"是我们的生产者，"C"是我们的消费者，中间的红色部分是一个队列，即RabbitMQ为消费者保存消息的一个消息缓存。
![](http://www.rabbitmq.com/img/tutorials/python-one.png)

在写生产者和消费者代码开始之前，我们需要下载需要的Jar，并解压:
``` shell
$ unzip rabbitmq-java-client-bin-*.zip
$ cp rabbitmq-java-client-bin-*/*.jar ./
```

#### 发送消息
![](http://www.rabbitmq.com/img/tutorials/sending.png)

创建一个连接并连接到RabbitMQ服务器的操作如下:
``` java
ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();
```
如果你需要验证用户名和密码：
``` java
factory.setUsername("yourname");
factory.setPassword("yourpassword");
```
上面创建的这个连接是一个抽象的socket连接，我们只需要关心RabbitMQ的相关协议就好。这里我们连接到了本地的一个消息代理（RabbitMQ服务器），当然你可以切换ip地址到你的RabbitMQ服务器中。
发送消息的时候，我们必须先声明一个队列让我们去发送，然后再把消息发布到这个队列中去:
``` java
channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    String message = "Hello World!";
    channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
    System.out.println(" [x] Sent '" + message + "'");
```
声明一个队列就意味着如果改队列不存在的话就会创建一个新队列。同时消息都是字节数组，编码方式你可以任意修改。
最后，就是关闭连接了:
``` java
channel.close();
connection.close();
```

#### 生产者完整代码
``` java
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

public class Send {

  private final static String QUEUE_NAME = "hello";

  public static void main(String[] argv) throws Exception {
    ConnectionFactory factory = new ConnectionFactory();
    factory.setHost("localhost");
    Connection connection = factory.newConnection();
    Channel channel = connection.createChannel();

    channel.queueDeclare(QUEUE_NAME, false, false, false, null);
    String message = "Hello World!";
    channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
    System.out.println(" [x] Sent '" + message + "'");

    channel.close();
    connection.close();
  }
}
```

正在翻译中，未完待续...



