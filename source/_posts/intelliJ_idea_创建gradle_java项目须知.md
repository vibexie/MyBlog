---
title: 'IntelliJ IDEA创建gradle java项目须知'
date: 2018-04-12 08:41:52
tags: java
---
1. 按照正常步骤创建项目后，发现gradle并没有自动生成src等java项目需要目录，解决方案：Preference -> Gradle -> Create directories for empty content roots automatically开
2. build.gradle不会自动sync，解决方案：Preference -> Gradle -> Use auto-import开

