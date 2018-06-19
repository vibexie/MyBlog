---
title: 使用gradle下载jar
date: 2016-08-17 23:15:29
tags: 工具
---
突然发现个问题，到现在我还不懂Maven和Gradle等构建工具，主要还是工作和学习中，对构建工具的需求不是很大。今天，碰到一个烦恼的Problem，就是我的一个java项目需要下载spring对rabbitMQ的依赖Jars，于是我进入spring.io想下载相关jars。
但是...
就给了两种方式：
* Maven
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.amqp</groupId>
        <artifactId>spring-rabbit</artifactId>
        <version>1.6.1.RELEASE</version>
    </dependency>
</dependencies>
```
<!-- more -->
* Gradle
``` xml
dependencies {
    compile 'org.springframework.amqp:spring-rabbit:1.6.1.RELEASE'
}
```
可是，我现在不会Maven和Gradle呀，我想说现在一些网站都不考虑下我们这些小白的感受啊，我们需要的是J，A，R！
反正我这一下也是肯定学不会Maven或者Gradle了，就算学会了，项目也不肯立马转构建工具，我只要J，A，R！
### 通过Gradle下载jar的方法
* 在你的电脑上安装gradle。
* 新建一个目录，进去创建一个文件build.gradle
``` xml
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
        compile 'org.springframework.amqp:spring-rabbit:1.6.1.RELEASE'
}

task copyJars(type: Copy) {
  from configurations.runtime
  into 'lib' // 目标位置
}
```
* 通过gradle copyJars命令下载jar，下载成功后jar将在lib目录下
``` bash
VibeXie-MBP:gradle2Jars vibexie$ gradle copyJars
:copyJars UP-TO-DATE

BUILD SUCCESSFUL

Total time: 5.018 secs

This build could be faster, please consider using the Gradle Daemon: https://docs.gradle.org/2.13/userguide/gradle_daemon.html
```

虽然说，方式很low，但是这还是能帮我们这些小白快速解决问题。哈哈，现在我要学习下gradle了，不然还怎么装逼~~
