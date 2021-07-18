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

# MySQL 基本架构

## 基础说明

![](http://file.tzcqupt.top/MySQL-base.png)

MySQL 分为 Server 层和存储引擎层两部分

Server 层包括连接器、查询缓存、分析器、优化器、执行器等，涵盖 MySQL 的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

存储引擎层负责数据的存储和提取.

# Explain关键字

## 数据准备

[MySQL官网提供的影院数据库](https://downloads.mysql.com/docs/sakila-db.zip)

## 使用explain查询SQL的执行计划

~~~mysql
explain select * from film;
~~~

| id   | select\_type | table | partitions | type | possible\_keys | key  | key\_len | ref  | rows | filtered | Extra |
| :--- | :----------- | :---- | :--------- | :--- | :------------- | :--- | :------- | :--- | :--- | :------- | :---- |
| 1    | SIMPLE       | film  | NULL       | ALL  | NULL           | NULL | NULL     | NULL | 1000 | 100      | NULL  |

## 执行计划的字段解释和举例

### id

数字越大越先执行，如果数字一样大，那么就从上往下依次执行，id列为null就表示这是一个结果集，不需要使用它来进行查询。

### select_type

#### simple

表示不需要union操作或者不包含子查询的简单select查询，有连接查询时，外层的查询为simple，且只有一个。

~~~mysql
explain select * from film where film_id =2;
~~~

| id   | select\_type | table | partitions | type  | possible\_keys | key     | key\_len | ref   | rows | filtered | Extra |
| :--- | :----------- | :---- | :--------- | :---- | :------------- | :------ | :------- | :---- | :--- | :------- | :---- |
| 1    | SIMPLE       | film  | NULL       | const | PRIMARY        | PRIMARY | 2        | const | 1    | 100      | NULL  |

#### primary

一个需要union操作或者含有子查询的select，位于最外层的查询，select_type即为primary，且只有一个。

```mysql
explain select film_id from film
union all select film_id from film_actor;
```

| id   | select\_type | table       | partitions | type  | possible\_keys | key                   | key\_len | ref  | rows | filtered | Extra       |
| :--- | :----------- | :---------- | :--------- | :---- | :------------- | :-------------------- | :------- | :--- | :--- | :------- | :---------- |
| 1    | PRIMARY      | film        | NULL       | index | NULL           | idx\_fk\_language\_id | 1        | NULL | 1000 | 100      | Using index |
| 2    | UNION        | film\_actor | NULL       | index | NULL           | idx\_fk\_film\_id     | 2        | NULL | 5462 | 100      | Using index |

#### union

union连接的两个select查询，第一个查询是dervied派生表，除了第一个表外，第二个以后的表select_type都是union。

#### union result

包含union的结果集，在union和union all语句中,因为它不需要参与查询，所以id字段为null。

#### dependent union

与union一样，出现在union 或union all语句中，但是这个查询要受到外部查询的影响。

~~~mysql
explain select * from film_category where film_id
in (select film_id from film union all select film_id from film_actor);
~~~



| id   | select\_type       | table          | partitions | type    | possible\_keys    | key               | key\_len | ref  | rows | filtered | Extra       |
| :--- | :----------------- | :------------- | :--------- | :------ | :---------------- | :---------------- | :------- | :--- | :--- | :------- | :---------- |
| 1    | PRIMARY            | film\_category | NULL       | ALL     | NULL              | NULL              | NULL     | NULL | 1000 | 100      | Using where |
| 2    | DEPENDENT SUBQUERY | film           | NULL       | eq\_ref | PRIMARY           | PRIMARY           | 2        | func | 1    | 100      | Using index |
| 3    | DEPENDENT UNION    | film\_actor    | NULL       | ref     | idx\_fk\_film\_id | idx\_fk\_film\_id | 2        | func | 1    | 100      | Using index |

#### subquery

除了from子句中包含的子查询外，其他地方出现的子查询都可能是subquery。

~~~mysql
explain select (select 1 from film);
~~~

| id   | select\_type | table | partitions | type  | possible\_keys | key                   | key\_len | ref  | rows | filtered | Extra          |
| :--- | :----------- | :---- | :--------- | :---- | :------------- | :-------------------- | :------- | :--- | :--- | :------- | :------------- |
| 1    | PRIMARY      | NULL  | NULL       | NULL  | NULL           | NULL                  | NULL     | NULL | NULL | NULL     | No tables used |
| 2    | SUBQUERY     | film  | NULL       | index | NULL           | idx\_fk\_language\_id | 1        | NULL | 1000 | 100      | Using index    |

#### dependent subquery

与dependent union类似，表示这个subquery的查询要受到外部表查询的影响。

#### derived

from子句中出现的子查询，也叫做派生表，其他数据库中可能叫做内联视图或嵌套select。

#### materialization 

物化通过将子查询结果作为一个临时表来加快查询执行速度，正常来说是常驻内存，下次查询会再次引用临时表。

### **table**

显示的查询表名，如果查询使用了别名，那么这里显示的是别名，如果不涉及对数据表的操作，那么这显示为null，如果显示为尖括号括起来的就表示这个是临时表，后边的N就是执行计划中的id，表示结果来自于这个查询产生。如果是尖括号括起来的，与类似，也是一个临时表，表示这个结果来自于union查询的id为M,N的结果集。

### type

> 依次性能从好到差：system，const，eq_ref，ref，fulltext，ref_or_null，unique_subquery，index_subquery，range，index_merge，index，ALL，除了all之外，其他的type都可以使用到索引，除了index_merge之外，其他的type只可以用到一个索引。一般来说，好的sql查询至少达到range级别，最好能达到ref。

#### system

表中只有**一行数据**或者是空表，且只能用于myisam和memory表，如果是Innodb引擎表，type列在这个情况通常都是all或者index。

#### const

使用唯一索引或者主键，返回记录一定是**1行记录**的等值where条件时，通常type是const，其他数据库也叫做唯一索引扫描。

#### eq_ref

出现在要连接多个表的查询计划中，驱动表循环获取数据，这行数据是第二个表的主键或者唯一索引，作为条件查询只返回一条数据，且必须为not null，唯一索引和主键是多列时，只有所有的列都用作比较时才会出现eq_ref。

#### ref

不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现，常见与辅助索引的等值查找或者多列主键、唯一索引中，使用第一个列之外的列作为等值查找也会出现，总之，返回数据不唯一的等值查找就可能出现。

#### fulltext

全文索引检索，全文索引的优先级很高，若全文索引和普通索引同时存在时，mysql不管代价，优先选择使用全文索引。

#### ref_or_null

与ref方法类似，只是增加了null值的比较，实际用的不多。

#### unique_subquery

用于where中的in形式子查询，子查询返回不重复值唯一值。

#### index_subquery

用于in形式子查询使用到了辅助索引或者in常数列表，子查询可能返回重复值，可以使用索引将子查询去重。

#### range

索引范围扫描，常见于使用>,<,is null,between ,in ,like等运算符的查询中。

#### index_merge

表示查询使用了两个以上的索引，最后取交集或者并集，常见and ，or的条件使用了不同的索引，官方排序这个在ref_or_null之后，但是实际上由于要读取多个索引，性能可能大部分时间都不如range。

#### index

索引全表扫描，把索引从头到尾扫一遍，常见于使用索引列就可以处理不需要读取数据文件的查询、可以使用索引排序或者分组的查询。

#### all

这个就是全表扫描数据文件，然后再在server层进行过滤返回符合要求的记录。

### **possible_keys**

查询可能使用到的索引。

### key

查询真正使用到的索引，type为index_merge时，这里可能出现两个以上的索引，其他的select_type这里只会出现一个。

### **key_len**

用于处理查询的索引长度，如果是单列索引，那就是整个索引长度，如果是多列索引，那么查询不一定都能使用到所有的列，具体使用到了多少个列的索引，这里就会计算进去，没有使用到的列不会计算进去。留意下这个列的值，算一下你的多列索引总长度就知道有没有使用到所有的列了。另外，key_len只计算where条件用到的索引长度，而排序和分组就算用到了索引，也不会计算到key_len中。

### ref

如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

### rows

这里是执行计划中估算的扫描行数，不是精确值。

### **extra**

#### no tables used

不带from字句的查询或者From dual查询。

#### NULL

查询的列未被索引覆盖，并且where筛选条件是索引的前导列，意味着用到了索引，但是部分字段未被索引覆盖，必须通过“回表”来实现，不是纯粹地用到了索引，也不是完全没用到索引。

#### using index

查询时不需要回表查询，直接通过索引就可以获取查询的数据。

#### Using where

查询的列未被索引覆盖，where筛选条件非索引的前导列。

#### Using where Using index

查询的列被索引覆盖，并且where筛选条件是索引列之一但是不是索引的前导列，意味着无法直接通过索引查找来查询到符合条件的数据。(常见复合索引的后列 如index(id,name) 查询的条件为name)

#### Using index condition

与Using where类似，查询的列不完全被索引覆盖，where条件中是一个前导列的范围。

#### using temporary

表示使用了临时表存储中间结果。临时表可以是内存临时表和磁盘临时表，执行计划中看不出来，需要查看status变量，used_tmp_table，used_tmp_disk_table才能看出来。

#### using filesort

mysql 会对结果使用一个外部索引排序，而不是按索引次序从表里读取行。此时mysql会根据联接类型浏览所有符合条件的记录，并保存排序关键字和行指针，然后排序关键字并按顺序检索行信息。这种情况下一般也是要考虑使用索引来优化的。

#### using intersect

表示使用and的各个索引的条件时，该信息表示是从处理结果获取交集。

#### using union

表示使用or连接各个使用索引的条件时，该信息表示从处理结果获取并集。

#### using sort_union/using sort_intersection

用and和or查询信息量大时，先查询主键，然后进行排序合并后返回结果集。

#### firstmatch(tb_name)

5.6.x开始引入的优化子查询的新特性之一，常见于where字句含有in()类型的子查询。如果内表的数据量比较大，就可能出现这个。

#### loosescan(m..n)

5.6.x之后引入的优化子查询的新特性之一，在in()类型的子查询中，子查询返回的可能有重复记录时，就可能出现这个。

### filtered

使用explain extended时会出现这个列，5.7之后的版本默认就有这个字段，不需要使用explain extended了。这个字段表示存储引擎返回的数据在server层过滤后，剩下多少满足查询的记录数量的比例，注意是百分比，不是具体记录数。

# MySQL优化

## 慢查询的总体优化思路

### 思路

- 优化更需要优化的SQL
- 定位优化对象的性能瓶颈
- 明确的优化目标
- 从explain执行计划入手

### 实践

#### 永远用小结果集驱动大的结果集 (小表 left join 大表 )



- 尽可能在索引中完成排序(order by 字段最好加了索引)
- 只取出自己需要的列，不要用select *
- 仅使用最有效的过滤条件
- 尽可能避免复杂的join和子查询
- 小心使用order by，group by，distinct语句(尽量走索引)
- 合理设计并利用索引

## 单表查询优化

1. 明确需要的字段,而不是```select *```
2. 使用连接JOIN 代替子查询
3. 使用分页语句(limit [offset,] rows) 或者where 字句时,有什么限制条件就尽量加上,查一条就limit 1

## 多表查询优化

多表查询包括 `INNER JOIN`/`LEFT JOIN`/`RIGHT JOIN`

`cross join(交叉连接)`==> 两个表的笛卡尔积

### JOIN 实现原理

MySQL 中的JOIN算法==>Nested Loop Join,就是通过驱动表的结果集作为循环基础数据,然后一条一条的通过该结果集中的数据作为过滤条件到下一个表中查询数据,然后合并结果.

### 优化结论

1. 内连接加上`ON`限定条件,否则会被解释为交叉连接

2. 连接表格使用` , `会被解释为交叉连接

3. 超大型数据尽可能不写子查询,使用JOIN替换

   1. **当查询出来的数据需要进一步处理时,采用子查询,可读性和效率都更好**
   2. 特定的场景,如直接从数据库中读取,采用`JOIN`比`WHERE` 快

4. 使用`UNION`来代替手动创建的临时表

   1. `UNION`就是把两次或多次查询结果合并起来
   2. `UNION`会去掉重复的行,而`UNION ALL`不会
   3. 子句中有`ORDER BY/LIMIT`需要用括号`()`包起来,推荐放到所有字句后,对最终合并的结果来排序或筛选

5. 尽量避免在 `WHERE`子句中对字段进行`NULL`值判断,否则将导致引擎放弃使用索引而进行全表扫描.

6. `IN`和`NOT IN`也要慎用,否则导致全盘扫描

   1. 对于连续的数值,采用`BETWEEN AND`

   2. 采用 `EXISTS`代替`IN`

      ~~~mysql
      select id from t where num in(1,2,3)
      #====>
      select id from t where num between 1 and 3
      
      select num from a where num in(select num from b)
      #====>
      select num from a where exists(select 1 from b where num=a.num)
      ~~~

7. **尽量使用数字型字段**,若只包含数值信息的字段尽量不要设计为字符型,这会降低查询和连接的性能,增加内存开销. 是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了 

8. 尽量使用表变量来代替临时表
9. 索引才能将MySQL的查询优化得更好

## 慢查询优化

### 开启慢查询日志

~~~sql
show variables like 'slow_query_log'  
--查看是否开启慢查询日志
-- 查看慢查询的日志位置
show variables like '%log%';
set global slow_query_log_file='/var/lib/mysql/VM_0_17_centos-slow.log';
--慢查询日志的位置

set global slow_query_log =on;
--开启慢查询日志

set global long_query_time=1;  
--大于1秒钟的数据记录到慢日志中，如果设置为默认0，则会有大量的信息存储在磁盘中，磁盘很容易满掉
~~~

### 分析日志

1. MySQL提供的 **mysqldumpslow**

    ~~~mysql
    #slow记录最多的10个语句
    mysqldumpslow -s r -t 10 /VM_0_17_centos-slow.log 
    #按照时间排序前10中含有 left join的
    mysqldumpslow -s r -t 10 -g "left join" /VM_0_17_centos-slow.log 
    ~~~

2. **mysqlsla**工具

    mysqlsla会自动判断日志类型,建立一个配置文件`.mysqlsla`在文件里写上`top=100`,这样会打印出前100条结果.参数说明:

   ~~~
   queries total: 总查询次数 unique:去重后的sql数量
   sorted by : 输出报表的内容排序
   
   最重大的慢sql统计信息, 包括 平均执行时间, 等待锁时间, 结果行的总数, 扫描的行总数.
   
   Count, sql的执行次数及占总的slow log数量的百分比.
   Time, 执行时间, 包括总时间, 平均时间, 最小, 最大时间, 时间占到总慢sql时间的百分比.
   95% of Time, 去除最快和最慢的sql, 覆盖率占95%的sql的执行时间.
   Lock Time, 等待锁的时间.
   95% of Lock , 95%的慢sql等待锁时间.
   Rows sent, 结果行统计数量, 包括平均, 最小, 最大数量.
   Rows examined, 扫描的行数量.
   Database, 属于哪个数据库
   Users, 哪个用户,IP, 占到所有用户执行的sql百分比
   Query abstract, 抽象后的sql语句
   Query sample, sql语句
   ~~~

   

## 其他优化原则

1. 不在数据库做运算,运算迁移至业务层

2. 数据库命名简洁明确(长度不超过30个字符)

3. 控制列数量(字段少而精,字段数建议在20以内)

4. 平衡范式与冗余(效率优先,往往牺牲范式)

5. 拒绝3B(big sql语句,big transaction,big batch)

6. 用好数值类型(用合适的字段类型节约空间)

7. 字符转化为数字(节约空间,提高查询性能)

8. 避免使用NULL 字段(NULL 字段很难查询优化,NULL 字段的索引需要额外空间,NULL 字段的复合索引无效)

9. 少用text类型(尽量使用varchar类型代替)

10. 合理使用索引(改善查询,减慢更新,索引一定不是越多越好)

11. 字符字段建前缀索引

12. 不在索引做列运算

13. innodb引擎 主键推荐使用自增列(主键建立聚簇索引,主键不应该被修改,字符串不应该做主键);不用外键,由程序保证约束

14. 不适用select *

15. OR 改写为 IN(字段没有索引的情况下,性能差别较大)

16. OR 改写为 union(索引无效变有效)

17. 使用UNION ALL 代替UNION( UNION 有去重开销)

18. 分页limit 优化(偏移量越大,执行越慢)

    

## 索引失效场景

1. 符合索引尽量全匹配
2. 最佳左前缀法则(带头索引不能死,中间索引不能断)
3. 不要在索引上左任何操作(计算,函数,自动/手动类型转换),否则会导致索引失效而转向全表扫描
4. mysql存储引擎不能继续使用索引中范围条件(between,< ,>,in)
5. 尽量使用覆盖索引(子查询索引的列==>索引列和查询列一致),减少 select *
6. 索引字段上使用(!= 或者 <>)判断时,会导致索引失效而转向全表扫描
7. 索引字段上使用(is null /is not null)判断时,会导致索引失效而转向全表扫描(**mysql 8** 时索引不会失效)
8. 索引字段使用 like 以通配符开头('% 字符串')时,会导致索引失效而转向全表扫描
9. 索引字段是字符串,但查询时不加单引号','会导致索引失效而转向全表扫描
10. 索引字段上使用(or)判断时,会导致索引失效而转向全表扫描



# MySQL 索引

## 索引的基本原理

 通过不断地缩小想要获取数据的范围来筛选出最终想要的结果，同时把随机的事件变成顺序的事件 ===>**把无序的数据变成有序的数据**

1. 把创建了索引的列的内容进行排序

2. 对排序结果生成倒排表

3. 在倒排表内容上拼上数据地址链

4. 在查询的时候，先拿到倒排表内容，再取出数据地址链，从而拿到具体数据

## 硬盘中数据的读取

### 硬盘中的概念

以机械硬盘为例.

#### 扇区

 磁盘存储的最小单位，扇区一般大小为512Byte。 

#### 硬盘块

 文件系统与磁盘交互的的最小单位（计算机系统读写磁盘的最小单位），一个磁盘块由连续几个（2^n）扇区组成，块一般大小一般为4KB。 

#### 硬盘读取数据

 磁盘读取数据靠的是机械运动，每次读取数据花费的时间可以分为**寻道时间、旋转延迟、传输时间**三个部分 .

##### 寻道时间 

 磁臂移动到指定磁道所需要的时间，主流磁盘一般在5ms以下 

##### 旋转延迟

 旋转延迟就是我们经常听说的磁盘转速，比如一个磁盘7200转，表示每分钟能转7200次，也就是说1秒钟能转120次，旋转延迟就是1/120/2 = 4.17ms 

##### 传输时间

 传输时间指的是从磁盘读出或将数据写入磁盘的时间，一般在零点几毫秒，相对于前两个时间可以忽略不计 

## InnoDB 中的索引

### 索引解释

**InnoDB中有2种索引：**主键索引（聚集索引）、辅助索引（非聚集索引）。

**主键索引：**每个表只有一个主键索引，叶子节点同时保存了主键的值也数据记录。

**辅助索引：**叶子节点保存了索引字段的值以及主键的值。

辅助索引的查询过程在MySQL中叫做回表.

### 创建索引的原则

1. 最左前缀匹配原则，组合索引非常重要的原则，MySQL会一直向右匹配直到遇到范围查询(`>、<、between、like`)就停止匹配，比如`a = 1 and b = 2 and c > 3 and d = 4` 如果建立`(a,b,c,d)`顺序的索引，d是用不到索引的，如果建立`(a,b,d,c)`的索引则都可以用到，a,b,d的顺序可以任意调整。 
2.  较频繁作为查询条件的字段才去创建索引 
3.  更新频繁字段不适合创建索引 
4.  尽量的扩展索引，不要新建索引。比如表中已经有a的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。 
5.  定义有外键的数据列一定要建立索引。
6.  对于那些查询中很少涉及的列，重复值比较多的列不要建立索引。 
7.   对于定义为text、image和bit的数据类型的列不要建立索引。 

## MySQL中的页

###  解释

MySQL中和磁盘交互的最小单位称为页，页是MySQL内部定义的一种数据结构，默认为16kb，相当于4个磁盘块，也就是说MySQL每次从磁盘中读取一次数据是16KB，要么不读取，要读取就是16KB，此值可以修改的。 

### 页结构

 MySQL中页是InnoDB 中存储数据的基本单位，也是MySQL中管理数据的最小单位，和磁盘交互的时候都是以页来进行的，默认是16kb，MySQL中采用b+树存储数据，页相当于b+树中的一个节点。 