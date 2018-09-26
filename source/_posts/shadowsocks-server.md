---
title: Shadowsocks服务器
date: 2018-09-24 08:12:15
tags: 随笔
---
#### 安装（yum）
```shell
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

#### 使用
后台运行
```shell
ssserver -p 443 -k 密码 -m aes-256-cfb --user nobody -d start
```
停止运行
```shell
ssserver -d stop
```