---
title: Android Studio使用中的问题
date: 2016-12-09 23:06:25
tags: android
---
之前几个公司都是使用Eclipse开发安卓，所以以前对Eclipse还是非常熟悉，只是对AS学习过一些简单使用，现在新公司开始用AS了，所以对上手AS时遇到的一些问题做一些总结。

### 调试
使用logcat打印日志的时候，无法选择需要过滤的应用，应用选择框显示的是android no debuggable application，logcat胡乱一通把手机上的log全都打印出来了，使得自己无法正常调试bug，这个问题迫切需要解决。

解决方案:
AndroidStudio中 Tools->Android->Enable ADB Integration active.
之后需等待一会，可能adb会重启，之后就会发现那个应用选择框中正常显示你已启动的app了，log也当然会过滤。
<!-- more -->
### 定位错误代码
每次有错误代码时，AS居然不会显示哪行错了，也不会高亮错误代码，简直是巨坑。

解决方案:
取消Power Save Mode。Power Save Mode模式一旦开启，为了省电，所以错误代码也不会给你高亮了。所以需要 File->Power Save Mode 进行关闭。

跳转到下一个错误位置。操作步骤:菜单栏: Navigate —> Next Highlighted Error，快捷键:Mac: Fn + F2,Windows\/Linux: F2

跳转到上一个错误位置。操作步骤:菜单栏: Navigate —> Previous Highlighted Error，快捷键:Mac: Fn + Shift + F2，Windows\/Linux: Shift + F2

### 加速Android Studio/Gradle构建
可查阅 [Sam的博文链接](http://blog.isming.me/2015/03/18/android-build-speed-up/)
