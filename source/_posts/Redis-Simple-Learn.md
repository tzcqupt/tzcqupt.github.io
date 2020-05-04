---
layout: post
title: Redis入门学习笔记
Author: tzcqupt
tags: [Redis]
categories: [数据库]
comments: true
toc: true
date: 2020-04-27 00:00:00
---
# 介绍

 Redis是一款高性能的NOSQL系列的非关系型数据库。

# 什么是NOSQL  

NoSQL(NoSQL = Not Only SQL),意即“不仅仅是SQL”,是一项全新的数据库理念,泛指非关系型的数据库。

## NOSQL和关系型数据库比较  

### 优点

1. 成本:nosql数据库简单易部署,基本都是开源软件,不需要像使用oracle那样花费大量成本购买使用,相比关系型数据库价格便宜。
2. 查询速度:nosql数据库将数据存储于缓存之中,关系型数据库将数据存储在硬盘中,自然查询速度远不及nosql数据库。
3. 存储数据的格式:nosql的存储格式是key,value形式、文档形式、图片形式等等,所以可以存储基础类型以及对象或者是集合等各种格式,而数据库则只支持基础类型。
4. 扩展性:关系型数据库有类似join这样的多表查询机制的限制导致扩展很艰难。  

### 缺点

1. 维护的工具和资料有限,因为nosql是属于新的技术,不能和关系型数据库10几年的技术同日而语。
2. 不提供对sql的支持,如果不支持sql这样的工业标准,将产生一定用户的学习和使用成本。
3. 不提供关系型数据库对事务的处理。

## 非关系型数据库的优势

1. 性能NOSQL是基于键值对的,可以想象成表中的主键和值的对应关系,而且不需要经过SQL层的解析,所以性能非常高。
2. 可扩展性同样也是因为基于键值对,数据之间没有耦合性,所以非常容易水平扩展。

## 关系型数据库的优势

1. 复杂查询可以用SQL语句方便的在一个表以及多个表之间做非常复杂的数据查询。
2. 事务支持使得对于安全性能很高的数据访问要求得以实现。对于这两类数据库,对方的优势就是自己的弱势,反之亦然。

## 总结  

关系型数据库与NoSQL数据库并非对立而是互补的关系,即通常情况下使用关系型数据库,在适合使用NoSQL的时候使用NoSQL数据库,让NoSQL数据库对关系型数据库的不足进行弥补。  
一般会将数据存储在关系型数据库中,在nosql数据库中备份存储关系型数据库的数据

# 主流的NOSQL产品

## 键值(Key-Value)存储数据库

- 相关产品: Tokyo Cabinet/Tyrant、Redis、Voldemort、Berkeley DB
- 典型应用: 内容缓存,主要用于处理大量数据的高访问负载。 
- 数据模型: 一系列键值对
- 优势: 快速查询
- 劣势: 存储的数据缺少结构化

## 列存储数据库

- 相关产品:Cassandra, HBase, Riak
- 典型应用:分布式的文件系统
- 数据模型:以列簇式存储,将同一列数据存在一起
- 优势:查找速度快,可扩展性强,更容易进行分布式扩展
- 劣势:功能相对局限

## 文档型数据库

- 相关产品:CouchDB、MongoDB
- 典型应用:Web应用(与Key-Value类似,Value是结构化的)
- 数据模型: 一系列键值对
- 优势:数据结构要求不严格
- 劣势: 查询性能不高,而且缺乏统一的查询语法

## 图形(Graph)数据库

- 相关数据库:Neo4J、InfoGrid、Infinite Graph
- 典型应用:社交网络
- 数据模型:图结构
- 优势:利用图结构相关算法。
- 劣势:需要对整个图做计算才能得出结果,不容易做分布式的集群方案。

# 安装redis

## centos7配置redis开机启动

1. 执行`vim /etc/systemd/system/redis.service`,写入下面代码

    ~~~bash
        [Unit]
    Description=redis-server
        After=network.target

        [Service]
        Type=forking
    ExecStart=/usr/local/bin/redis-server /usr/local/bin/redis.conf
        PrivateTmp=true

        [Install]
        WantedBy=multi-user.target
    ~~~

2. 测试命令

   - 先关闭redis-server     
    `systemctl stop redis.service`  
   - 开启redis-server     
     如果服务是开启状态,使用此命令会启动失败。  
    `systemctl start redis.service `

3. 设置开机启动  
    `systemctl enable redis.service`

   >cat /etc/centos-release查看centos版本

## Windows设置redis开机启动

1. 修改redis.windows.conf配置文件,限制内存,防止偶尔闪退,加上`maxmemory 209715200`
2. 在安装redis的路径下执行`redis-server --service-install redis.windows.conf`
3. redis-server --service-start启动redis
4. 在计算机的管理==>服务里面查看redis设置为自动启动

## 开放外网访问

打开`redis.conf`文件在`NETWORK`部分修改 注释掉`bind 127.0.0.1`可以使所有的ip访问redis

修改 `protected-mode`，值改为`no`

# Redis详细介绍

Redis是用C语言开发的一个开源的高性能键值对(key-value)数据库,Redis通过提供多种键值数据类型来适应不同场景下的存储需求,目前为止Redis支持的键值数据类型如下:

1. 字符串类型 `string`==>`Map<String,String>`
2. 哈希类型 `hash`==>`Map<String,Map<String,String>>`
3. 列表类型 `list`==>`Map<String,List<String>>`
4. 集合类型 `set`==>`Map<String,Set<String>>`
5. 有序集合类型 `sortedset`==>可排序的`set`

# Redis的应用场景

- 缓存(数据查询、短连接、新闻内容、商品内容等等)
- 聊天室的在线好友列表
- 任务队列。(秒杀、抢购、12306等等)
- 应用排行榜
- 网站访问统计
- 数据过期处理(可以精确到毫秒)
- 分布式集群架构中的`session`分离		

# Redis操作

## Redis的数据结构  

Redis存储的是:key,value格式的数据,其中key都是字符串,value有5种不同的数据结构

1. 字符串类型 `string`
2. 哈希类型 `hash` : `Map`格式  
3. 列表类型 `list` : `Linkedlist`格式。支持重复元素
4. 集合类型 `set`  : 不允许重复元素
5. 有序集合类型 `sortedset`:不允许重复元素,且元素有顺序	

### 字符串类型 `string`

1. 存储  
~~~bash
set key value 
set username zhangsan
~~~
2. 获取
~~~bash
get key
get username  
"zhangsan"
~~~
3. 删除
~~~bash
del key
del age
(integer) 1
~~~

### 哈希类型 `hash`

1. 存储 
    ~~~bash
    hset key field value
    hset myhash username lisi 
      	(integer) 1  
    hset myhash password 123 
      	(integer) 1
    ~~~
2. 获取
   - `hget key field`: 获取指定的`field`对应的值  
     `hget myhash username`  
     `"lisi"`
   - `hgetall key`:获取所有的`field`和`value`  
     `hgetall myhash`
	
3. 删除 

	~~~bash
	   hdel key field   
	   hdel myhash username 
	   (integer) 1
	~~~


### 列表类型 `list`

可以添加一个元素到列表的头部(左边)或者尾部(右边)

1. 添加

   1. `lpush key value`: 将元素加入列表左表		

   2. `rpush key value`:将元素加入列表右边	

      ~~~sql
      	lpush myList a
      	(integer) 1
      	lpush myList b
      	(integer) 2
      	 rpush myList c
      	(integer) 3
      ~~~

2. 获取

  ~~~bash
  lrange key start end :范围获取  
  lrange myList 0 -1 :表示获取所有的
  ~~~

3. 删除

   - `lpop key`: 删除列表最左边的元素,并将元素返回
   - `rpop key`: 删除列表最右边的元素,并将元素返回

### 集合类型 `set`

 不允许重复元素

1. 存储:`sadd key value`

   ~~~sql	
   127.0.0.1:6379> sadd myset a
   (integer) 1
   127.0.0.1:6379> sadd myset a
   (integer) 0
   ~~~

2. 获取:`smembers key`:获取`set`集合中所有元素

   ~~~sql
   	127.0.0.1:6379> smembers myset
   	1) "a"
   ~~~

3. 删除:`srem key value`:删除`set`集合中的某个元素	

   ~~~sql	
   	127.0.0.1:6379> srem myset a
   	(integer) 1
   ~~~

### 有序集合类型 `sortedset`

不允许重复元素,且元素有顺序.每个元素都会关联一个double类型的分数。Redis正是通过分数来为集合中的成员进行从小到大的排序。

1. 存储:`zadd key score value`

   ~~~sql	
   	127.0.0.1:6379> zadd mysort 60 zhangsan
   	(integer) 1
   	127.0.0.1:6379> zadd mysort 500 lisi
   	(integer) 1
   	127.0.0.1:6379> zadd mysort 80 wangwu
   	(integer) 1
   ~~~

2. 获取:`zrange key start end [withscores]`

   ~~~sql	
   	127.0.0.1:6379> zrange mysort 0 -1
   	1) "lisi"
   	2) "zhangsan"
   	3) "wangwu"
   
   	127.0.0.1:6379> zrange mysort 0 -1 withscores
   	1) "zhangsan"
   	2) "60"
   	3) "wangwu"
   	4) "80"
   	5) "lisi"
   	6) "500"
   ~~~

3. 删除:`zrem key value`

   ~~~sql
   	127.0.0.1:6379> zrem mysort lisi
   	(integer) 1
   ~~~

## 通用命令

1. `keys * `: 查询所有的键
2. `type key` : 获取键对应的value的类型
3. `del key`:删除指定的key value

## 持久化

1. Redis是一个内存数据库,当Redis服务器重启,获取电脑重启,数据会丢失,我们可以将Redis内存中的数据持久化保存到硬盘的文件中。

2. Redis持久化机制:

   1. `RDB`:默认方式,不需要进行配置,默认就使用这种机制  
      在一定的间隔时间中,检测key的变化情况,然后持久化数据

      1. 编辑`Redis.windwos.conf`文件

         ~~~bash
         	#   after 900 sec (15 min) if at least 1 key changed
         	save 900 1
         	#   after 300 sec (5 min) if at least 10 keys changed
         	save 300 10
         	#   after 60 sec if at least 10000 keys changed
         	save 60 10000					
         ~~~

      2. 重新启动Redis服务器,并指定配置文件名称
         `D:\JavaWeb2019\Redis\windows-64\Redis-2.8.9>Redis-server.exe Redis.windows.conf`				

   2. AOF:日志记录的方式,可以记录每一条命令的操作。可以每一次命令操作后,持久化数据  
      编辑`Redis.windwos.conf`文件  

      ~~~bash
      	appendonly no(关闭aof) --> appendonly yes (开启aof)	
      	# appendfsync always : 每一次操作都进行持久化
      	appendfsync everysec : 每隔一秒进行一次持久化
      	# appendfsync no	 : 不进行持久化
      ~~~

## Java客户端 Jedis

 一款java操作Redis数据库的工具.

### 使用步骤

1. 下载jedis的jar包
2. 使用

~~~java
	//1. 获取连接
   	Jedis jedis = new Jedis("localhost",6379);
  	//2. 操作
    jedis.set("username","zhangsan");
   	//3. 关闭连接
   	jedis.close();
~~~

### Jedis操作各种Redis中的数据结构

#### 字符串类型 `string`

~~~java
 	//set
	//get
	
	 //1. 获取连接
     Jedis jedis = new Jedis();//如果使用空参构造,默认值 "localhost",6379端口
     //2. 操作
     //存储
     jedis.set("username","zhangsan");
     //获取
     String username = jedis.get("username");
     System.out.println(username);
     //可以使用setex()方法存储可以指定过期时间的 key value
     jedis.setex("activecode",20,"hehe");//将activecode:hehe键值对存入Redis,并且20秒后自动删除该键值对
     //3. 关闭连接
     jedis.close();
~~~

#### 哈希类型 `hash` : `Map`

~~~java
	//hset
	//hget
	//hgetAll
	//1. 获取连接
     Jedis jedis = new Jedis();//如果使用空参构造,默认值 "localhost",6379端口
     //2. 操作
     // 存储hash
     jedis.hset("user","name","lisi");
     jedis.hset("user","age","23");
     jedis.hset("user","gender","female");

     // 获取hash
     String name = jedis.hget("user", "name");
     System.out.println(name);
 	  // 获取hash的所有map中的数据
 	 Map<String, String> user = jedis.hgetAll("user");

     // keyset
     Set<String> keySet = user.keySet();
     for (String key : keySet) {
         //获取value
         String value = user.get(key);
         System.out.println(key + ":" + value);
     }

     //3. 关闭连接
     jedis.close();
~~~

#### 列表类型 `list` : `LinkedList`

支持重复元素

  ~~~ java
  	//lpush / rpush
  	//lpop / rpop
  	//lrange start end : 范围获取
  	
  	 //1. 获取连接
       Jedis jedis = new Jedis();//如果使用空参构造,默认值 "localhost",6379端口
       //2. 操作
       // list 存储
       jedis.lpush("mylist","a","b","c");//从左边存
       jedis.rpush("mylist","a","b","c");//从右边存
  
       // list 范围获取
       List<String> mylist = jedis.lrange("mylist", 0, -1);
       System.out.println(mylist);
       
       // list 弹出
       String element1 = jedis.lpop("mylist");//c
       System.out.println(element1);
  
       String element2 = jedis.rpop("mylist");//c
       System.out.println(element2);
  
       // list 范围获取
       List<String> mylist2 = jedis.lrange("mylist", 0, -1);
       System.out.println(mylist2);
  
       //3. 关闭连接
       jedis.close();
  ~~~

#### 集合类型 `set`  

不允许重复元素

~~~	java
	//sadd
	//smembers:获取所有元素

	//1. 获取连接
     Jedis jedis = new Jedis();//如果使用空参构造,默认值 "localhost",6379端口
     //2. 操作
        // set 存储
    jedis.sadd("myset","java","php","c++");
    
      // set 获取
    Set<String> myset = jedis.smembers("myset");
    System.out.println(myset);
    
      //3. 关闭连接
    jedis.close();
~~~

#### 有序集合类型 sortedset

不允许重复元素,且元素有顺序

  ~~~java
  	//zadd
  	//zrange
  
  	//1. 获取连接
       Jedis jedis = new Jedis();//如果使用空参构造,默认值 "localhost",6379端口
       //2. 操作
       // sortedset 存储
       jedis.zadd("mysortedset",3,"亚瑟");
       jedis.zadd("mysortedset",30,"后裔");
       jedis.zadd("mysortedset",55,"孙悟空");
  
       // sortedset 获取
       Set<String> mysortedset = jedis.zrange("mysortedset", 0, -1);
  
       System.out.println(mysortedset);
  
       //3. 关闭连接
       jedis.close();
  ~~~

### jedis连接池: `JedisPool`

#### 使用

1. 创建JedisPool连接池对象

2. 调用方法 getResource()方法获取Jedis连接

   ~~~java
   //0.创建一个配置对象
   JedisPoolConfig config = new JedisPoolConfig();
   config.setMaxTotal(50);
   config.setMaxIdle(10);
   
   //1.创建Jedis连接池对象
   JedisPool jedisPool = new JedisPool(config,"localhost",6379);
   
   //2.获取连接
   Jedis jedis = jedisPool.getResource();
   //3. 使用
   jedis.set("hehe","heihei");
    //4. 关闭 归还到连接池中
    jedis.close();
   ~~~
   
#### 连接池工具类

  ~~~java
      public class JedisPoolUtils {

              private static JedisPool jedisPool;

              static{
                  //读取配置文件
                  InputStream is = JedisPoolUtils.class.getClassLoader().getResourceAsStream("jedis.properties");
                  //创建Properties对象
                  Properties pro = new Properties();
                  //关联文件
                  try {
                      pro.load(is);
                  } catch (IOException e) {
                      e.printStackTrace();
                  }
                  //获取数据,设置到JedisPoolConfig中
                  JedisPoolConfig config = new JedisPoolConfig();
                  config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
                  config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));

                  //初始化JedisPool
                  jedisPool = new JedisPool(config,pro.getProperty("host"),Integer.parseInt(pro.getProperty("port")));
              }

              /**
               * 获取连接方法
               */
              public static Jedis getJedis(){
                  return jedisPool.getResource();
              }
          }
  ~~~

<span id=spring-data-Redis></span>

# Redis整合

## Spring整合Redis

### 普通整合

#### 版本依赖

~~~xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>4.3.8.RELEASE</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.8.3.RELEASE</version>
</dependency>
~~~

#### 配置文件

~~~xml
<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <!--最大空闲数-->
    <property name="maxIdle" value="50"/>
    <!--最大连接数-->
    <property name="maxTotal" value="100"/>
    <!--最大等待时间-->
    <property name="maxWaitMillis" value="20000"/>
</bean>
<!--引入配置文件-->
<context:property-placeholder location="redis.properties"/>
<!--配置spring连接工厂-->
<bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <property name="hostName" value="${redis.host}"/>
    <property name="port" value="${redis.port}"/>
    <property name="poolConfig" ref="poolConfig"/>
</bean>
<!--配置 Spring RedisTemplate -->
<bean id="jdkSerializationRedisSerializer"
      class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer">
</bean>
<bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer">
</bean>
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="keySerializer" ref="stringRedisSerializer"/>
    <property name="valueSerializer" ref="jdkSerializationRedisSerializer"/>
</bean>
~~~

#### 使用方式

~~~java
@Test
public void testSaveSameConnection() {
    Role role = new Role();
    role.setId(1L);
    role.setRoleName("redis-role-name-1");
    role.setNote("redis-role-note-1");
    //保证存和取来自同一个连接
    SessionCallback callback = new SessionCallback() {
        @Override
        public Object execute(RedisOperations redisOperations) throws DataAccessException {
            redisOperations.boundValueOps("ssm:redis:test:role-1").set(role);
            return redisOperations.boundValueOps("ssm:redis:test:role-1").get();
        }
    };
    Role savedRole = (Role) redisTemplate.execute(callback);
    log.info(savedRole.toString());
}
~~~

### Redis的发布订阅模式

#### xml配置

~~~xml
<bean id="redisTemplateValueWithString" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="connectionFactory"/>
    <property name="defaultSerializer" ref="stringRedisSerializer"/>
    <property name="keySerializer" ref="stringRedisSerializer"/>
    <!--测试发布订阅模式时，需要用string序列化器-->
    <property name="valueSerializer" ref="stringRedisSerializer"/>
</bean>
<!--注册接受消息的类-->
<bean id="redisMsgListener" class="com.tang.ssm.redis.listener.RedisMessageListener">
    <property name="redisTemplate" ref="redisTemplateValueWithString"/>
</bean>
<!--监听容器-->
<bean id="topicContainer" class="org.springframework.data.redis.listener.RedisMessageListenerContainer"
      destroy-method="destroy">
    <property name="connectionFactory" ref="connectionFactory"/>
    <!--连接池，需要线程池生存，才能继续监听-->
    <property name="taskExecutor">
        <bean class="org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler">
            <property name="poolSize" value="3"/>
        </bean>
    </property>
    <!--消息监听Map-->
    <property name="messageListeners">
        <map>
            <!--配置监听者-->
            <entry key-ref="redisMsgListener">
                <!--监听类-->
                <bean class="org.springframework.data.redis.listener.ChannelTopic">
                    <constructor-arg value="chat"/>
                </bean>
            </entry>
        </map>
    </property>
</bean>
~~~

#### 接受消息的类

~~~java
/**
 * redis的发布订阅模式
 * 接受消息的类
 * 该类在xml中注册，用的时set方法注册
 */
@Slf4j
public class RedisMessageListener implements MessageListener {

    private RedisTemplate redisTemplateValueWithString;

    public RedisTemplate getRedisTemplate() {
        return redisTemplateValueWithString;
    }

    public void setRedisTemplate(RedisTemplate redisTemplate) {
        this.redisTemplateValueWithString = redisTemplate;
    }

    @Override
    public void onMessage(Message message, byte[] bytes) {
        //接受消息
        byte[] body = message.getBody();
        //使用值序列化器转换
        try {
            String msgBody = (String) getRedisTemplate().getValueSerializer()
                    .deserialize(body);
            log.info(msgBody);
        } catch (Exception e) {
            log.error(e.toString());
        }
        //log.info(msgBody);
        //获取channel
        byte[] channel = message.getChannel();
        //使用字符串序列化器转换
        String channelStr = (String) getRedisTemplate().getStringSerializer()
                .deserialize(channel);
        log.info(channelStr);
        //渠道名称转换
        String byteStr = new String(bytes);
        log.info(byteStr);
    }
}
~~~

#### 测试类

~~~java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext_redis.xml"})
@Slf4j
public class RedisTemplateTest {
    /**
     * 涉及到发布订阅模式时，需要用到String序列化器
     */
    @Autowired
    @Qualifier("redisTemplateValueWithString")
    private RedisTemplate redisTemplateValueWithString;
    @Test
    public void testPublishTopic() {
        String channel = "chat";
        String message = "redis publish!!";
        redisTemplateValueWithString.convertAndSend(channel, message);
    }
}
~~~

### Spring整合redis缓存

#### 步骤

1. 整合Spring和MyBatis
2. 配置redisTemplate(连接池，序列化器配置，参考#普通整合)
3. 配置redis的缓存管理器，设置超时时间，reids缓存名字，方便service层绑定，启用spring缓存
4.  缓存注解解释
   1. **@Cacheable**定义缓存策略 当缓存中有值，则返回缓存数据，否则访问方法得到数据 通过value引用缓存管理器，通过key定义键
   2. **@CachePut**则表示无论如何都会执行方法，最后将方法的返回值再保存到缓存中， 使用在插入数据的地方，则表示保存到数据库后，会同期插入到Redis缓存中。更新数据时，也会同时更新缓存
   3. **@CacheEvict**删除缓存对应的key

#### Xml配置整合

##### application_redis.xml

~~~xml
<!--redis版xml-->
<!--省略连接池，redisTemplate配置-->

<!--启用缓存 -->
<cache:annotation-driven />
<!--注册redis缓存-->
<bean id="cacheManager" class="org.springframework.data.redis.cache.RedisCacheManager">
    <constructor-arg ref="redisTemplateValueWithJdk"/>
    <!--设置超时时间为10分钟，单位为秒-->
    <property name="defaultExpiration" value="60"/>
    <property name="cacheNames">
        <list>
            <value>redisCacheManager</value>
        </list>
    </property>
</bean>
~~~

##### application_jdbc.xml

~~~xml

<!--jdbc版xml-->
<!--启用扫描机制，并指定扫描对应的包-->
    <context:annotation-config/>
    <context:component-scan base-package="com.tang.ssm.redis.service"/>
    <!--从配置文件读取 数据库连接参数 -->
    <context:property-placeholder location="classpath:db.properties"/>
    <!--1.将连接池放入spring容器 -->
    <bean name="dataSource" class="com.zaxxer.hikari.HikariDataSource">
        <property name="jdbcUrl" value="${jdbc.jdbcUrl}"/>
        <property name="driverClassName" value="${jdbc.driverClass}"/>
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
        <!-- 等待连接池分配连接的最大时长（毫秒），超过这个时长还没可用的连接则发生SQLException， 缺省:30秒 -->
        <property name="connectionTimeout" value="${jdbc.connectionTimeout}"/>
        <!-- 一个连接idle状态的最大时长（毫秒），超时则被释放（retired），缺省:10分钟 -->
        <property name="idleTimeout" value="${jdbc.idleTimeout}"/>
        <!-- 一个连接的生命时长（毫秒），超时而且没被使用则被释放（retired），缺省:30分钟，建议设置比数据库超时时长少30秒，参考MySQL
            wait_timeout参数（show variables like '%timeout%';） -->
        <property name="maxLifetime" value="${jdbc.maxLifetime}"/>
        <!-- 连接池中允许的最大连接数。缺省值：10；推荐的公式：((core_count * 2) + effective_spindle_count) -->
        <property name="maximumPoolSize" value="${jdbc.maximumPoolSize}"/>
        <property name="minimumIdle" value="${jdbc.minimumIdle}"/>
    </bean>
    <!-- 集成MyBatis -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--指定MyBatis配置文件-->
        <property name="configLocation" value="classpath:/mybatis/mybatis-config.xml"/>
        <property name="typeAliasesPackage" value="com.tang.ssm.redis.pojo"/>
    </bean>

    <!-- 事务管理器配置数据源事务 -->
    <bean id="transactionManager"
          class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 使用注解定义事务 -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <!-- 采用自动扫描方式创建mapper bean -->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="com.tang.ssm.redis.dao"/>
        <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
        <property name="annotationClass" value="org.springframework.stereotype.Repository"/>
    </bean>
~~~

#### Java注解配置

##### RedisConfig

~~~JAVA
@Configuration
@EnableCaching
@ComponentScan("com.tang.ssm.redis")
public class RedisConfig {

    @Bean(name = "redisTemplate")
    public RedisTemplate initRedisTemplate() {
        JedisPoolConfig poolConfig = new JedisPoolConfig();
        // 最大空闲数
        poolConfig.setMaxIdle(50);
        // 最大连接数
        poolConfig.setMaxTotal(100);
        // 最大等待毫秒数
        poolConfig.setMaxWaitMillis(20000);
        // 创建Jedis连接工厂
        JedisConnectionFactory connectionFactory = new JedisConnectionFactory(poolConfig);
        connectionFactory.setHostName("localhost");
        connectionFactory.setPort(6379);
        // 调用后初始化方法，没有它将抛出异常
        connectionFactory.afterPropertiesSet();
        // 自定Redis序列化器
        RedisSerializer jdkSerializationRedisSerializer = new JdkSerializationRedisSerializer();
        RedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // 定义RedisTemplate，并设置连接工程
        RedisTemplate redisTemplate = new RedisTemplate();
        redisTemplate.setConnectionFactory(connectionFactory);
        // 设置序列化器
        redisTemplate.setDefaultSerializer(stringRedisSerializer);
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setValueSerializer(jdkSerializationRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        redisTemplate.setHashValueSerializer(jdkSerializationRedisSerializer);
        return redisTemplate;
    }

    @Bean(name = "redisCacheManager")
    public CacheManager initRedisCacheManager(@Autowired RedisTemplate redisTemplate) {
        RedisCacheManager cacheManager = new RedisCacheManager(redisTemplate);
        // 设置超时时间为10分钟，单位为秒
        cacheManager.setDefaultExpiration(600);
        // 设置缓存名称
        List<String> cacheNames = new ArrayList<>();
        cacheNames.add("redisCacheManager");
        cacheManager.setCacheNames(cacheNames);
        return cacheManager;
    }
}

~~~

##### JdbcConfig

~~~java
@Configuration
@ComponentScan("com.tang.ssm.redis")
@EnableTransactionManagement
public class JdbcConfig implements TransactionManagementConfigurer {

    private DataSource dataSource = null;

    /**
     * 创建一个数据源，并存入 spring 容器中
     */
    @Bean("dataSource")
    public DataSource initDataSource() {
        if (dataSource != null) {
            return dataSource;
        }
        try {
            HikariDataSource dataSource = new HikariDataSource();
            //todo 采取读取配置文件的方式一直失败
            dataSource.setDriverClassName("com.mysql.jdbc.Driver");
            dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/ssm_learn?useUnicode=true&useSSL=false&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai");
            dataSource.setUsername("root");
            dataSource.setPassword("root123");
            return dataSource;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    /**
     * * 配置SqlSessionFactoryBean
     *
     * @return SqlSessionFactoryBean
     */
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactoryBean initSqlSessionFactory() {
        SqlSessionFactoryBean sqlSessionFactory = new SqlSessionFactoryBean();
        sqlSessionFactory.setDataSource(initDataSource());
        // 配置MyBatis配置文件
        Resource resource = new ClassPathResource("mybatis/mybatis-config.xml");
        sqlSessionFactory.setConfigLocation(resource);
        sqlSessionFactory.setTypeAliasesPackage("com.tang.ssm.redis.pojo");
        return sqlSessionFactory;
    }

    /**
     * * 通过自动扫描，发现MyBatis Mapper接口
     *
     * @return Mapper扫描器
     */
    @Bean("mapperScannerConfigurer")
    public MapperScannerConfigurer initMapperScannerConfigurer() {
        MapperScannerConfigurer msc = new MapperScannerConfigurer();
        // 扫描包
        msc.setBasePackage("com.tang.ssm.redis.dao");
        msc.setSqlSessionFactoryBeanName("sqlSessionFactory");
        // 区分注解扫描
        msc.setAnnotationClass(Repository.class);
        return msc;
    }

    /**
     * 实现接口方法，注册注解事务，当@Transactional 使用的时候产生数据库事务
     */
    @Override
    @Bean(name = "annotationDrivenTransactionManager")
    public PlatformTransactionManager annotationDrivenTransactionManager() {
        DataSourceTransactionManager transactionManager = new DataSourceTransactionManager();
        transactionManager.setDataSource(initDataSource());
        return transactionManager;
    }
}

~~~

#### Serice层缓存注解配置

~~~java
@Service
public class RoleServiceImpl implements RoleService {
    @Autowired
    private RoleDao roleDao = null;

    /**
     * 使用@Cacheable定义缓存策略 当缓存中有值，则返回缓存数据，否则访问方法得到数据 通过value引用缓存管理器，通过key定义键
     *
     * @param id 角色编号
     * @return 角色
     */
    @Override
    @Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
    @Cacheable(value = "redisCacheManager", key = "'ssm:redis:redis_role_'+#id")
    public Role getRole(Long id) {
        return roleDao.getRole(id);
    }

    /**
     * 使用@CachePut则表示无论如何都会执行方法，最后将方法的返回值再保存到缓存中
     * 使用在插入数据的地方，则表示保存到数据库后，会同期插入到Redis缓存中
     *
     * @param role 角色对象
     * @return 角色对象（会回填主键）
     */
    @Override
    @Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
    @CachePut(value = "redisCacheManager", key = "'ssm:redis:redis_role_'+#result.id")
    public Role insertRole(Role role) {
        roleDao.insertRole(role);
        return role;
    }

    /**
     * 使用@CachePut，表示更新数据库数据的同时，也会同步更新缓存
     *
     * @param role 角色对象
     * @return 影响条数
     */
    @Override
    @Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
    @CachePut(value = "redisCacheManager", key = "'ssm:redis:redis_role_'+#role.id")
    public int updateRole(Role role) {
        return roleDao.updateRole(role);
    }

    /**
     * 使用@CacheEvict删除缓存对应的key
     *
     * @param id 角色编号
     * @return 返回删除记录数
     */
    @Override
    @Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
    @CacheEvict(value = "redisCacheManager", key = "'ssm:redis:redis_role_'+#id")
    public int deleteRole(Long id) {
        return roleDao.deleteRole(id);
    }

    @Override
    @Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
    public List<Role> findRoles(String roleName, String note) {
        return roleDao.findRoles(roleName, note);
    }

    @Override
    @Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
    public int insertRoles(List<Role> roleList) {
        for (Role role : roleList) {
            //同一类的方法调用自己方法，产生自调用[插入：失效]问题
            this.insertRole(role);
        }
        return roleList.size();
    }
}
~~~

## SpringBoot整合Redis

Spring Data Redis提供了RedisTemplate工具类，里面封装了对Redis的5种数据结构的各种操作。



- `RedisTemplate.opsForValue() `：操作字符串

- `RedisTemplate.opsForHash()` ：操作hash

- `RedisTemplate.opsForList()`：操作list

- `RedisTemplate.opsForSet()`：操作set

- `RedisTemplate.opsForZSet()`：操作zset

其他的通用命令,如expire,可以通过`RedisTemplate.xx()`来直接调用

### 普通整合

#### 依赖配置

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <!--不依赖redis的异步客户端-->
            <groupId>io.lettuce</groupId>
            <artifactId>lettuce-core</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<!--redis客户端驱动-->
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
~~~

~~~yaml
spring:
  redis:
    port: 6379
    host: localhost
    timeout: 2000 #毫秒
    jedis:
      pool:
        max-idle: 10
        min-idle: 5
        max-active: 10
        max-wait: 2000
~~~
#### 测试类
~~~java
/**
 * redisTemplate 操作存储数据到redis中，key显示为乱码。
 * 默认的序列化器为 JdkSerializationRedisSerializer
 * 存储string类型的，使用StringRedisTemplate.但是不能存储对象
 * StringRedisTemplate的序列化器为StringRedisSerializer
*/
//测试类
@SpringBootTest(classes = MyRedisApplication.class)
@RunWith(SpringRunner.class)
@Slf4j
public class RedisTest {
    @Autowired
    private RedisTemplate redisTemplate;
    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    /**
     * 将默认的redisTemplate的序列化器设置为stringSerializer
     */
    @Test
    public void initRedisTemplate() {
        RedisSerializer stringSerializer = redisTemplate.getStringSerializer();
        redisTemplate.setKeySerializer(stringSerializer);
        redisTemplate.setValueSerializer(stringSerializer);
        redisTemplate.setHashValueSerializer(stringSerializer);
        redisTemplate.setHashKeySerializer(stringSerializer);
        log.info("成功将"+redisTemplate.getClass().getSimpleName()+"的序列化器设置为了String类型");
        log.info(redisTemplate.getKeySerializer().toString());
        redisTemplate.opsForValue().set("springboot:test:test:redis_string_key", "value1");
    }

    @Test
    public void testStringAndHash() {
        stringRedisTemplate.opsForValue().set("springboot:test:test:string_int_key", "1");
        //使用运算
        stringRedisTemplate.opsForValue().increment("springboot:test:test:string_int_key", 1);
        String value = stringRedisTemplate.opsForValue().get("springboot:test:test:string_int_key");
        log.info(value);
    }
    /**
     * 测试操作列表（链表）可重复，顺序存取
     */
    @Test
    public void testList() {
        // 插入两个列表,注意它们在链表的顺序
        // 链表从左到右顺序为 v10,v8,v6,v4,v2
        stringRedisTemplate.opsForList().leftPushAll(
                "springboot:test:list1", "v2", "v4", "v6", "v8", "v10");
        // 链表从左到右顺序为 v1,v2,v3,v4,v5,v6
        stringRedisTemplate.opsForList().rightPushAll(
                "springboot:test:list2", "v1", "v2", "v3", "v4", "v5", "v6");
        // 绑定 list2 链表操作
        BoundListOperations listOps = stringRedisTemplate.boundListOps("springboot:test:list2");
        // 从右边弹出一个成员
        Object result1 = listOps.rightPop();
        // 获取定位元素，Redis 从 0 开始计算,这里值为 v2
        Object result2 = listOps.index(1);
        // 从左边插入链表
        listOps.leftPush("v0");
        // 求链表长度
        Long size = listOps.size();
        // 求链表下标区间成员，整个链表下标范围为 0 到 size-1，这里不取最后一个元素
        List elements = listOps.range(0, size-2);
        
    }
    /**
     * 测试操作无序集合set，无序
     */
    @Test
    public void testSet() {
        // 请注意：这里 v1 重复两次，因为集合不允许重复，所以只是插入 5 个成员到集合中
        stringRedisTemplate.opsForSet().add("springboot:test:set1",
                "v1","v1","v2","v3","v4","v5");
        stringRedisTemplate.opsForSet().add("springboot:test:set2", "v2","v4","v6","v8");
        // 绑定 set1 集合操作
        BoundSetOperations setOps = stringRedisTemplate.boundSetOps("springboot:test:set1");
        // 增加两个元素
        setOps.add("v6", "v7");
        // 删除两个元素
        setOps.remove("v1", "v7");
        // 返回所有元素
        Set set1 = setOps.members();
        // 求成员数
        Long size = setOps.size();
        // 求交集
        Set inter = setOps.intersect("springboot:test:set2");
        // 求交集，并且用新集合 inter 保存
        setOps.intersectAndStore("springboot:test:set2", "springboot:test:set:inter");
        // 求差集
        Set diff = setOps.diff("springboot:test:set2");
        // 求差集，并且用新集合 diff 保存
        setOps.diffAndStore("springboot:test:set2", "springboot:test:set:diff");
        // 求并集
        Set union = setOps.union("springboot:test:set2");
        // 求并集，并且用新集合 union 保存
        setOps.unionAndStore("springboot:test:set2", "springboot:test:set:union");
    }

    /**
     * 测试操作有序的set，不可重复
     */
    @Test
    public void testZset() {
        Set<ZSetOperations.TypedTuple<String>> typedTupleSet = new HashSet<>();
        for (int i=1; i<=9; i++) {
            // 分数
            double score = i*0.1;
        // 创建一个 TypedTuple 对象，存入值和分数
            ZSetOperations.TypedTuple<String> typedTuple
                    = new DefaultTypedTuple<>("value" + i, score);
            typedTupleSet.add(typedTuple);
        }
        // 往有序集合插入元素
        stringRedisTemplate.opsForZSet().add("springboot:test:zset1", typedTupleSet);
        // 绑定 zset1 有序集合操作
        BoundZSetOperations<String, String> zsetOps
                = stringRedisTemplate.boundZSetOps("springboot:test:zset1");
        // 增加一个元素
        zsetOps.add("value10", 0.26);
        Set<String> setRange = zsetOps.range(1, 6);
        // 按分数排序获取有序集合
        Set<String> setScore = zsetOps.rangeByScore(0.2, 0.6);
        // 定义值范围
        RedisZSetCommands.Range range = new RedisZSetCommands.Range();
        range.gt("value3");// 大于 value3
        //range.gte("value3");// 大于等于 value3
        //range.lt("value8");// 小于 value8
        //range.lte("value8");// 小于等于 value8
        // 按值排序，请注意这个排序是按字符串排序
        Set<String> setLex = zsetOps.rangeByLex(range);
        // 删除元素
        zsetOps.remove("value9", "value2");
        // 求分数
        Double score = zsetOps.score("value8");
        // 在下标区间下，按分数排序，同时返回 value 和 score
        Set<ZSetOperations.TypedTuple<String>> rangeSet = zsetOps.rangeWithScores(1, 6);
        // 在分数区间下，按分数排序，同时返回 value 和 score
        Set<ZSetOperations.TypedTuple<String>> scoreSet = zsetOps.rangeByScoreWithScores(1, 6);
        // 按从大到小排序
        Set<String> reverseSet = zsetOps.reverseRange(2, 8);
    }
}

~~~

### Redis事务

命令组合为**watch..multi..exec**

![Redis事务](https://gitee.com/tzcqupt/blog-image/raw/master/img/Redis事务.PNG)

#### 测试代码

~~~java
	/**
     * 1. 如果程序执行期间，key1的值被修改过（断点调试），则key2和key3没有值
     * 2. 程序执行期间，若①处的代码执行，则程序会抛出异常，因为字符串不能+1，但是key2和key3有值。这是redis事务的特点
     */
@Test
public void testMulti() {
    redisTemplate.opsForValue().set("springboot:test:multi:key1", "value1");
    List list = (List) redisTemplate.execute(new SessionCallback<Object>() {
        @Override
        public Object execute(RedisOperations operations) throws DataAccessException {
            // 设置要监控 key1
            operations.watch("springboot:test:multi:key1");
            // 开启事务，在 exec 命令执行前，全部都只是进入队列
            operations.multi();
            operations.opsForValue().set("springboot:test:multi:key2", "value2");
            operations.opsForValue().increment("springboot:test:multi:key1", 1);// ①此处执行命令会抛出异常
            // 获取值将为 null，因为 redis 只是把命令放入队列
            Object value2 = operations.opsForValue().get("springboot:test:multi:key2");
            log.info("命令在队列，所以 value 为 null【" + value2 + "】");
            operations.opsForValue().set("springboot:test:multi:key3", "value3");
            Object value3 = operations.opsForValue().get("springboot:test:multi:key3");
            log.info("命令在队列，所以 value 为 null【" + value3 + "】");
            // 执行 exec 命令，将先判别 key1 是否在监控后被修改过，如果是则不执行事务，否则就执行事务
            return operations.exec();// ②
        }
    });
    log.info(list.toString());
}
~~~

### Redis发布订阅

#### Java代码配置

~~~java
@SpringBootApplication
public class MyRedisApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyRedisApplication.class, args);
    }

    @Autowired
    private RedisTemplate redisTemplate = null;
    @Autowired
    private RedisConnectionFactory redisConnectionFactory = null;
    @Autowired
    private MessageListener redisMsgListener;
    /**
     * 任务池
     */
    private ThreadPoolTaskScheduler taskScheduler = null;

    /**
     * 创建任务池，运行线程等待处理redis的消息
     */
    @Bean
    public ThreadPoolTaskScheduler initTaskScheduler() {
        if (taskScheduler != null) {
            return taskScheduler;
        }
        taskScheduler = new ThreadPoolTaskScheduler();
        taskScheduler.setPoolSize(20);
        return taskScheduler;
    }

    /**
     * 定义redis的消息监听容器
     *
     * @return 监听容器
     */
    @Bean
    public RedisMessageListenerContainer initRedisContainer() {
        RedisMessageListenerContainer container
                = new RedisMessageListenerContainer();
        // Redis 连接工厂
        container.setConnectionFactory(redisConnectionFactory);
        // 设置运行任务池
        container.setTaskExecutor(initTaskScheduler());
        // 定义监听渠道，名称为 topic1
        Topic topic = new ChannelTopic("springboot:redis:topic");
        // 使用监听器监听 Redis 的消息
        container.addMessageListener(redisMsgListener, topic);
        return container;
    }
}

~~~

#### 测试代码

~~~java
@Test
public void testPublishMessage() {
    //测试发布消息
    redisTemplate.convertAndSend(
        "springboot:redis:topic", "测试redis发布订阅模式");
}
~~~