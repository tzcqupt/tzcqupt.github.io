---
layout: post
title: MySQL 优化学习
Author: tzcqupt
tags: [ MySQL]
categories: [数据库]
comments: true
toc: true
date: 2020-05-23 00:00:00
---

# 优化前设置

## 开启慢查询日志

~~~sql
show variables like 'slow_query_log'  
--查看是否开启慢查询日志
-- 查看慢查询的日志位置
show variables like '%log%';
set global slow_query_log_file='/var/lib/mysql/VM_0_17_centos-slow.log';
--慢查询日志的位置

set global slow_query_log =on;=on;
--开启慢查询日志

set global long_query_time=1;  
--大于1秒钟的数据记录到慢日志中，如果设置为默认0，则会有大量的信息存储在磁盘中，磁盘很容易满掉
~~~



