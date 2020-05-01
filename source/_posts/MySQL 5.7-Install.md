---
layout: post
title: MySQL 5.7安装
Author: tzcqupt
tags: [软件安装, MySQL]
categories: [数据库]
comments: true
toc: true
date: 2020-04-28 00:00:00
---
# 前提条件

**版本建议使用5.7.21**

1. 获取Mysql安装包，去官网直接下载：https://downloads.mysql.com/archives/community/
2. 解压到D盘根目录:D:/mysql-5.7.21



# 安装

若本机已安装mysql，此处将端口设为3307

1. 配置my.ini：在根目录下新建my.ini文件，在[mysqld]下配置：

2. 数据库根目录：basedir = D:/mysql-5.7.21-3307

3. 数据存放目录：datadir = D:/mysql-5.7.21-3307/data

4. 端口：port = 3307

5. 添加字符编码的设置：character-set-server=utf8

   ~~~ini
   [mysql]
   #设置mysql客户端默认字符集
   
   default-character-set=utf8 
   [mysqld]
   #设置3307端口
   port = 3307
   
   #设置mysql的安装目录
   
   basedir = D:/mysql-5.7.21
   datadir = D:/mysql-5.7.21/data
   
   #允许最大连接数
   
   max_connections=200
   
   #服务端使用的字符集默认为8比特编码的latin1字符集
   
   character-set-server=utf8
   
   #创建新表时将使用的默认存储引擎
   
   default-storage-engine=INNODB
   ~~~

   

6. 开始执行安装和添加服务的步骤，在cmd中执行如下命令：

   `D:\mysql-5.7.21\bin>mysqld install mysql --defaults-file="D:/mysql-5.7.21\my.ini"`
   成功安装后会提示：Service successfully installed.

7. 安装成功后服务的名称就是mysql

   

# 初始化数据库

mysql服务安装成功后，就需要初始化数据库了，否则是无法启动服务的。

在bin目录下执行如下命令：

~~~shell
D:\mysql-5.7.21\bin>mysqld.exe --defaults-file="D:\mysql-5.7.21\my.ini" --initialize --explicit_defaults_for_timestamp
~~~

default-file 即配置文件路径，必须进行指定。

--initialize 说明执行数据库初始化。

--explicit_defaults_for_timestamp说明Timestamp类型的字段，必须进行指定，否则就是NULL。（这个是为了避免麻烦，因为5.6以后的mysql的Timestamp类型字段进行了一些调整）


# 启动/停止数据库

~~~shell
net start mysql
net stop mysql
~~~

# 修改密码

接下来就需要登录修改密码了，登录的临时密码在data目录的的日志文件里是”.err”文件，打开搜索”password”关键字。

~~~bash
2019-11-07T02:39:33.180905Z 1 [Note] A temporary password is generated for root@localhost: #JheTLHho2!L
~~~

临时密码就是：`#JheTLHho2!L`

登录mysql：

~~~bash
D:\mysql-5.7.21\bin>mysql -u root -p
Enter password:#JheTLHho2!L
~~~


登录成功后，修改root的密码了：

~~~bash
mysql> set password = password('root123');
~~~