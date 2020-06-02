---
title: Linux常用命令总结
Author: tzcqupt
categories: [Linux]
tags: [Linux,腾讯云]
comments: true
toc: true
date: 2020-06-02 00:00:00
---

# 让客户端和腾讯云一直保持连接

1. 修改`sshd_config`配置文件

   ~~~bash
   [root@VM_0_17_centos ~]# vim /etc/ssh/sshd_config
   # 找到并修改
   #服务端每隔多少秒向客户端发送一个心跳数据
   ClientAliveInterval 30
   #客户端多少次没有响应，服务器自动断开连接
   ClientAliveCountMax 86400
   
   ~~~

2. 重启`sshd`服务

   ~~~bash
   [root@VM_0_17_centos ~]# service sshd restart
   ~~~

   

