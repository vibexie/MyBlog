---
title: 发布Android Library
date: 2016-12-17 00:42:39
tags: android
---
前些天研究了一下如何发布android studio library到远程仓库JCenter中，在网上找了好多方案，貌似都实现不了...

刚好今天有点时间，那就把上次那个问题解决吧。

It is very important！巨坑！

[Bintray](https://bintray.com) 从2016年11月开始，只能以组织公司的名义注册了，respository也从user层级移到了organiztion层级，算是个非常重大的调整了。所以，之前相关发布library到jcenter的教程都可以作废了。

严重提醒：上坑勿跳！
<!-- more -->

----------
总不能在一棵树上吊死，[Bintray](https://bintray.com)不行，咱们就换成 [Jitpack](https://jitpack.io) 吧。jitpack能够非常简单方便的帮你把github上的开源项目以library的形式发布出去，体验了一把，非常爽。

* 创建AS项目，同时添加lib module，最后上传Github
如何通过Android studio新建安卓项目，并且添加module，我就不介绍了，网上的教程都是可以的，以我的一个项目为例，项目结构如下：
![](http://qiniu.vibexie.com/blog/public-android-library-p1.png-width500)

* 项目jitpack相关配置
首先在项目build.gradle中添加

``` xml
buildscript { 
  dependencies {
    classpath 'com.github.dcendents:android-maven-gradle-plugin:1.5' // Add this line
```

&nbsp;&nbsp;&nbsp;&nbsp;然后再在library/build.gradle中添加
``` xml
apply plugin: 'com.github.dcendents.android-maven'  
group='com.github.YourUsername'
```

&nbsp;&nbsp;&nbsp;&nbsp;确认你的git仓库是否有Gradle wrapper，如果没有，就切换到项目根目录下使用gradle wrapper，注意gradle目录不要加入到.gitignore中被忽略。

&nbsp;&nbsp;&nbsp;&nbsp;使用git tag打一个tag，如v1.0，再把标签push到github上。

* 使用github账号登陆jitpack.io

&nbsp;&nbsp;&nbsp;&nbsp;安装下图中的步骤，点击get it。
![](http://qiniu.vibexie.com/blog/public-android-library-p2.png-width800)

&nbsp;&nbsp;&nbsp;&nbsp;点击get it，就会告诉您如何使用library了。

* 在项目用使用远程仓(当你点击get it后会告诉你)
将下列代码加入到你的项目build.gradle中
``` xml
allprojects {
		repositories {
			...
			maven { url 'https://jitpack.io' }
		}
	}
```

&nbsp;&nbsp;&nbsp;&nbsp;再同时添加依赖
``` xml
dependencies {
    compile 'com.github.vibexie:CircularSeekbar:v1.0'
}
```

参考:
1. [Jitpack官方文档](https://jitpack.io/docs/ANDROID/)（不好找）
2. 我Github上的 [这个项目](https://github.com/vibexie/CircularSeekbar) 已经发布到jitpack，相关配置大家可以借鉴。
