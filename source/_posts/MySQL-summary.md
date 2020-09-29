---
layout: post
title: MySQL 学习强化总结版
date: 2020-09-24
Author: tzcqupt
tags: [MySQL]
categories: [MySQL]
comments: true
toc: true

---

# MySQL 安装

## Windows安装MySQL

### 前提条件

1. 获取Mysql安装包，去官网直接下载：https://downloads.mysql.com/archives/community/
2. 解压到D盘根目录:D:/mysql-5.7.21

### 安装

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

   

### 初始化数据库

mysql服务安装成功后，就需要初始化数据库了，否则是无法启动服务的。

在bin目录下执行如下命令：

~~~shell
D:\mysql-5.7.21\bin>mysqld.exe --defaults-file="D:\mysql-5.7.21\my.ini" --initialize --explicit_defaults_for_timestamp
~~~

default-file 即配置文件路径，必须进行指定。

--initialize 说明执行数据库初始化。

--explicit_defaults_for_timestamp说明Timestamp类型的字段，必须进行指定，否则就是NULL。（这个是为了避免麻烦，因为5.6以后的mysql的Timestamp类型字段进行了一些调整）


### 启动/停止数据库

~~~shell
net start mysql
net stop mysql
~~~

### 修改密码

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

> 登录其他主机mysql
>
> `mysql -h ip -uroot -p`

登录成功后，修改root的密码了：

~~~bash
mysql> set password = password('root123');
~~~

## Linux安装MySQL

### 卸载原有的MySQL

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

###  安装软件

#### 下载解压

~~~bash
#进入自定义的mysql目录
[root@VM_0_17_centos ~]## cd /usr/local/soft/mysql
#下载软件包
[root@VM_0_17_centos mysql]## wget -c https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
#解压包
[root@VM_0_17_centos mysql]## tar -xvf mysql-5.7.23-1.el7.x86_64.rpm-bundle.tar
#查看包
[root@VM_0_17_centos mysql]## ls
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

#### 依次安装MySQL rpm

~~~bash
[root@VM_0_17_centos mysql]#rpm -ivh mysql-community-server-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-libs-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-client-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-common-5.7.23-1.el7.x86_64.rpm --force --nodeps
rpm -ivh mysql-community-libs-compat-5.7.23-1.el7.x86_64.rpm --force --nodeps
~~~

#### 启动MySQL

~~~bash
[root@VM_0_17_centos mysql]## systemctl start mysqld
[root@VM_0_17_centos mysql]## ps -aux|grep mysqld
mysql     5983  0.0 16.9 1139408 172280 ?      Sl   11:30   0:01 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid
root      8268  0.0  0.0 112712   964 pts/1    R+   12:39   0:00 grep --color=auto mysqld
[root@VM_0_17_centos mysql]## 
~~~

> 启动失败，需要对配置文件进行配置

~~~bash
#编辑配置文件
[root@VM_0_17_centos mysql]## vim /etc/my.cnf
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

#### 配置开机启动

~~~bash
[root@VM_0_17_centos mysql]## systemctl enable mysqld 
[root@VM_0_17_centos mysql]## systemctl daemon-reload
~~~

#### 登录MySQL

1. 查看默认的root密码。

   ~~~bash
   [root@VM_0_17_centos mysql]## cat /var/log/mysqld.log |grep 'temporary password'
   [root@VM_0_17_centos mysql]#[Note] A temporary password is generated for root@localhost: iefe_ggtr; 
   #其中 iefe_ggtr; 就是初始密码了。
   ~~~

2. (可选)查询不到密码，需要初始化数据库，生成默认系统库表

   ~~~bash
   [root@VM_0_17_centos mysql]## mysqld --initialize --user=mysql --console 
   [root@VM_0_17_centos mysql]## grep 'temporary password' /var/log/mysqld.log 
   [Note] A temporary password is generated for root@localhost: iefe_ggtr
   ~~~

3. 登录MySQL

   ~~~bash
   [root@VM_0_17_centos mysql]## mysql -uroot -p
   Enter password: iefe_ggtr
   ## 登录成功
   mysql->
   ~~~

4. （可选）使用查询到的默认密码登录失败。

   1. 修改`/etc/my.cnf`，在`[mysqld]`下面添加一行: `skip-grant-tables=1`，让`mysqld`启动时不对密码进行验证

   2. 重启`mysqld`服务 ：`systemctl restart mysqld`

   3. 使用 root 用户登录：`mysql -uroot`

   4. `use mysql`切换到`mysql`数据库，更新`user`表。

      ~~~bash
      ## 网上说明执行这条语句，能将root用户的密码修改为root123
      update user set authentication_string = password ( 'root123' ), password_expired = 'N' , password_last_changed = now() where user = 'root' ;
      ## 自己实际环境中执行
      update user set authentication_string = password ( 'root123' )where user = 'root' ;
      ~~~

   5. 退出`mysql`，修改`/etc/my.cnf`文件，注释掉之前的`skip-grant-tables=1`。

   6. 重启`mysqld`服务，使用新密码登录即可。

#### 修改密码

~~~bash
mysql> set password = password('root123');
~~~

### 重置密码

~~~bash
# 修改配置文件
vim /etc/my.cnf
# 将skip-grant-tables注释掉
# 刷新配置
systemctl restart mysqld
# 直接登录 mysql -uroot -p
# 修改密码
>mysql update user set authentication_string=password('newpassword')  where user='root' 
# 退出mysql,编辑配置文件,去掉skip-grant-tables的注释
# 使用新密码登录mysql
~~~



## 参考文章

https://zixuephp.net/article-261.html

https://blog.csdn.net/qq_23077403/article/details/85093705

# MySQL 数据类型

## 整数

| 类型                | 字节数 | 无符号值范围         |
| :------------------ | :----- | :------------------- |
| `unsigned tinyint`  | 1      | 0 到 255             |
| `unsigned smallint` | 2      | 0 到 65535           |
| `unsigned int`      | 4      | 0 到 43亿            |
| `unsigned bigint`   | 8      | 0 到10 <sup>19</sup> |

`int(n)` 无论 n 等于多少, int永远占4个字节;n 表示的是显示宽度，不足的用 0 补足，超过的无视长度而直接显示整个数字，但这要整型设置了 `unsigned zerofill` 才有效

~~~sql
mysql> CREATE TABLE test3 (
       `a` int,
       `b` int(5),
       `c` int(5) unsigned,
       `d` int(5) zerofill,
       `e` int(5) unsigned zerofill,
       `f` int    zerofill,
       `g` int    unsigned zerofill
     );
Query OK, 0 rows affected (0.01 sec)

mysql> insert into test3 values (1,1,1,1,1,1,1),(11,11,11,11,11,11,11),(12345,12345,12345,12345,12345,12345,12345);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> select * from test3;
+-------+-------+-------+-------+-------+------------+------------+
| a     | b     | c     | d     | e     | f          | g          |
+-------+-------+-------+-------+-------+------------+------------+
|     1 |     1 |     1 | 00001 | 00001 | 0000000001 | 0000000001 |
|    11 |    11 |    11 | 00011 | 00011 | 0000000011 | 0000000011 |
| 12345 | 12345 | 12345 | 12345 | 12345 | 0000012345 | 0000012345 |
+-------+-------+-------+-------+-------+------------+------------+
3 rows in set (0.00 sec)

mysql> show create table test3;
| Table | Create Table                                                   
| test3 | CREATE TABLE `test3` (
  `a` int(11) DEFAULT NULL,
  `b` int(5) DEFAULT NULL,
  `c` int(5) unsigned DEFAULT NULL,
  `d` int(5) unsigned zerofill DEFAULT NULL,
  `e` int(5) unsigned zerofill DEFAULT NULL,
  `f` int(10) unsigned zerofill DEFAULT NULL,
  `g` int(10) unsigned zerofill DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)
~~~

> 原始的d字段是`有符号`的,使用了`zerofill`会自动转为无符号

## 浮点数

float/double/decimal

1. float 数值类型用于表示单精度浮点数值，而 double 数值类型用于表示双精度浮点数值，float 和 double 都是浮点型，而 decimal 是定点型。

2. 浮点型和定点型可以用类型名称后加（M，D）来表示(如经纬度`decimal(10,7)`)，M 表示该值的总共长度，D 表示小数点后面的长度，M 和 D 又称为精度和标度。

3. float 和 double 在不指定精度时，默认会按照实际的精度来显示，而 DECIMAL 在不指定精度时，默认整数为 10，小数为 0。

4. decimal 采用的是**四舍五入**,float 和 double 采用的是**四舍六入五成双**

   > **四舍六入五成双**:就是 5 以下舍弃 5 以上进位，如果需要处理数字为 5 的时候，需要看 5 后面是否还有不为 0 的任何数字，如果有，则直接进位，如果没有，需要看 5 前面的数字，若是奇数则进位，若是偶数则将 5 舍掉

5. `float`、`double`会存在精度问题，`decimal`精度正常的

## 字符串



| 类型       | 范围                                 | 存储所需字节 | 说明                            |
| :--------- | :----------------------------------- | :----------- | :------------------------------ |
| char(M)    | [0,m]，m 的范围 [0,2<sup>8</sup>-1]  | m            | 定长字符串                      |
| varchar(M) | [0,m]，m 的范围 [0,2<sup>16</sup>-1] | m            | 0-65535 字节                    |
| tinyblob   | 0-255(2<sup>8</sup>-1) 字节          | L+1          | 不超过 255 个字符的二进制字符串 |
| blob       | 0-65535(2<sup>16</sup>-1) 字节       | L+2          | 二进制形式的长文本数据          |
| mediumblob | 0-16777215(2<sup>24</sup>-1) 字节    | L+3          | 二进制形式的中等长度文本数据    |
| longblob   | 0-4294967295(2<sup>32</sup>-1) 字节  | L+4          | 二进制形式的极大文本数据        |
| tinytext   | 0-255(2<sup>8</sup>-1) 字节          | L+1          | 短文本字符串                    |
| text       | 0-65535(2<sup>16</sup>-1) 字节       | L+2          | 长文本数据                      |
| mediumtext | 0-16777215(2<sup>24</sup>-1) 字节    | L+3          | 中等长度文本数据                |
| longtext   | 0-4294967295(2<sup>32</sup>-1) 字节  | L+4          | 极大文本数据                    |

1. char 类型占用固定长度，如果存放的数据为固定长度的建议使用 char 类型，如：手机号码、身份证等固定长度的信息。

2. 表格中的 L 表示存储的数据本身占用的字节，L 以外所需的额外字节为存放该值的长度所需的字节数。

   MySQL 通过存储值的内容及其长度来处理可变长度的值，这些额外的字节是无符号整数。

## 日期

| 类型      | 字节大小 | 范围                                                         | 格式                | 用途                     |
| :-------- | :------- | :----------------------------------------------------------- | :------------------ | :----------------------- |
| DATE      | 3        | 1000-01-01/9999-12-31                                        | YYYY-MM-DD          | 日期值                   |
| TIME      | 3        | '-838:59:59'/'838:59:59'                                     | HH:MM:SS            | 时间值或持续时间         |
| YEAR      | 1        | 1901/2155                                                    | YYYY                | 年份值                   |
| DATETIME  | 8        | 1000-01-01 00:00:00/9999-12-31 23:59:59                      | YYYY-MM-DD HH:MM:SS | 混合日期和时间值         |
| TIMESTAMP | 4        | 1970-01-01 00:00:00/2038 结束时间是第 **2147483647** 秒，北京时间 **2038-1-19 11:14:07**，格林尼治时间 2038 年 1 月 19 日 凌晨 03:14:07 | YYYYMMDD HHMMSS     | 混合日期和时间值，时间戳 |

## 数据类型选择建议

- **选小不选大**：一般情况下选择可以正确存储数据的最小数据类型，越小的数据类型通常更快，占用磁盘，内存和 CPU 缓存更小。
- **简单就好**：简单的数据类型的操作通常需要更少的 CPU 周期，例如：整型比字符操作代价要小得多，因为字符集和校对规则 (排序规则) 使字符比整型比较更加复杂。
- **尽量避免 NULL**：尽量制定列为 NOT NULL，除非真的需要 NULL 类型的值，有 NULL 的列值会使得索引、索引统计和值比较更加复杂。
- **浮点类型的建议统一选择 decimal**
- **记录时间的建议使用 int 或者 bigint 类型，将时间转换为时间戳格式，如将时间转换为秒、毫秒，进行存储，方便走索引**

# DDL 操作数据库、表

##  数据库CRUD

1. C(Create):创建
   * 创建数据库  
     `create database 数据库名称;`
   * 创建数据库，判断不存在，再创建  
     `create database if not exists 数据库名称;`
   * 创建数据库，并指定字符集  
     `create database 数据库名称 character set 字符集名;`
   * 练习:创建db4数据库，判断是否存在，并制定字符集为gbk  
     `create database if not exists db4 character set gbk;`
2. R(Retrieve):查询
   * 查询所有数据库的名称  
     `show databases;`
   * 查询某个数据库的字符集:查询某个数据库的创建语句  
     `show create database 数据库名称;`
3. U(Update):修改  
   修改数据库的字符集  
    `alter database 数据库名称 character set 字符集名称;`
4. D(Delete):删除
   * 删除数据库  
     `drop database 数据库名称;`
   * 判断数据库存在，存在再删除  
     `drop database if exists 数据库名称;`
5. 使用数据库  
   * 查询当前正在使用的数据库名称  
     `select database();`
   * 使用数据库  
     `use 数据库名称;`

## 操作表CRUD

### C(Create):创建

- 语法  

  ~~~mysql
  create table 表名(
                      列名1 数据类型1,
                      列名2 数据类型2,
                      ....
                      列名n 数据类型n
                   );
  ~~~

     >注意：最后一列，不需要加逗号(,)

- 创建表

  ~~~mysql
  create table student(
  id int,
  name varchar(32),
  age int ,
  score double(4,1),
  birthday date,
  insert_time timestamp
  );
  ~~~

- 复制表：  
  `create table 表名 like 被复制的表名;`	

### R(Retrieve)：查询

* 查询某个数据库中所有的表名称  
  `show tables;`
* 查询表结构  
  `desc 表名;`

### U(Update):修改

1. 修改表名  
   `alter table 表名 rename to 新的表名;`
2. 修改表的字符集  
   `alter table 表名 character set 字符集名称;`
3. 添加一列  
   `alter table 表名 add 列名 数据类型;`
4. 修改列名称 类型  
   `alter table 表名 change 列名 新列别 新数据类型;`  
   `alter table 表名 modify 列名 新数据类型;`
5. 删除列  
   `alter table 表名 drop 列名;`
6. D(Delete):删除  
   `drop table 表名;`  
   `drop table  if exists 表名 ;`

# DML 增删改表中数据

## 添加数据

  `insert into 表名(列名1,列名2,...列名n) values(值1,值2,...值n);`

1. 列名和值要一一对应。    
2. 如果表名后，不定义列名，则默认给所有列添加值   
   `insert into 表名 values(值1,值2,...值n);`  	
3. 除了数字类型，其他类型需要使用引号(单双都可以)引起来

## 删除数据  

单表删除  `delete from 表名 [where 条件];`  
多表删除`delete [别名1,别名2] from 表1 [[as] 别名1],表2 [[as] 别名2] [where条件];`

1. 如果不加条件，则删除表中所有记录。  
2. 如果要删除所有记录  
   1. `delete from 表名;` -- ***不推荐使用***。有多少条记录就会执行多少次删除操作  
   2. `TRUNCATE TABLE 表名;` -- **推荐使用**，效率更高 先删除表，然后再创建一张一样的表。

## 修改数据

```mysql
update 表名 set 列名1 = 值1, 列名2 = 值2,... [where 条件];
```

>如果不加任何条件，则会将表中所有记录全部修改。

# DQL 查询表中的记录 简单查询

   `select * from 表名;`

## 语法

~~~mysql
select
	字段列表
from
	表名列表
where
	条件列表
group by
	分组字段
having
	分组之后的条件
order by
	排序
limit
	分页限定
~~~

## 基础查询

1. 多个字段的查询  
   `select 字段名1，字段名2... from 表名;`  

   >如果查询所有字段，则可以使用*来替代字段列表。

2. 去除重复  
   `distinct`

3. 计算列

   1. 一般可以使用四则运算计算一些列的值。（一般只会进行数值型的计算）
   2. `ifnull`(列名,默认值)  
      判断列名的值是否为 NULL,如果为 NULL 则返回第二个参数,如果不为 NULL 则返回第一个参数的值。  

4. 起别名`as`  

   >as也可以省略

## 条件查询

1. where子句后跟条件

2. 运算符

   * `> 、< 、<= 、>= 、= 、<>`
   * `BETWEEN...AND `
   * `IN(集合)`
   * `LIKE：模糊查询`  
     占位符：
   * `_` 单个任意字符
   * `%` 多个任意字符
   * `IS NULL ` 
   * `and` 或 `&&`
   * `or ` 或 `||` 
   * ` not`  或 `!`

   > 比较:数值按照大小比较,字符按照`ASCII`码对应的值进行比较,比较时按照字符对应的位置一个字符一个字符的比较。

- 查询年龄大于20岁

  `SELECT * FROM student WHERE age > 20;`     

  `SELECT * FROM student WHERE age >= 20;`

- 查询年龄等于20岁  
  `SELECT * FROM student WHERE age = 20;`

- 查询年龄不等于20岁  
  `SELECT * FROM student WHERE age != 20;`

  `SELECT * FROM student WHERE age <> 20;`

- 查询年龄大于等于20 小于等于30  

  `SELECT * FROM student WHERE age >= 20 &&  age <=30;`

  `SELECT * FROM student WHERE age >= 20 AND  age <=30;`

  `SELECT * FROM student WHERE age BETWEEN 20 AND 30;`

- 查询年龄22岁，18岁，25岁的信息  
  `SELECT * FROM student WHERE age = 22 OR age = 18 OR age = 25`

  `SELECT * FROM student WHERE age IN (22,18,25);`

- 查询英语成绩为null  
  `SELECT * FROM student WHERE english = NULL; `

   >不对的。null值不能使用 = (!=) 判断

  `SELECT * FROM student WHERE english IS NULL;`

- 查询英语成绩不为null  
  `SELECT * FROM student WHERE english  IS NOT NULL;`

- 查询姓马的有哪些？ like    
  `SELECT * FROM student WHERE NAME LIKE '马%';`

- 查询姓名第二个字是化的人     
  `SELECT * FROM student WHERE NAME LIKE "_化%";`

- 查询姓名是3个字的人    
  `SELECT * FROM student WHERE NAME LIKE '___';`

- 查询姓名中包含德的人   
  `SELECT * FROM student WHERE NAME LIKE '%德%';`

## NULL的坑

**查询运算符、like、between and、in、not in对NULL值查询不起效。**

**IS NULL/IS NOT NULL（NULL值专用查询）**

## 经典问题

~~~mysql
select * from students;
select * from students where name like '%';
~~~

当name没有NULL值时，返回的结果一样。

当name有NULL值时，**第2个sql查询不出name为NULL的记录**。

## 总结

- **like中的%可以匹配一个到多个任意的字符，_可以匹配任意一个字符**
- **空值查询需要使用IS NULL或者IS NOT NULL，其他查询运算符对NULL值无效**
- **建议创建表的时候，尽量设置表的字段不能为空，给字段设置一个默认值**

# DQL 查询语句 复杂查询

## 排序查询

* 语法：order by 子句  
  `order by 排序字段1 排序方式1, 排序字段2 排序方式2...`
* 排序方式
  * `ASC`：升序，默认的。
  * `DESC`：降序。

>如果有多个排序条件，则当前边的条件值一样时，才会判断第二条件。  
>如学生有年龄和成绩,先按照成绩降序排名,如果成绩一样,按照年龄升序排名给出结果

## 聚合函数 

将一列数据作为一个整体，进行纵向的计算。

1. `count`：计算个数
   1. 一般选择非空的列：主键
   2. `count(*)`
2. `max`：计算最大值
3. `min`：计算最小值
4. `sum`：计算和
5. `avg`：计算平均值

>聚合函数的计算，排除`null`值。  
>
>1. 选择不包含非空的列进行计算  
>2. `IFNULL`函数

## 分组查询

1. group by 分组字段；

   >分组之后查询的字段：分组字段、聚合函数

2. where 和 having 的区别？  

   1. where 在**分组之前**进行限定，如果不满足条件，则不参与分组。having在**分组之后**进行限定，如果不满足结果，则不会被查询出来
   2. where 后不可以跟聚合函数，**having可以进行聚合函数的判断**。

- 按照性别分组。分别查询男、女同学的平均分  

  `SELECT sex , AVG(math) FROM student GROUP BY sex;`

- 按照性别分组。分别查询男、女同学的平均分,人数  

  `SELECT sex , AVG(math),COUNT(id) FROM student GROUP BY sex;`

- 按照性别分组。分别查询男、女同学的平均分,人数 要求：分数低于70分的人，不参与分组  
  `SELECT sex , AVG(math),COUNT(id) FROM student WHERE math > 70 GROUP BY sex;`

- 按照性别分组。分别查询男、女同学的平均分,人数 要求：分数低于70分的人，不参与分组,分组之后。人数要大于2个人  
  `SELECT sex , AVG(math),COUNT(id) FROM student WHERE math > 70 GROUP BY sex HAVING COUNT(id) > 2;`  
  `SELECT sex , AVG(math),COUNT(id) 人数 FROM student WHERE math > 70 GROUP BY sex HAVING 人数 > 2;`	

## 分页查询

1. 语法：`limit` 开始的索引,每页查询的条数;

   > limit [offset,] count;
   >
   > offset 表示偏移量,就是跳过多少行记录,offset可以省略,默认为0,表示跳过0行
   >
   > count 表示offset行之后开始取数据,取count行记录

2. 公式：开始的索引 = （当前的页码 - 1） * 每页显示的条数<==>当前页=(索引/条数)+1  
   每页显示3条记录 

   `SELECT * FROM student LIMIT 0,3;` ===> 第1页=(0/3)+1

   `SELECT * FROM student LIMIT 3,3;` ===> 第2页=(3/3)+1

   `SELECT * FROM student LIMIT 6,3;` ===> 第3页=(6/3)+1

   ```mysql
   select 列 from 表名 limit (page - 1) * pageSize,pageSize;
   ```

>`limit` 是一个MySQL"方言",limit开始的索引默认为0,从0开始计数

### 实例

#### 获取前n行记录

```mysql
select 列 from 表 limit 0,n;
#等价于
select 列 from 表 limit n;
```

#### 获取最大的一条记录

如我们需要获取订单金额最大的一条记录，可以这么做：先按照金额降序，然后取第一条记录

~~~mysql
select a.id 订单编号,a.price 订单金额 from t_order a order by a.price desc limit 1;
#等价于
select a.id 订单编号,a.price 订单金额 from t_order a order by a.price desc limit 0,1;
~~~

#### 获取排名第n到m的记录

我们需要先跳过n-1条记录，然后取m-n+1条记录

~~~mysql
select 列 from 表 limit n-1,m-n+1;
~~~

## 联合查询

有学生表，分数表，课程表==>查询每门功课成绩最好的前两名

`union 不能在 order by/limit后使用`。可以使用select a.* from(select…order by s.score limit 0,2)a union all select b.* from(..) b

~~~mysql
select a.* from (select s.*, c.c_name, s2.s_score
from student s
         join score s2 on s.s_id = s2.s_id
         left join course c on s2.c_id = c.c_id
where s2.c_id = "01" order by s2.s_score desc limit 0,2) a
union all
select b.* from (select s.*, c.c_name, s2.s_score
from student s
         join score s2 on s.s_id = s2.s_id
         left join course c on s2.c_id = c.c_id
where s2.c_id = "02"  order by s2.s_score desc limit 0,2) b
union all
select c.* from (select s.*, c.c_name, s2.s_score
from student s
         join score s2 on s.s_id = s2.s_id
         left join course c on s2.c_id = c.c_id
where s2.c_id = "03"
order by s_score desc limit 0,2)c
~~~

# MySQL 函数

## 数值型函数

| 函数名称        | 作 用                                                      |
| :-------------- | :--------------------------------------------------------- |
| abs             | 求绝对值                                                   |
| sqrt            | 求二次方根                                                 |
| mod             | 求余数                                                     |
| ceil 和 ceiling | 两个函数功能相同，都是返回不小于参数的最小整数，即向上取整 |
| floor           | 向下取整，返回值转化为一个BIGINT                           |
| rand            | 生成一个0~1之间的随机数，传入整数参数是，用来产生重复序列  |
| round           | 对所传参数进行四舍五入                                     |
| sign            | 返回参数的符号                                             |
| pow 和 power    | 两个函数的功能相同，都是所传参数的次方的结果值             |
| sin             | 求正弦值                                                   |
| asin            | 求反正弦值，与函数 SIN 互为反函数                          |
| cos             | 求余弦值                                                   |
| acos            | 求反余弦值，与函数 COS 互为反函数                          |
| tan             | 求正切值                                                   |
| atan            | 求反正切值，与函数 TAN 互为反函数                          |
| cot             | 求余切值                                                   |

## 字符串函数

| 函数名称            | 作 用                                                        |
| :------------------ | :----------------------------------------------------------- |
| length              | 计算字符串长度函数，返回字符串的字节长度                     |
| concat              | 合并字符串函数，返回结果为连接参数产生的字符串，参数可以使一个或多个 |
| insert              | 替换字符串函数                                               |
| lower               | 将字符串中的字母转换为小写                                   |
| upper               | 将字符串中的字母转换为大写                                   |
| left                | 从左侧字截取符串，返回字符串左边的若干个字符                 |
| right               | 从右侧字截取符串，返回字符串右边的若干个字符                 |
| trim                | 删除字符串左右两侧的空格                                     |
| replace             | 字符串替换函数，返回替换后的新字符串                         |
| substr 和 substring | 截取字符串，返回从指定位置开始的指定长度的字符换             |
| reverse             | 字符串反转（逆序）函数，返回与原始字符串顺序相反的字符串     |

## 日期和时间函数

| 函数名称                | 作 用                                                        |
| :---------------------- | :----------------------------------------------------------- |
| curdate 和 current_date | 两个函数作用相同，返回当前系统的日期值                       |
| curtime 和 current_time | 两个函数作用相同，返回当前系统的时间值                       |
| now 和 sysdate          | 两个函数作用相同，返回当前系统的日期和时间值                 |
| unix_timestamp          | 获取UNIX时间戳函数，返回一个以 UNIX 时间戳为基础的无符号整数 |
| from_unixtime           | 将 UNIX 时间戳转换为时间格式，与UNIX_TIMESTAMP互为反函数     |
| month                   | 获取指定日期中的月份                                         |
| monthname               | 获取指定日期中的月份英文名称                                 |
| dayname                 | 获取指定曰期对应的星期几的英文名称                           |
| dayofweek               | 获取指定日期是一周中是第几天，返回值范围是1~7,1=周日         |
| week                    | 获取指定日期是一年中的第几周，返回值的范围是否为 0〜52 或 1〜53 |
| dayofyear               | 获取指定曰期是一年中的第几天，返回值范围是1~366              |
| dayofmonth              | 获取指定日期是一个月中是第几天，返回值范围是1~31             |
| year                    | 获取年份，返回值范围是 1970〜2069                            |
| time_to_sec             | 将时间参数转换为秒数                                         |
| sec_to_time             | 将秒数转换为时间，与TIME_TO_SEC 互为反函数                   |
| date_add 和 adddate     | 两个函数功能相同，都是向日期添加指定的时间间隔               |
| date_sub 和 subdate     | 两个函数功能相同，都是向日期减去指定的时间间隔               |
| addtime                 | 时间加法运算，在原始时间上添加指定的时间                     |
| subtime                 | 时间减法运算，在原始时间上减去指定的时间                     |
| datediff                | 获取两个日期之间间隔，返回参数 1 减去参数 2 的值             |
| date_format             | 格式化指定的日期，根据参数返回指定格式的值                   |
| weekday                 | 获取指定日期在一周内的对应的工作日索引                       |

## MySQL 聚合函数

| 函数名称 | 作用                             |
| :------- | :------------------------------- |
| max      | 查询指定列的最大值               |
| min      | 查询指定列的最小值               |
| count    | 统计查询结果的行数               |
| sum      | 求和，返回指定列的总和           |
| avg      | 求平均值，返回指定列数据的平均值 |

## MySQL 流程控制函数

| 函数名称 | 作用           |
| :------- | :------------- |
| if       | 判断，流程控制 |
| ifnull   | 判断是否为空   |
| case     | 搜索语句       |

1. IF(expr,v1,v2)

   当 expr 为真是返回 v1 的值，否则返回 v2

2. IFNULL(v1,v2)

   v1为空返回v2，否则返回v1。 

3. case相当于Java的if...else if.. else

   **方式1：**

   ```mysql
   CASE  <表达式>
      WHEN <值1> THEN <操作>
      WHEN <值2> THEN <操作>
      ...
      ELSE <操作>
   END CASE;
   ```

   **方式2：**

   ```mysql
   CASE
       WHEN <条件1> THEN <命令>
       WHEN <条件2> THEN <命令>
       ...
       ELSE commands
   END CASE;
   ```

   1. 数据准备

      ~~~mysql
      CREATE TABLE t_stu (
        id   INT AUTO_INCREMENT  COMMENT '编号',
        name VARCHAR(10) COMMENT '姓名',
        sex  TINYINT COMMENT '性别,0:未知,1:男,2:女',
        PRIMARY KEY (id)
      ) COMMENT '学生表';
      
      ~~~

   2. 方式一写法

      ~~~mysql
      SELECT
            t.name 姓名,
            (CASE t.sex
             WHEN 1
               THEN '男'
             WHEN 2
               THEN '女'
             ELSE '未知' END) 性别
          FROM t_stu t;
      ~~~

   3. 方式二写法

      ~~~mysql
      SELECT
          t.name          姓名,
          (CASE
          WHEN t.sex = 1
          THEN '男'
          WHEN t.sex = 2
          THEN '女'
          ELSE '未知' END) 性别
          FROM t_stu t;
      ~~~

# MySQL 查询深入

## 连接查询

### 笛卡尔积

#### 解释

有两个集合A和B，笛卡尔积表示A集合中的元素和B集合中的元素任意相互关联产生的所有可能的结果。

假如A中有m个元素，B中有n个元素，**A、B笛卡尔积**产生的结果有**m*n**个结果，相当于循环遍历两个集合中的元素，任意组合。

#### 语法

```mysql
select 字段 from 表1,表2[,表N];
#或者
select 字段 from 表1 join 表2 [join 表N];
```

#### Java 伪代码

~~~java
for(Object eleA : A){
    for(Object eleB : B){
        System.out.print(eleA+","+eleB);
    }
}
~~~



### 内连接

#### 解释

内连接相当于在笛卡尔积的基础上加上了连接的条件。

当没有连接条件的时候，内连接上升为笛卡尔积。

#### 语法

```mysql
select 字段 from 表1 inner join 表2 on 连接条件;
select 字段 from 表1 join 表2 on 连接条件;
select 字段 from 表1, 表2 [where 关联条件];
```

#### Java 伪代码

~~~
for(Object eleA : A){
    for(Object eleB : B){
        if(连接条件是否为true){
            System.out.print(eleA+","+eleB);
        }
    }
}
~~~

### 外连接

分为:主表和从表，要查询的信息**主要**来自于哪个表，谁就是主表。

外连接查询结果为主表中所有记录。如果从表中有和它匹配的，则显示匹配的值，这部分相当于内连接查询出来的结果；如果从表中没有和它匹配的，则显示null。

**左外链接：使用left join关键字，left join左边的是主表。**

**右外连接：使用right join关键字，right join右边的是主表。**

## 子查询

### 分类

- 标量子查询（结果集只有一行一列）
- 列子查询（结果集只有一列多行）
- 行子查询（结果集有一行多列）
- 表子查询（结果集一般为多行多列）

**按子查询出现在主查询中的不同位置分**

- **select后面**：仅仅支持标量子查询。

- **from后面**：支持表子查询

  >将子查询的结果集充当`一张表`，要求必须起别名，否者这个表找不到。然后将真实的表和子查询结果表进行连接查询。

- **where或having后面**：支持标量子查询（单列单行）、列子查询（单列多行）、行子查询（多列多行）

- **exists后面（即相关子查询）**：表子查询（多行、多列）

## MySQL 连接查询内部如何优化

msql内部使用了一个内存缓存空间，就叫他`join_buffer`吧，先把外循环的数据放到`join_buffer`中，然后对从表进行遍历，从表中取一条数据和`join_buffer`的数据进行比较，然后从表中再取第2条和`join_buffer`数据进行比较，直到从表遍历完成，使用这方方式来减少从表的io扫描次数，当`join_buffer`足够大的时候，大到可以存放主表所有数据，那么从表只需要全表扫描一次（即只需要一次全表io读取操作）。

mysql中这种方式叫做`Block Nested Loop`。

# MySQL 事务

## 事务的特性

### 原子性(Atomicity)

事务的整个过程如原子操作一样，最终要么全部成功，或者全部失败，这个原子性是从最终结果来看的，从最终结果来看这个过程是不可分割的。

### 一致性(Consistency)

事务开始之前、执行中、执行完毕，这些时间点，多个人去观察事务操作的数据的时候，看到的数据都是一致的，比如在事务操作过程中，A连接看到的是100，那么B此时也去看的时候也是100，不会说AB看到的数据不一样，他们在某个时间点看到的数据是一致的。

### 隔离性(Isolation)

一个事务的执行不能被其他事务干扰。即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

### 持久性(Durability)

一个事务一旦提交，他对数据库中数据的改变就应该是永久性的。当事务提交之后，数据会持久化到硬盘，修改是永久性的。

## 事务中的问题

### 脏读

一个事务在执行的过程中读取到了**其他事务还没有提交**的数据。

### 读已提交

一个事务操作过程中可以读取到其他事务已经提交的数据。

事务中的每次读取操作，读取到的都是数据库中**其他事务已提交**的最新的数据（相当于当前读）

### 可重复读

一个事务操作中对于一个读取操作不管多少次，读取到的结果都是一样的。

### 幻读

幻读现象例子：

可重复读模式下，比如有个用户表，手机号码为主键，有两个事物进行如下操作

事务A操作如下：
1、打开事务
2、查询号码为X的记录，不存在
3、插入号码为X的数据，插入报错（为什么会报错，先向下看）
4、查询号码为X的记录，发现还是不存在（由于是可重复读，所以读取记录X还是不存在的）

事物B操作：在事务A第2步操作时插入了一条X的记录，所以会导致A中第3步插入报错（违反了唯一约束）

上面操作对A来说就像发生了幻觉一样，明明查询X（A中第二步、第四步）不存在，但却无法插入成功

**幻读可以这么理解：事务中后面的操作（插入号码X）需要上面的读取操作（查询号码X的记录）提供支持，但读取操作却不能支持下面的操作时产生的错误，就像发生了幻觉一样。**

## 事务的隔离级别

### 隔离级别

1. **读未提交：READ-UNCOMMITTED**
2. **读已提交：READ-COMMITTED**
3. **可重复读：REPEATABLE-READ**
4. **串行：SERIALIZABLE**

### 隔离级别出现的问题

| 隔离级别         | 脏读 | 不可重复读 | 幻读 |
| :--------------- | :--- | :--------- | :--- |
| READ-UNCOMMITTED | 有   | 有         | 无   |
| READ-COMMITTED   | 无   | 有         | 无   |
| REPEATABLE-READ  | 无   | 无         | 有   |
| SERIALIZABLE     | 无   | 无         | 无   |

# MySQL 踩坑点

## NULL

- **NULL作为布尔值的时候，不为1也不为0**
- **任何值和NULL使用运算符（>、<、>=、<=、!=、<>）或者（in、not in、any/some、all），返回值都为NULL**
- **当IN和NULL比较时，无法查询出为NULL的记录**
- **当NOT IN 后面有NULL值时，不论什么情况下，整个sql的查询结果都为空**
- **判断是否为空只能用IS NULL、IS NOT NULL**
- **count(字段)无法统计字段为NULL的值，count(\*)可以统计值为null的行**
- **当字段为主键的时候，字段会自动设置为not null**
- **NULL导致的坑让人防不胜防，强烈建议创建字段的时候字段不允许为NULL，给个默认值**

## 枚举ENUM

### ENUM 枚举值字面量和内部索引

#### 字面量

在创建表结构时，`MySQL` 会自动删除 `ENUM` 枚举值的尾随空格，例如会把 `'medium '` 转换成 `'medium'`。

#### 内部索引

ENUM 类型中的任何一个枚举值都有一个内部的数字索引.

1. 创建表结构时指定的枚举值都会分配一个内部索引,索引的小标从`1`开始.
2. 空字符串错误值的索引为 `0`，这样，我们可以直接使用 `0` 值来查询那些插入的或更新的无效的枚举值

3.  `NULL` 值的索引为 `NULL`

4. `ENUM` 最多只能包含 `65,535` 个不同的枚举值

   > 在老版本,或者取消了严格模式的MySQL中,如果在 `ENUM` 列中插入无效值（即，允许值列表中不存在的字符串），则会插入空字符串 ( `''` ) 作为特殊错误值，这个空字符串可以通过此字符串具有数字值 `0` 来区分 **正常** 的空字符串

当在 ENUM 列上使用 `SUM()` 或 `AVG()` 等聚合函数时，因为这些函数的参数必须是一个数字，所以 `MySQL` 会自动使用它们的索引值作为参数。也就是说，对于需要计算的场景，都会使用内部索引。其实，真实的枚举值，只有在插入或者显示或者查询时才会用到。

```mysql
numbers ENUM('0','1','2')
#枚举成员的字符串值为 '0'、'1' 和 '2'，而数字索引值为 1 、2 和 3
```

### ENUM 中的 NULL 或空 `''` 值问题

1. 如果在 ENUM 列中插入无效值（即，允许值列表中不存在的字符串），则会插入空字符串 ( `''` ) 作为特殊错误值，这个特殊错误值空字符串的索引为 `0` ，从而与实际的 **正常** 的空字符串 ( 索引大于 1 ) 区分开来

   >当然了，如果启用了严格的 SQL 模式 ( `sql_mode` ) ，尝试插入无效的 ENUM 值会导致错误

2. 如果一个 `ENUM` 列添加了 `NULL` 约束，那么这个 `ENUM` 列就允许 `NULL` 值，且默认的值就是 `NULL`
3. 如果一个 `ENUM` 列添加了 `NOT NULL` 约束，那么它的默认值就是第一个枚举值。

### ENUM 枚举值的排序问题

因为 `ENUM` 类型存储的是枚举值的内部索引，所以 ENUM 值根据其索引号进行排序，具体显示出来，则取决于定义列是的枚举成员顺序。

例如，如果在定义列时，指定了 `'b'` 在 `'a'` 前面 `('b'，'a')`，那么 `'b'` 的顺序就会在 `'a'` 之前，且空字符串在非空字符串之前排序，`NULL` 值在所有其他枚举值之前排序

也就是排序的顺序默认是 `NULL '' 'b' 'a'`

> 1. 指定`ENUM`列的排序顺序使用字母顺序表
> 2. 使用 `ORDER BY CAST (col AS CHAR)` 或 `ORDER BY CONCAT(col)` 确保 enum 列按词法排序而不是索引编号排序

### 数据类型的限制

1. 枚举值不能是表达式,即使该表达式用于计算字符串值
2. 不能使用用户变量作为枚举值
3. **建议不要使用数字用作枚举值**

## 分页

`limit [offset,] count;`

1. limit 中offset和count的值不能用表达式

2. limit中offset 和 count都必须大于等于0

3. 分页排序时，排序不要有二义性，二义性情况下可能会导致分页结果乱序，可以在后面追加一个主键排序

   > 排序过程中遇到相同的值,增加排序规则

## 分组

分组中select后面的列只能有2种

1. 出现在group by 后面的列
2. 使用聚合函数的列

可以通过子查询的方式避免这种限制

**数据准备**

~~~mysql
drop table if exists t_order;

-- 创建订单表
create table t_order(
  id int not null AUTO_INCREMENT COMMENT '订单id',
  user_id bigint not null comment '下单人id',
  user_name varchar(16) not null default '' comment '用户名',
  price decimal(10,2) not null default 0 comment '订单金额',
  the_year SMALLINT not null comment '订单创建年份',
  PRIMARY KEY (id)
) comment '订单表';

-- 插入数据
insert into t_order(user_id,user_name,price,the_year) values
  (1001,'路人甲Java',11.11,'2017'),
  (1001,'路人甲Java',22.22,'2018'),
  (1001,'路人甲Java',88.88,'2018'),
  (1002,'刘德华',33.33,'2018'),
  (1002,'刘德华',12.22,'2018'),
  (1002,'刘德华',16.66,'2018'),
  (1002,'刘德华',44.44,'2019'),
  (1003,'张学友',55.55,'2018'),
  (1003,'张学友',66.66,'2019');
~~~

 **需求：获取每个用户下单的最大金额及下单的年份，输出：用户id，最大金额，年份** 

~~~mysql

#解除mysql分组限制的原始写法,查询出的数据可能不准
SELECT
	user_id as 用户id,
	max( price ) as 最大金额,
	the_year as 年份 
FROM
	t_order t 
GROUP BY
	t.user_id;
# 子查询的写法,避免分组查询限制
SELECT
	user_id AS `用户id`,
	price AS 最大金额,
	the_year AS 年份 
FROM
	t_order t1 
WHERE
	( t1.user_id, t1.price ) IN ( SELECT t.user_id, MAX( t.price ) FROM t_order t GROUP BY t.user_id ) limit 1;
	
# 第二种写法
	SELECT
	user_id AS `用户id`,
	price AS 最大金额,
	the_year AS 年份 
FROM
	t_order t1,
	( SELECT t.user_id uid, MAX( t.price ) pc FROM t_order t GROUP BY t.user_id ) t2 
WHERE
	t1.user_id = t2.uid 
	AND t1.price = t2.pc 
	LIMIT 1;
~~~

