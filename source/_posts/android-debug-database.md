---
title: Android Debug Database
date: 2017-05-27 22:21:23
tags: android
---
在Eclipse中，调试Sqlite非常麻烦，需要将数据导出到电脑上，用Sqlite查看工具查看，极大的降低了生产力。

使用Android Studio开发后，你会发现有很多方法对数据进行调试，SQLScout是一个强大的插件，我没有用过，理由很简单，因为要花钱购买才能使用。。。

[Android-Debug-Database](https://github.com/amitshekhariitbhu/Android-Debug-Database) 是最近发现的一个强大的调试Sqlite数据库和SharedPreferences的库，只需你在app build.gradle中加入library
``` shell
debugCompile 'com.amitshekhar.android:debug-db:1.0.0'
```
运行APP后在logcat中会出现下列log
```shell
D/DebugDB: Open http://XXX.XXX.X.XXX:8080 in your browser
```
Of couse, 打开这个链接，就可以在Web页面调试查看App的Sqlite数据库和SharedPreferences了。

最后，做个笔记，一条使用的sql，sqlite3查询所有表的建表语句如下：
``` sql
select * from sqlite_master WHERE type = "table";
```
