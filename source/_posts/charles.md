---
title: 破解Charles
date: 2016-08-25 01:27:53
tags: 工具
---
#### Charles是用来干嘛的？
Charles是一个抓包工具，和大家用的多的Fiddler差不多。但是，fiddler不支持mac和linux平台，所以Charles是为全平台抓包而生的。的确，fiddler好用，但是如果你是mac或者linux用户，你最好的选择就是Charles！

#### 快快快，我也要抓包！
别急，我们先来安装Charles，当然好软件还是要收费的，当然收费的在大天朝也是有办法破解的。
<!-- more -->
* 在charles官网下载安装（macOS, Windows, Linux都可）
下载地址: [https://www.charlesproxy.com/download/](https://www.charlesproxy.com/download/)

* 下载破解Jar
下载地址: [http://obakk2u63.bkt.clouddn.com/blog/charles.jar](http://obakk2u63.bkt.clouddn.com/blog/charles.jar)

* 替换charles.jar文件（以macOS为例）
找到Charles的安装目录，并进入/Contents/Java/
``` bash
VibeXie-MBP:Java vibexie$ cd /Applications/Charles.app/Contents/Java/
VibeXie-MBP:Java vibexie$ ls
bcpkix-jdk15on-1.52.jar			miglayout-core-5.0.jar
bcprov-jdk15on-1.52.jar			miglayout-swing-5.0.jar
bounce-0.19.jar				org.eclipse.egit.github.core-2.1.5.jar
charles-sni-patch-1.0.jar		protobuf-java-2.6.1.jar
charles.jar				quaqua-9.4.1.jar
flying-saucer-core-9.0.8-SNAPSHOT.jar	rhino-1.7R5.jar
gson-2.3.1.jar				servlet-api-2.4.jar
image4j-0.7.jar				swing-layout-1.0.jar
jasypt-1.9.2.jar			webp-imageio-1.0.jar
jcifs-1.3.17.jar			xmlpull-1.1.3.1.jar
joda-time-2.7.jar			xpp3_min-1.1.4c.jar
json-20140107.jar			xstream-1.4.8.jar
jsyntaxpane-0.9.4.jar
```
再替换为下载下来的charles.jar，重启即可破解成功。


