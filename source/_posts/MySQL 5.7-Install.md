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
# Windows安装MySQL

## 前提条件

**版本建议使用5.7.21**

1. 获取Mysql安装包，去官网直接下载：https://downloads.mysql.com/archives/community/
2. 解压到D盘根目录:D:/mysql-5.7.21



## 安装

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

   

## 初始化数据库

mysql服务安装成功后，就需要初始化数据库了，否则是无法启动服务的。

在bin目录下执行如下命令：

~~~shell
D:\mysql-5.7.21\bin>mysqld.exe --defaults-file="D:\mysql-5.7.21\my.ini" --initialize --explicit_defaults_for_timestamp
~~~

default-file 即配置文件路径，必须进行指定。

--initialize 说明执行数据库初始化。

--explicit_defaults_for_timestamp说明Timestamp类型的字段，必须进行指定，否则就是NULL。（这个是为了避免麻烦，因为5.6以后的mysql的Timestamp类型字段进行了一些调整）


## 启动/停止数据库

~~~shell
net start mysql
net stop mysql
~~~

## 修改密码

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

# Linux安装MySQL

## 卸载原有的MySQL

1. 关闭正在运行的MySQL

   ` systemctl stop mysql`

2. 查看MySQL是否运行

   - `systemctl status mysql`

   - `ps -ef |grep mysql`

3. 查看原有的MySQL

   `rpm -qa|grep -i mysql`

   ~~~bash
   mysql-community-libs-5.5.62-2.el7.x86_64
   mysql-community-server-5.5.62-2.el7.x86_64
   mysql-community-client-5.5.62-2.el7.x86_64
   mysql-community-common-5.5.62-2.el7.x86_64
   ~~~

4. 卸载原有的MySQL组件

   执行下面的命令依次卸载

   - `yum -y remove mysql-community-client-5.5.62-2.el7.x86_64`
   - `yum -y remove mysql-community-libs-5.5.62-2.el7.x86_64`
   - `yum -y remove mysql-community-common-5.5.62-2.el7.x86_64`

##  安装软件

### 下载解压

~~~bash
#进入自定义的mysql目录
[root@VM_0_17_centos ~]# cd /usr/local/soft/mysql
#下载软件包
[root@VM_0_17_centos mysql]# wget -c https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
#解压包
[root@VM_0_17_centos mysql]# tar -xvf mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
#查看包
[root@VM_0_17_centos mysql]# ls
mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar                 
mysql-community-libs-5.7.23-1.el7.x86_64.rpm
mysql-community-client-5.7.23-1.el7.x86_64.rpm           
mysql-community-libs-compat-5.7.23-1.el7.x86_64.rpm
mysql-community-common-5.7.23-1.el7.x86_64.rpm           
mysql-community-minimal-debuginfo-5.7.23-1.el7.x86_64.rpm
mysql-community-devel-5.7.23-1.el7.x86_64.rpm            
mysql-community-server-5.7.23-1.el7.x86_64.rpm
mysql-community-embedded-5.7.23-1.el7.x86_64.rpm         
mysql-community-server-minimal-5.7.23-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.23-1.el7.x86_64.rpm  
mysql-community-test-5.7.23-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.23-1.el7.x86_64.rpm

~~~

### 依次安装MySQL rpm

~~~bash
[root@VM_0_17_centos mysql]#rpm -ivh mysql-community-server-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-libs-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-client-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-common-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-libs-compat-5.7.23-1.el7.x86_64.rpm --force --nodeps
~~~

### 启动MySQL

~~~bash
[root@VM_0_17_centos mysql]# systemctl start mysqld
[root@VM_0_17_centos mysql]# ps -aux|grep mysqld
mysql     5983  0.0 16.9 1139408 172280 ?      Sl   11:30   0:01 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
root      8268  0.0  0.0 112712   964 pts/1    R+   12:39   0:00 grep --color=auto mysqld
[root@VM_0_17_centos mysql]# 
~~~

> 启动失败，需要对配置文件进行配置

~~~bash
#编辑配置文件
[root@VM_0_17_centos mysql]# vim /etc/my.cnf
#配置文件内容
#数据库存放目录，如果不存在旧创建
datadir=/var/lib/mysql/data
#连接文件，如果不存在则创建，并设置mysql用户所有者和mysql组
socket=/var/lib/mysql/mysql.sock
#定义默认时间戳
explicit_defaults_for_timestamp=true
#设置运行用户
user=mysql
~~~

### 配置开机启动

~~~bash
[root@VM_0_17_centos mysql]# systemctl enable mysqld 
[root@VM_0_17_centos mysql]# systemctl daemon-reload
~~~

### 登录MySQL

1. 查看默认的root密码。

   ~~~bash
   [root@VM_0_17_centos mysql]# cat /var/log/mysqld.log |grep 'temporary password'
   [root@VM_0_17_centos mysql]#[Note] A temporary password is generated for root@localhost: iefe_ggtr; 
   #其中 iefe_ggtr; 就是初始密码了。
   ~~~

2. (可选)查询不到密码，需要初始化数据库，生成默认系统库表

   ~~~bash
   [root@VM_0_17_centos mysql]# mysqld --initialize --user=mysql --console 
   [root@VM_0_17_centos mysql]# grep 'temporary password' /var/log/mysqld.log 
   [Note] A temporary password is generated for root@localhost: iefe_ggtr
   ~~~

3. 登录MySQL

   ~~~bash
   [root@VM_0_17_centos mysql]# mysql -uroot -p
   Enter password: iefe_ggtr
   # 登录成功
   mysql->
   ~~~

4. （可选）使用查询到的默认密码登录失败。

   1. 修改`/etc/my.cnf`，在`[mysqld]`下面添加一行: `skip-grant-tables=1`，让`mysqld`启动时不对密码进行验证

   2. 重启`mysqld`服务 ：`systemctl restart mysqld`

   3. 使用 root 用户登录：`mysql -uroot`

   4. `use mysql`切换到`mysql`数据库，更新`user`表。

      ~~~bash
      # 网上说明执行这条语句，能将root用户的密码修改为root123
      update user set authentication_string = password ( 'root123' ), password_expired = 'N' , password_last_changed = now() where user = 'root' ;
      # 自己实际环境中执行
      update user set authentication_string = password ( 'root123' )where user = 'root' ;
      ~~~

   5. 退出`mysql`，修改`/etc/my.cnf`文件，注释掉之前的`skip-grant-tables=1`。

   6. 重启`mysqld`服务，使用新密码登录即可。

### 修改密码

~~~bash
mysql> set password = password('root123');
~~~

# 参考文章

https://zixuephp.net/article-261.html

https://blog.csdn.net/qq_23077403/article/details/85093705