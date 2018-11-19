---
title: MySQL使用Federated引擎实现跨数据库同步
date: 2016-09-14 00:59:26
tags: mysql
---
近期为了实现2个系统间用户数据的同步，走了很多弯路。这2个系统一个是java + mysql, 另外一个是php + mysql。当一个系统用户数据的增删改查时，另外一个系统需要立即同步。

1. 刚刚开始，觉得最简单的方式就是两个异构系统互相写接口，通过接口的方式去同步对方的数据。但是问题出现了，多一个接口请求就对服务器多一份压力，大大降低了系统的性能，同时无法实现系统间事务操作，很容易造成一个系统插入了一条数据另外一个系统没有插入数据的情况。
2. 后来觉得消息队列貌似挺靠谱的，应该非常适合这样的情景，于是研究了一番RabbitMQ。同样，问题依然有，在使用了消息确认机制的前提下，系统对消息的处理速度太慢了，实测每秒同步20条消息左右，同时系统1也无法知道系统2顺利完成收到的消息。所以，消息队列实现同步的方案也pass。

<!-- more -->
偶然间，发了新大陆了，认识了从来没有使用过的Mysql引擎Federated，它可以实现操作数据库1的一张表，数据库2的一张表也同步。修改远程的数据库表如同修改本地数据库一样方便。

先看看你的mysql支持不支持federated引擎。使用show engines命令

* 如果FEDERATED 的Support 为YES，那就可以直接使用了。
* 如果为NO, 那你需要在my.conf文件中最后加入一行federated即可，然后重启就能看看的Support变成了YES。
* 再若根本就没有FEDERATED，那你的mysql就可能因为版本太低不支持了。

接下来，准备两个数据库，本地和远程。
先在远程数据库建表：
``` mysql
CREATE TABLE test_table (
    id     int(20) NOT NULL auto_increment,
    name   varchar(32) NOT NULL default '',
    PRIMARY KEY  (id)
);
```
再在本地数据库建一张字段完全一致的表，表名可以不一样：
``` mysql
CREATE TABLE test_table (
    id     int(20) NOT NULL auto_increment,
    name   varchar(32) NOT NULL default '',
    PRIMARY KEY  (id)
)ENGINE=FEDERATED CONNECTION='mysql://username:password@ip:端口/数据库名/test_table';
```
如上声明了引擎为federated，同时与远程表进行了长连接。再次说明一下，远程和本地的两张表名可以不一样。

接下来，对本地test_table表进行增删改查，远程test_table会同时修改，达到同步的效果。

#### 重要提醒
Federated引擎不支持事务操作,所以当你选择使用Federated的时候，你需要优先考虑这个缺点是否会影响你的业务。
我这里是实现两个数据库的同步，所以只需将对远程数据表修改的sql放在事务的最后即可，如果修改远程数据库失败，即报错回滚了。
