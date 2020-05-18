---
layout: post
title: 定时/延迟任务处理
date: 2020-05-10
Author: tzcqupt
tags: [Quartz]
categories: [定时任务]
comments: true
toc: true
---
# 延迟任务

需要延迟执行的任务，使用场景:

1. 红包 24 小时未被查收，需要延迟执退还业务；
2. 每个月账单日，需要给用户发送当月的对账单；
3. 订单下单之后 30 分钟后，用户如果没有付钱，系统需要自动取消订单。

## 实现思路分析

在某个时间节点执行某个任务。

1. 一直死循环判断当前时间节点有没有要执行的任务

2. 使用JDK或者其他工具类

   JDK工具:DelayQueue、ScheduledExecutorService

   其他工具:Redis、MQ、Netty

## 延迟任务实现

### 任务调用

```java
@Slf4j
public class DelayTaskExample {
    /**
     * 存放定时任务的Map
     */
    private static Map<String, Long> TaskMap = new HashMap<>(20);

    public static void main(String[] args) throws InterruptedException {
        log.info("程序启动时间:" + LocalDateTime.now());
        // 添加定时任务
        TaskMap.put("task-1", Instant.now().plusSeconds(3).toEpochMilli());
        TaskMap.put("task-2", Instant.now().plusSeconds(5).toEpochMilli());
        // 调用无限循环实现延迟任务
        /// loopTask();
        // 调用固定频率实现延迟任务
        /// scheduledExecutorServiceTask();
        // DelayQueue 方式
        DelayQueue delayQueue = new DelayQueue();
        // 添加延迟任务
        delayQueue.put(new DelayElement(1000));
        delayQueue.put(new DelayElement(3000));
        delayQueue.put(new DelayElement(5000));
        log.info("开始时间:" +  DateFormat.getDateTimeInstance().format(new Date()));
        while (!delayQueue.isEmpty()){
            // 执行延迟任务
            log.info(delayQueue.take().toString());
        }
        log.info("结束时间:" +  DateFormat.getDateTimeInstance().format(new Date()));
    }
}
```

### 无限循环实现延迟任务

```java
/**
 * 无限循环实现延迟任务
 */
public static void loopTask() {
    Long itemLong = 0L;
    while (true) {
        Iterator it = TaskMap.entrySet().iterator();
        while (it.hasNext()) {
            Map.Entry entry = (Map.Entry) it.next();
            itemLong = (Long) entry.getValue();
            // 有任务需要执行
            if (Instant.now().toEpochMilli() >= itemLong) {
                // 延迟任务，业务逻辑执行
                log.info("执行任务:" + entry.getKey() +
                        " ，执行时间:" + LocalDateTime.now());
                // 删除任务
                TaskMap.remove(entry.getKey());
            }
        }
    }
}
```

### `ScheduledExecutorSevice`实现延迟任务

```java
/**
 * 实现固定频率 一直循环执行任务
 */
public static void scheduledExecutorServiceTask() {
    ScheduledExecutorService service = Executors.newScheduledThreadPool(1);
    service.scheduleAtFixedRate(
            () -> log.info("正在执行任务,执行时间:" + LocalDateTime.now()), 2, 2, TimeUnit.SECONDS);

}
```

### `DelayQueue`实现延迟任务

`DelayQueue` 是一个支持延时获取元素的无界阻塞队列，队列中的元素必须实现 Delayed 接口，并重写 `getDelay(TimeUnit)` 和 `compareTo(Delayed)` 方法。

```java
static class DelayElement implements Delayed{
    // 延迟截止时间（单面:毫秒）
    long delayTime = System.currentTimeMillis();
    public DelayElement(long delayTime) {
        this.delayTime = (this.delayTime + delayTime);
    }

    /**
     * 获取剩余时间
     * @param unit 时间单位
     * @return
     */
    @Override
    public long getDelay(TimeUnit unit) {
        return unit.convert(delayTime - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    }

    // 队列里元素的排序依据
    @Override
    public int compareTo(Delayed o) {
        if (this.getDelay(TimeUnit.MILLISECONDS) > o.getDelay(TimeUnit.MILLISECONDS)) {
            return 1;
        } else if (this.getDelay(TimeUnit.MILLISECONDS) < o.getDelay(TimeUnit.MILLISECONDS)) {
            return -1;
        } else {
            return 0;
        }
    }
    @Override
    public String toString() {
        return DateFormat.getDateTimeInstance().format(new Date(delayTime));
    }
}
```

### `Redis`实现延迟任务

#### 通过数据判断的方式

**SpringBoot整合Redis的方式**

```xml
<!--Redis 配置-->
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
```

```java
@Slf4j
public class RedisTask {
    private static final String DELAY_QUEUE_KEY ="springboot:task:delayQueue";
    public static void main(String[] args) throws InterruptedException {
        long delayTime = Instant.now().plusSeconds(30).getEpochSecond();
        try (Jedis jedis = new Jedis()){
            jedis.zadd(DELAY_QUEUE_KEY, delayTime, "order_1");
            // 继续添加测试数据
            jedis.zadd(DELAY_QUEUE_KEY, Instant.now().plusSeconds(2).getEpochSecond(), "order_2");
            jedis.zadd(DELAY_QUEUE_KEY, Instant.now().plusSeconds(2).getEpochSecond(), "order_3");
            jedis.zadd(DELAY_QUEUE_KEY, Instant.now().plusSeconds(7).getEpochSecond(), "order_4");
            jedis.zadd(DELAY_QUEUE_KEY, Instant.now().plusSeconds(10).getEpochSecond(), "order_5");
            // 开启延迟队列
            doDelayQueue(jedis);
        }

    }
    /**
     * 延迟队列消费
     * @param jedis Redis 客户端
     */
    public static void doDelayQueue(Jedis jedis) throws InterruptedException {
        while (true) {
            // 当前时间
            Instant nowInstant = Instant.now();
            // 上一秒时间
            long lastSecond = nowInstant.plusSeconds(-1).getEpochSecond();
            long nowSecond = nowInstant.getEpochSecond();
            // 查询当前时间的所有任务
            Set<String> data = jedis.zrangeByScore(DELAY_QUEUE_KEY, lastSecond, nowSecond);
            for (String item : data) {
                // 消费任务
                log.info("消费:" + item);
            }
            // 删除已经执行的任务
            jedis.zremrangeByScore(DELAY_QUEUE_KEY, lastSecond, nowSecond);
            Thread.sleep(1000); // 每秒轮询一次
        }
    }
}
```

### `RabbitMQ`实现延迟任务

`RabbitMQ` 实现延迟队列的方式有两种:

- 通过消息过期后进入死信交换器，再由交换器转发到延迟消费队列，实现延迟功能；
- 使用 `rabbitmq-delayed-message-exchange` 插件实现延迟功能，插件安装方式参考[消息队列文章](https://tzcqupt.gitee.io/2020/05/10/mq-Learn/#RabbitMQ)。

#### `SpringBoot`配置`RabbitMQ`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

```properties
#RabbitMQ配置
spring.rabbitmq.host=127.0.0.1
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
```

#### `RabbitMQ`延迟队列配置

```java
@Configuration
public class DelayedConfig {
    final static String QUEUE_NAME = "springboot.delay.order";
    final static String EXCHANGE_NAME = "delayed_exchange";
    @Bean
    public Queue queue() {
        return new Queue(DelayedConfig.QUEUE_NAME);
    }

    /**
     * 配置默认的交换机
     * @return
     */
    @Bean
    CustomExchange customExchange() {
        Map<String, Object> args = new HashMap<>(20);
        args.put("x-delayed-type", "direct");
        //参数二为类型:必须是x-delayed-message
        return new CustomExchange(DelayedConfig.EXCHANGE_NAME, "x-delayed-message", true, false, args);
    }

    /**
     * 绑定队列到交换器
     * @param queue 队列
     * @param exchange 交换机
     * @return
     */
    @Bean
    Binding binding(Queue queue, CustomExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with(DelayedConfig.QUEUE_NAME).noargs();
    }
}
```

#### 发送端配置

```java
@Component
@Slf4j
public class DelayedSender {
    @Autowired
    private AmqpTemplate rabbitTemplate;

    public void send(String msg) {
        log.info("消息发送时间:" + Instant.now());

        rabbitTemplate.convertAndSend(DelayedConfig.EXCHANGE_NAME, DelayedConfig.QUEUE_NAME, msg, message -> {
            message.getMessageProperties().setHeader("x-delay", 3000);
            return message;
        });
    }
}
```

#### 接收端配置

```java
@Component
@RabbitListener(queues = "springboot.delay.order")
@Slf4j
public class DelayedReceiver {
    @RabbitHandler
    public void process(String msg) {
        log.info("消息接收时间:" + Instant.now());
        log.info("消息内容:" + msg);
    }
}
```

#### 测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = TaskApplication.class)
public class DelayedTest {

    @Autowired
    private DelayedSender sender;

    @Test
    public void TestMqDelay() throws InterruptedException {
        sender.send("Hi Admin.");
        // 等待接收程序执行之后，再退出测试
        Thread.sleep(5 * 1000);
    }
}
```

# Quartz

## 相关依赖

~~~xml
<dependencies>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz</artifactId>
            <version>2.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.quartz-scheduler</groupId>
            <artifactId>quartz-jobs</artifactId>
            <version>2.2.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-jcl</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
        </dependency>
</dependencies>
~~~



## 核心概念

### `Job`

任务接口，代表一个任务，要执行的具体内容。此接口中只有一个方法，自定义任务类需实现该接口。

`JobExecutionContext`类提供了调度上下文的各种信息。Job运行时的信息保存在 `JobDataMap`实例中；

~~~java
 void execute(JobExecutionContext context) throws JobExecutionException;
//自己的任务类
@Slf4j
public class RamJob implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) {
        log.info("hello quartz,first Job!!"+ Instant.now());
    }
}
~~~

### `JobDetail`

Quartz在每次执行Job时，都重新创建一个Job实例，所以它不直接接受一个Job的实例，相反它接收一个Job实现类，以便运行时通过`newInstance()`的反射机制实例化Job。

代表一个具体的可执行的调度程序，`Job`是这个可执行调度程序所要执行的内容。

~~~java
 JobDetail jobDetail =
                JobBuilder.newJob(RamJob.class)
                        // job描述
                        .withDescription("this is a ram job")
                        // job的name和group
                        .withIdentity("ram job", "ramGroup")
                        .build();
~~~



### `Trigger`

任务触发器，什么时候去触发自定义的任务，===>设置任务的调度策略。

#### `SimpleTrigger `

适用于在某一特定的时间执行一次，或者在某一特定的时间以某一特定时间间隔执行多次。上述功能决定了 `SimpleTrigger `的参数包括 `start-time`,` end-time`, `repeat count`, 以及 `repeat interval`。

#### `CronTrigger `

使用[corn表达式](https://cron.qqe2.com/)进行任务调度，`CronTrigger` 主要适用于基于日历的调度安排。

### `Calendar`

它是一些日历特定时间点的集合，一个`Trigger`可以和多个`Calendar`关联，以便排除或包含某些时间点。

假设，我们安排每周星期一早上10:00执行任务，但是如果碰到法定的节日，任务则不执行，这时就需要在Trigger触发机制的基础上使用 Calendar进行定点排除。针对不同时间段类型，Quartz在`org.quartz.impl.calendar`包下提供了若干个`Calendar` 的实现类，如`AnnualCalendar`、`MonthlyCalendar`、`WeeklyCalendar`分别针对每年、每月和每周进行定义。

举例使用:安排一个任务，每小时运行一次，并将五一节和国际节排除在外

~~~java
import java.util.Calendar;  
  
import java.util.Date;  
  
import java.util.GregorianCalendar;  
  
import org.quartz.impl.calendar.AnnualCalendar;  
  
import org.quartz.TriggerUtils;  
  
public class CalendarExample {  
  
public static void main(String[] args) throws Exception {  
SchedulerFactory sf = new StdSchedulerFactory();   
Scheduler scheduler = sf.getScheduler();   
//①法定节日是以每年为周期的，所以使用AnnualCalendar   
AnnualCalendar holidays = new AnnualCalendar();   
//②五一劳动节  
Calendar laborDay = new GregorianCalendar();   
laborDay.add(Calendar.MONTH,5);   
laborDay.add(Calendar.DATE,1);  
// ②-1:排除的日期，如果设置为false则为包含  
holidays.setDayExcluded(laborDay, true); 
//③国庆节  
Calendar nationalDay = new GregorianCalendar();   
nationalDay.add(Calendar.MONTH,10);   
nationalDay.add(Calendar.DATE,1);  
//③-1:排除该日期  
holidays.setDayExcluded(nationalDay, true);
//④向Scheduler注册日历  
scheduler.addCalendar("holidays", holidays, false, false);
//⑤4月1号 上午10点 
Date runDate = TriggerUtils.getDateOf(0,0, 10, 1, 4); 
JobDetail job = new JobDetail("job1", "group1", SimpleJob.class);   
SimpleTrigger trigger = new SimpleTrigger("trigger1", "group1", runDate,  null,   
  SimpleTrigger.REPEAT_INDEFINITELY, 60L * 60L * 1000L);  
//⑥让Trigger应用指定的日历规则  
trigger.setCalendarName("holidays");
scheduler.scheduleJob(job, trigger);   
scheduler.start();  
  
}    
}  
~~~



### `Scheduler`

 代表一个Quartz的独立运行容器，Trigger和JobDetail可以注册到Scheduler中，两者在 Scheduler中拥有各自的组及名称，组及名称是Scheduler查找定位容器中某一对象的依据，Trigger的组及名称必须唯一，JobDetail的组和名称也必须唯一（但可以和Trigger的组和名称相同，因为它们是不同类型的）。

Scheduler可以将Trigger绑定到某一JobDetail中，这样当Trigger触发时，对应的Job就被执行。一个Job可以对应 多个Trigger，但一个Trigger只能对应一个Job。可以通过SchedulerFactory创建一个Scheduler实例。 Scheduler拥有一个SchedulerContext，它类似于ServletContext，保存着Scheduler上下文信息，Job和 Trigger都可以访问SchedulerContext内的信息。



~~~java
StdSchedulerFactory schedulerFactory = new StdSchedulerFactory();
Scheduler scheduler = schedulerFactory.getScheduler();
scheduler.scheduleJob(jobDetail, trigger);
~~~

#### `misfire`

错过的，指本来应该被执行但实际没有被执行的任务调度

### 入门代码

#### `RamJob`

~~~java
@Slf4j
public class RamJob implements Job {
    @Override
    public void execute(JobExecutionContext jobExecutionContext) {
        log.info("hello quartz,first Job!!"+ Instant.now());
    }
}
~~~

#### `RamQuartz`

~~~java
@Slf4j
public class RamQuartz {
    public static void main(String[] args) throws SchedulerException, InterruptedException {
        StdSchedulerFactory schedulerFactory = new StdSchedulerFactory();
        Scheduler scheduler = schedulerFactory.getScheduler();
        JobDetail jobDetail =
                JobBuilder.newJob(RamJob.class)
                        // job描述
                        .withDescription("this is a ram job")
                        // job的name和group
                        .withIdentity("ram job", "ramGroup")
                        .build();
        // job开始运行的时间
        Date startTime = new Date(System.currentTimeMillis() + 3 * 1000L);
        Trigger trigger =
                TriggerBuilder.newTrigger()
                        .withDescription("job trigger")
                        .withIdentity("ram trigger", "ram trigger group")
                        // 开始运行时间
                        //.startAt(startTime)
                        .startNow()
                        // SimpleTrigger只需执行一次或者在给定时间触发并且重复N次且每次执行延迟一定时间的任务。
                        // .withSchedule(SimpleScheduleBuilder.simpleSchedule())
                        //CronTrigger:按照日历触发,例如“每个周五”,每个月10日中午或者10:15分 corn表达式https://cron.qqe2.com/
                        //.withSchedule(CronScheduleBuilder.cronSchedule("0/5 * * * * ? *"))
                        .withSchedule(CalendarIntervalScheduleBuilder.calendarIntervalSchedule().withIntervalInSeconds(1))
                        .build();
        // 注册任务和调度器
        scheduler.scheduleJob(jobDetail, trigger);
        scheduler.start();
        log.info("job任务启动时间" + Instant.now());
        //scheduler.shutdown();
    }
}
~~~

## 任务持久化

### `RAMJobStore`

Quartz的默认调度程序都存储在JVM的内存里面。

### `JDBCJobStore`

配置Quartz，将任务信息存储在数据库中。

### 配置文件

**[MySQL数据库文件](https://github.com/quartz-scheduler/quartz/blob/master/quartz-core/src/main/resources/org/quartz/impl/jdbcjobstore/tables_mysql.sql)**下载

~~~sql
DROP TABLE IF EXISTS QRTZ_FIRED_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_PAUSED_TRIGGER_GRPS;
DROP TABLE IF EXISTS QRTZ_SCHEDULER_STATE;
DROP TABLE IF EXISTS QRTZ_LOCKS;
DROP TABLE IF EXISTS QRTZ_SIMPLE_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_SIMPROP_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_CRON_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_BLOB_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_TRIGGERS;
DROP TABLE IF EXISTS QRTZ_JOB_DETAILS;
DROP TABLE IF EXISTS QRTZ_CALENDARS;


CREATE TABLE QRTZ_JOB_DETAILS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    JOB_NAME  VARCHAR(200) NOT NULL,
    JOB_GROUP VARCHAR(200) NOT NULL,
    DESCRIPTION VARCHAR(250) NULL,
    JOB_CLASS_NAME   VARCHAR(250) NOT NULL,
    IS_DURABLE VARCHAR(1) NOT NULL,
    IS_NONCONCURRENT VARCHAR(1) NOT NULL,
    IS_UPDATE_DATA VARCHAR(1) NOT NULL,
    REQUESTS_RECOVERY VARCHAR(1) NOT NULL,
    JOB_DATA BLOB NULL,
    PRIMARY KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
);

CREATE TABLE QRTZ_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_NAME VARCHAR(200) NOT NULL,
    TRIGGER_GROUP VARCHAR(200) NOT NULL,
    JOB_NAME  VARCHAR(200) NOT NULL,
    JOB_GROUP VARCHAR(200) NOT NULL,
    DESCRIPTION VARCHAR(250) NULL,
    NEXT_FIRE_TIME BIGINT(13) NULL,
    PREV_FIRE_TIME BIGINT(13) NULL,
    PRIORITY INTEGER NULL,
    TRIGGER_STATE VARCHAR(16) NOT NULL,
    TRIGGER_TYPE VARCHAR(8) NOT NULL,
    START_TIME BIGINT(13) NOT NULL,
    END_TIME BIGINT(13) NULL,
    CALENDAR_NAME VARCHAR(200) NULL,
    MISFIRE_INSTR SMALLINT(2) NULL,
    JOB_DATA BLOB NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,JOB_NAME,JOB_GROUP)
        REFERENCES QRTZ_JOB_DETAILS(SCHED_NAME,JOB_NAME,JOB_GROUP)
);

CREATE TABLE QRTZ_SIMPLE_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_NAME VARCHAR(200) NOT NULL,
    TRIGGER_GROUP VARCHAR(200) NOT NULL,
    REPEAT_COUNT BIGINT(7) NOT NULL,
    REPEAT_INTERVAL BIGINT(12) NOT NULL,
    TIMES_TRIGGERED BIGINT(10) NOT NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
        REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_CRON_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_NAME VARCHAR(200) NOT NULL,
    TRIGGER_GROUP VARCHAR(200) NOT NULL,
    CRON_EXPRESSION VARCHAR(200) NOT NULL,
    TIME_ZONE_ID VARCHAR(80),
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
        REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_SIMPROP_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_NAME VARCHAR(200) NOT NULL,
    TRIGGER_GROUP VARCHAR(200) NOT NULL,
    STR_PROP_1 VARCHAR(512) NULL,
    STR_PROP_2 VARCHAR(512) NULL,
    STR_PROP_3 VARCHAR(512) NULL,
    INT_PROP_1 INT NULL,
    INT_PROP_2 INT NULL,
    LONG_PROP_1 BIGINT NULL,
    LONG_PROP_2 BIGINT NULL,
    DEC_PROP_1 NUMERIC(13,4) NULL,
    DEC_PROP_2 NUMERIC(13,4) NULL,
    BOOL_PROP_1 VARCHAR(1) NULL,
    BOOL_PROP_2 VARCHAR(1) NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
    REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_BLOB_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_NAME VARCHAR(200) NOT NULL,
    TRIGGER_GROUP VARCHAR(200) NOT NULL,
    BLOB_DATA BLOB NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP),
    FOREIGN KEY (SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
        REFERENCES QRTZ_TRIGGERS(SCHED_NAME,TRIGGER_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_CALENDARS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    CALENDAR_NAME  VARCHAR(200) NOT NULL,
    CALENDAR BLOB NOT NULL,
    PRIMARY KEY (SCHED_NAME,CALENDAR_NAME)
);

CREATE TABLE QRTZ_PAUSED_TRIGGER_GRPS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    TRIGGER_GROUP  VARCHAR(200) NOT NULL,
    PRIMARY KEY (SCHED_NAME,TRIGGER_GROUP)
);

CREATE TABLE QRTZ_FIRED_TRIGGERS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    ENTRY_ID VARCHAR(95) NOT NULL,
    TRIGGER_NAME VARCHAR(200) NOT NULL,
    TRIGGER_GROUP VARCHAR(200) NOT NULL,
    INSTANCE_NAME VARCHAR(200) NOT NULL,
    FIRED_TIME BIGINT(13) NOT NULL,
    SCHED_TIME BIGINT(13) NOT NULL,
    PRIORITY INTEGER NOT NULL,
    STATE VARCHAR(16) NOT NULL,
    JOB_NAME VARCHAR(200) NULL,
    JOB_GROUP VARCHAR(200) NULL,
    IS_NONCONCURRENT VARCHAR(1) NULL,
    REQUESTS_RECOVERY VARCHAR(1) NULL,
    PRIMARY KEY (SCHED_NAME,ENTRY_ID)
);

CREATE TABLE QRTZ_SCHEDULER_STATE
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    INSTANCE_NAME VARCHAR(200) NOT NULL,
    LAST_CHECKIN_TIME BIGINT(13) NOT NULL,
    CHECKIN_INTERVAL BIGINT(13) NOT NULL,
    PRIMARY KEY (SCHED_NAME,INSTANCE_NAME)
);

CREATE TABLE QRTZ_LOCKS
  (
    SCHED_NAME VARCHAR(120) NOT NULL,
    LOCK_NAME  VARCHAR(40) NOT NULL,
    PRIMARY KEY (SCHED_NAME,LOCK_NAME)
);


commit;
~~~



~~~properties
#quartz.properties
#调度标识名 集群中每一个实例都必须使用相同的名称 （区分特定的调度器实例）
org.quartz.scheduler.instanceName:DefaultQuartzScheduler
#ID设置为自动获取 每一个必须不同 （所有调度器实例中是唯一的）
org.quartz.scheduler.instanceId=AUTO
#数据保存方式为持久化 
org.quartz.jobStore.class =org.quartz.impl.jdbcjobstore.JobStoreTX
#表的前缀 
org.quartz.jobStore.tablePrefix = QRTZ_
#设置为TRUE不会出现序列化非字符串类到 BLOB 时产生的类版本问题
#org.quartz.jobStore.useProperties = true
#加入集群 true 为集群 false不是集群
org.quartz.jobStore.isClustered = false
#调度实例失效的检查时间间隔 
org.quartz.jobStore.clusterCheckinInterval=20000 
#容许的最大作业延长时间 
org.quartz.jobStore.misfireThreshold =60000
#ThreadPool 实现的类名 
org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
#线程数量 
org.quartz.threadPool.threadCount = 10
#线程优先级 
org.quartz.threadPool.threadPriority = 5（threadPriority 属性的最大值是常量 java.lang.Thread.MAX_PRIORITY，等于10。最小值为常量 java.lang.Thread.MIN_PRIORITY，为1）
#自创建父线程
#org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread= true 
#数据库别名
org.quartz.jobStore.dataSource = qzDS
#设置数据源
org.quartz.dataSource.qzDS.driver=com.mysql.jdbc.Driver
org.quartz.dataSource.qzDS.URL=jdbc:mysql://localhost:3306/quartz
org.quartz.dataSource.qzDS.user=root
org.quartz.dataSource.qzDS.password=123456
org.quartz.dataSource.qzDS.maxConnection=10

~~~

# `SpringCloud整合Quartz`

场景：业务代码采用的一个数据库，配置quartz采用的是另外的一个数据库来保存任务信息。

## 依赖

~~~xml
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.5.RELEASE</version>
		<relativePath/>
	</parent>
	<description>物业费模块</description>
	<repositories><!-- 代码库 -->
		<repository>
			<id>maven-ali</id>
			<url>http://maven.aliyun.com/nexus/content/groups/public//</url>
			<releases>
				<enabled>true</enabled>
			</releases>
			<snapshots>
				<enabled>true</enabled>
				<updatePolicy>always</updatePolicy>
				<checksumPolicy>fail</checksumPolicy>
			</snapshots>
		</repository>
	</repositories>
	<properties>
		<java.version>1.8</java.version>
		<mybatis-plus.version>3.0.5</mybatis-plus.version>
		<spring-cloud.version>Hoxton.SR3</spring-cloud.version>
	</properties>
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
			<optional>true</optional>
		</dependency>
		<!--mybatis-plus 持久层 -->
		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>${mybatis-plus.version}</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<scope>provided</scope>
		</dependency>
		<!--Spring Cloud -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-openfeign</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
		</dependency>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<!--quartz 任务调度框架-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-quartz</artifactId>
		</dependency>
	</dependencies>
~~~

## `yaml`配置

~~~yaml
spring:
  application:
    name: code-test
  jackson:
    default-property-inclusion: non_null  #返回json对象时过滤null属性
  datasource:
  	primary:
      driver-class-name: com.mysql.cj.jdbc.Driver
      # ?serverTimezone=GMT%2B8 是必须的
      url: jdbc:mysql://localhost:3306/code-test?serverTimezone=GMT%2B8&useSSL=false
      username: root
      password: root123
 	scheduler:
      driver-class-name: com.mysql.cj.jdbc.Driver
      # ?serverTimezone=GMT%2B8 是必须的
      url: jdbc:mysql://localhost:3306/quartz-test?serverTimezone=GMT%2B8&useSSL=false
      username: root
      password: root123
#mybatis-plus在代码中配置
#SpringCloud相关配置
eureka:
  instance: #在springCloud中服务的InstanceID默认值是:主机名：应用名：应用端口
    lease-renewal-interval-in-seconds: 15 #每隔N秒发送心跳续约,默认30
    lease-expiration-duration-in-seconds: 30 #如果N秒不发eureka就剔除当前注册实例,默认90
    prefer-ip-address: true #Eureka优先发布服务的IP地址而不是主机名,当应用程序向eureka注册时,它将使用其IP地址而不是其主机名
    instance-id: ${spring.cloud.client.ip-address}:${server.port}
  client:
    fetch-registry: true #定期更新eureka server拉去服务节点清单,快速感知服务下线
    registry-fetch-interval-seconds: 10 #指示从eureka服务器获取注册表信息的频率（以秒为单位）（网关项目可以缩小该值）,默认30
    instance-info-replication-interval-seconds: 10 #每隔N秒扫描一次本地实例,如果有变化向服务重新注册（以秒为单位）,默认30
    serviceUrl:
      defaultZone: http://127.0.0.1:8761/eureka/,http://127.0.0.1:8762/eureka/
    healthcheck:  #健康检查
      enabled: true
ribbon:
  ConnectTimeout: 2000 #http建立socket超时时间(毫秒),ribbon请求连接的超时时间,默认值2000（则表示连接服务的时间,一般不用配置太久,1~2秒左右就可以了）
  ReadTimeout: 2500 #http读取响应socket超时时间,负载均衡超时时间,默认值5000（当调用某个服务等待时间过长的时候, 对超时报错/熔断生效的是ReadTimeout）
  MaxAutoRetries: 0 #同一台实例最大重试次数,不包括首次调用
  MaxAutoRetriesNextServer: 1 #重试负载均衡其他的实例最大重试次数,切换相同Server的次数,默认1(不包括首次server)
  OkToRetryOnAllOperations: true #是否所有操作都重试,POST请求注意多次提交错误。默认false,设定为false的话,只有get请求会重试
  ServerListRefreshInterval: 5000 #调整刷新server list的时间间隔参数,默认30秒

feign:
  hystrix: #feign整合hystrix的开关
    enabled: true

hystrix:
  threadpool:
    default:
      coreSize: 50 #当使用线程隔离策略时,线程池的核心大小(Hystrix线程池大小默认为10)
      maximumSize: 100 #当Hystrix隔离策略为线程池隔离模式时,最大线程池大小的配置
      allowMaximumSizeToDivergeFromCoreSize: true #此属性语序配置的maximumSize生效
  command:
    default: #default全局有效,service id指定应用有效
      circuitBreaker: # 断路器配置
        requestVolumeThreshold: 20 #当在配置时间窗口内达到此数量的失败后,进行断路。默认:20个
        errorThresholdpercentage: 50 #出错百分比阈值,当达到此阈值后,开始断路。默认:50%
        sleepWindowInMilliseconds: 5000 #触发短路的时间值,当该值设为5000时,则当触发circuit break后的5000毫秒内都会拒绝request,也就是5000毫秒后才会关闭circuit。默认:5000
        forceOpen: false   #强制打开断路器,如果打开这个开关,那么拒绝所有request,默认false
        forceClosed: false  #强制关闭断路器 如果这个开关打开,circuit将一直关闭且忽略,默认false
      execution: #熔断器配置
        timeout:
          enabled: true #如果enabled为true,则会有两个执行方法超时的配置(ribbon的ReadTimeout,一个就是熔断器hystrix的timeoutInMilliseconds,此时谁的值小谁生效),
          #如果enabled为false,则熔断器不进行超时熔断,而是根据ribbon的ReadTimeout抛出的异常而熔断,也就是取决于ribbon
        isolation:
          thread:
            timeoutInMilliseconds: 2000 #断路器超时时间,默认1000ms(为了确保重试机制的正常运作,理论上（以实际情况为准）建议hystrix的超时时间为:(1 + MaxAutoRetries + MaxAutoRetriesNextServer) * ReadTimeout)
            interruptOnTimeout: true #超时时是否立马中断

management:
  endpoint:
    shutdown:
      enabled: true
    health:
      show-details: ALWAYS
  endpoints:
    web:
      exposure:
        include: '*'
~~~

## 数据源配置

> 这里的数据源采用的是properties配置，上面的yaml只是配置参考。

~~~java
@Configuration
public class DataSourceConfig {

	@Bean
	@Primary
	@ConfigurationProperties(prefix = "spring.datasource.primary")
	public DataSourceProperties primaryDataSourceProperties() {
		return new DataSourceProperties();
	}

	@Bean(name = "primaryDataSource")
	public DataSource primaryDataSource() {
		return primaryDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
	}

	/**
	 * 配置@Transactional注解事务
	 */
	@Bean
	public PlatformTransactionManager transactionManager() {
		return new DataSourceTransactionManager(primaryDataSource());
	}

	@Bean
	@ConfigurationProperties(prefix = "spring.datasource.scheduler")
	public DataSourceProperties quartzDataSourceProperties() {
		return new DataSourceProperties();
	}

	@Bean(name = "quartzDataSource")
	@QuartzDataSource
	public DataSource quartzDataSource() {
		return quartzDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
	}
}
~~~

## `MyBatis-Plus`配置

### Meta元素处理器

~~~java
@Slf4j
@Component
public class MyBatisPlusMetaObjectHandler implements MetaObjectHandler {
    @Override
    public void insertFill(MetaObject metaObject) {
        log.info("start insert fill ...");
        this.setFieldValByName("createTime", new Date(), metaObject);
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }

    @Override
    public void updateFill(MetaObject metaObject) {
        log.info("start update fill ....");
        this.setFieldValByName("updateTime", new Date(), metaObject);
    }
}
~~~

### 主配置

~~~java
@Configuration
@Slf4j
@AutoConfigureAfter(MyBatisPlusMetaObjectHandler.class)
public class MybatisPlusConfig {

  /** 分页插件 */
  @Bean
  public PaginationInterceptor paginationInterceptor() {
    return new PaginationInterceptor();
  }

  /** 逻辑删除标志 */
  @Bean
  public ISqlInjector sqlInjector() {
    return new LogicSqlInjector();
  }
  @Autowired
  private MyBatisPlusMetaObjectHandler metaObjectHandler;
  @Bean
  @ConditionalOnMissingBean // 当容器里没有指定的Bean的情况下创建该对象
  public MybatisSqlSessionFactoryBean sqlSessionFactory(
      @Qualifier("primaryDataSource") DataSource dataSource) {
    MybatisSqlSessionFactoryBean sqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
    // 设置数据源
    sqlSessionFactoryBean.setDataSource(dataSource);
    ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
    Resource[] resources;
    try {
      resources = resolver.getResources("classpath:/mapper/*.xml");
      sqlSessionFactoryBean.setTypeAliasesPackage("com.tang.code.entity");
      sqlSessionFactoryBean.setTypeEnumsPackage("com.tang.code.entity.enums");
    } catch (IOException e) {
      log.info("找不到mapper文件" + e.getMessage());
      throw new TangException(ResultCodeEnum.INTERNAL_SERVER_ERROR);
    }
    sqlSessionFactoryBean.setMapperLocations(resources);
    GlobalConfig globalConfig = new GlobalConfig();
    GlobalConfig.DbConfig dbConfig = new GlobalConfig.DbConfig();
    dbConfig
        .setDbType(DbType.MYSQL)
        .setIdType(IdType.AUTO)
        // 逻辑删除
        .setLogicDeleteValue("2")
        .setLogicNotDeleteValue("1")
        .setTableUnderline(true);
    globalConfig.setDbConfig(dbConfig);
    // 元素处理器
    globalConfig.setMetaObjectHandler(metaObjectHandler);
    sqlSessionFactoryBean.setGlobalConfig(globalConfig);
    // 全局配置
    MybatisConfiguration configuration = new MybatisConfiguration();
    configuration.setLogImpl(StdOutImpl.class);
    sqlSessionFactoryBean.setConfiguration(configuration);

    return sqlSessionFactoryBean;
  }
}
~~~

### MapperScannerConfig

~~~java
@Configuration
@AutoConfigureAfter(MybatisPlusConfig.class) // 保证在MybatisPlusConfig实例化之后再实例化该类
public class MyBatisMapperScannerConfig {

	@Bean // mapper接口的扫描器
	@ConditionalOnMissingBean // 当容器里没有指定的Bean的情况下创建该对象
	public MapperScannerConfigurer mapperScannerConfigurer() {
		MapperScannerConfigurer mapperScannerConfigurer = new MapperScannerConfigurer();
		mapperScannerConfigurer.setBasePackage("com.tang.code.mapper");
		return mapperScannerConfigurer;
	}
}
~~~



## `SpringCloud`相关配置

### 提供自定义健康信息

~~~java
/**
 * 提供自定义健康信息
 * 
 */
@Component
public class MyHealthIndicator implements HealthIndicator {

	@Override
	public Health health() {
		return Health.up().build();
	}
}
~~~



### 健康状态变化

~~~java
/**
 * 通知eureka健康状态变化
 *
 */
@Component
public class MyHealthCheckHandler implements HealthCheckHandler {

	@Autowired
	private MyHealthIndicator healthIndicator;

	@Override
	public InstanceStatus getStatus(InstanceStatus instanceStatus) {
		Status status = healthIndicator.health().getStatus();
		if (status.equals(Status.UP)) {
			return InstanceStatus.UP;
		} else {
			return InstanceStatus.DOWN;
		}
	}
}
~~~

## `Quartz`相关配置

### 配置文件

~~~properties
#==============================================================
#Configure Main Scheduler Properties
#==============================================================
#用在JobStore中来唯一标识实例,但是所有集群节点中必须相同。
org.quartz.scheduler.instanceName=quartz-test
#如果使用集群,instanceId必须唯一,设置成AUTO
org.quartz.scheduler.instanceId=AUTO
#==============================================================
#Configure JobStore
#==============================================================
#将任务持久化到数据中
#是不受应用容器事物管理的数据库存储实现类
org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
#是受应用容器管理事物的数据库存储实现类
#org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreCMT
org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.tablePrefix=QRTZ_
#参与到一个集群当中。这一属性会贯穿于调度框架的始终,用于修改集群环境中操作的默认行为。
org.quartz.jobStore.isClustered=true
#实例检入到数据库中的频率(单位：毫秒)
org.quartz.jobStore.clusterCheckinInterval=20000
org.quartz.jobStore.maxMisfiresToHandleAtATime=1
org.quartz.jobStore.misfireThreshold=120000
#这个选项在mysql下会非常容易出现死锁问题。quartz需要提升隔离级别来保障自己的运作，不过，由于各数据库实现的隔离级别定义都不一样，所以quartz提供一个设置序列化这样的隔离级别存在，因为例如oracle中是没有未提交读和可重复读这样的隔离级别存在。但是由于mysql默认的是可重复读，比提交读高了一个级别，所以已经可以满足quartz集群的正常运行。
org.quartz.jobStore.txIsolationLevelSerializable=false
#调度流程的第一步，也就是拉取待即将触发的triggers时，是上锁的状态，即不会同时存在多个线程拉取到相同的trigger的情况，也就避免的重复调度的危险。
org.quartz.jobStore.acquireTriggersWithinLock=true
#==============================================================
#Configure ThreadPool
#==============================================================
org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount=10
org.quartz.threadPool.threadPriority=5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread=true
#==============================================================
#Skip Check Update
#update:true
#not update:false
#==============================================================
org.quartz.scheduler.skipUpdateCheck=true
#============================================================================
# Configure Plugins
#============================================================================
org.quartz.plugin.triggHistory.class=org.quartz.plugins.history.LoggingJobHistoryPlugin
org.quartz.plugin.shutdownhook.class=org.quartz.plugins.management.ShutdownHookPlugin
org.quartz.plugin.shutdownhook.cleanShutdown=true
~~~

### `SpringBeanJobFactory`

~~~java
**
 * 自定义SpringBeanJobFactory
 * 
 */
@Configuration
public class AutoWiringSpringBeanJobFactory extends SpringBeanJobFactory {

	@Autowired
	private AutowireCapableBeanFactory beanFactory;

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		beanFactory = applicationContext.getAutowireCapableBeanFactory();
	}

	/**
	 * 这里覆盖了super的createJobInstance方法,对其创建出来的类再进行autowire
	 * 
	 * @throws Exception
	 */
	@Override
	protected Object createJobInstance(TriggerFiredBundle bundle) throws Exception {
		final Object jobInstance = super.createJobInstance(bundle);
		beanFactory.autowireBean(jobInstance);
		return jobInstance;
	}
}
~~~

### Quartz任务配置

```java
@Configuration
public class MyQuartzConfig {

    /**
     * 定义JobDetail,直接使用jobDetailFactoryBean.getObject获得的是空
     * 物业费账单生成。
     * @return
     */
    @Bean
    public JobDetailFactoryBean jobDetailPropertyCostBill() {
        JobDetailFactoryBean jobDetailFactoryBean = new JobDetailFactoryBean();
        jobDetailFactoryBean.setGroup("tangGroup");
        // durability表示在没有trigger关联的时候任务是否被保留,任务完成之后是否依然保留到数据库,默认false
        jobDetailFactoryBean.setDurability(true);
        // 当Quartz服务被中止后,再次启动或集群中其他机器接手任务时会尝试恢复执行之前未完成的所有任务
        jobDetailFactoryBean.setRequestsRecovery(true);
        jobDetailFactoryBean.setJobClass(QuartzTest.class);
        jobDetailFactoryBean.setDescription("quartz 整合测试");

      /*Map<String, String> jobDataAsMap = new HashMap<>();
      jobDataAsMap.put("targetObject", "targetObject"); // spring中bean的名字
      jobDataAsMap.put("targetMethod", "execute"); // 执行方法名
      jobDetailFactoryBean.setJobDataAsMap(jobDataAsMap);*/
        return jobDetailFactoryBean;
    }

    /**
     * 定义一个Trigger
     *
     * @return
     */
    @Bean
    public CronTriggerFactoryBean triggerPropertyCostBill(JobDetail jobDetailPropertyCostBill) {
        CronTriggerFactoryBean cronTriggerFactoryBean = new CronTriggerFactoryBean();
        // 设置jobDetail
        cronTriggerFactoryBean.setJobDetail(jobDetailPropertyCostBill);
        // 每晚11:55:00 执行任务0 55 23 * * ?
        cronTriggerFactoryBean.setCronExpression("0 55 23 * * ?");
        // trigger超时处理策略 默认1：总是会执行头一次 2:不处理
        cronTriggerFactoryBean.setMisfireInstruction(1);

        return cronTriggerFactoryBean;
    }
}
```

### `QuartzTest`

```java
//执行后保留作业数据
@PersistJobDataAfterExecution
//不允许并发执行，后一个任务需等待前一个任务完成
@DisallowConcurrentExecution
@Component
@Slf4j
public class QuartzTest extends QuartzJobBean {
  
  @Override
  protected void executeInternal(JobExecutionContext jobexecutioncontext) {
     log.info("quartz执行任务完成!");
  }
}
```

# `SpringBoot`中的定时任务

## 开启定时任务

在启动类商添加`@EnableScheduling`即可。

~~~java
@SpringBootApplication
@EnableScheduling
public class TaskApplication {
    public static void main(String[] args){
        SpringApplication.run(TaskApplication.class,args);
    }
}
~~~

## SpringBoot中使用

### corn表达式

corn在线表达式:http://cron.qqe2.com/

新建一个类，加上`@Component`被Spring管理

~~~java
@Component
@Slf4j
public class ScheduledTask {

    @Scheduled(cron = "0/5 * * * * ? ")
    public void printLog1() {
        //corn在线表达式 http://cron.qqe2.com/
        log.info("我是corn表达式的定时任务，每5秒打印一次时间" + new Date().toString());
    }
}
~~~

### fixRate

fixedRate:固定速率，单位是毫秒。表示一个**任务开始后**间隔多长时间才执行下一个执行

fixedRateString:从配置文件读取的固定速率。

~~~java
	/**
     * 2秒打印一次日志
     * 比如设置 fixedRate=5*1000。那么如果一个任务8:00:00开始执行，不管什么时候执行结束，该任务的下一次执行的时间是8:00:05
     */
    @Scheduled(fixedRate = 2000)
    public void printLog2() {
        log.info("我是fixedRate固定速率定时任务，每2秒打印一次时间,不管该任务执行多久" + new Date().toString());
    }
~~~

### fixDelay

fixedDelay:固定延迟，单位是毫秒。表示一个任务**执行结束后**间隔多长时间才执行下一个执行

fixedDelayString:从配置文件读取的固定延迟

```java
/**
 * 3秒打印一次日志
 * 比如设置 fixedDelay=5*1000。那么如果一个任务8:00:00开始执行，8:00:01执行结束，那么该任务的下一次执行的时间是8:00:06
 */
@Scheduled(fixedDelay = 1000)
public void printLog3() throws InterruptedException {
    Thread.sleep(2000);
    log.info("我是fixedDelay固定延迟，每次任务需要执行2s，延迟1秒执行" + new Date().toString());
}
```

### fixedDelayString

```java
@Scheduled(fixedDelayString = "${eventTracking.delayFixed}")
private void printLog4() throws InterruptedException {
    Thread.sleep(3000);
    log.info("我是fixedDelayString从配置文件读取的固定延迟，每次任务需要执行3s，延迟3秒执行" + new Date().toString());
}
```

### initialDelay

initialDelay:服务启动后立即延迟指定时间执行，单位是毫秒

```java
@Scheduled(initialDelay = 10000,fixedDelay = 10000)
private void printLog5() {
   log.info("服务启动后10s，我开始执行了,10秒执行一次"+new Date().toString());
}
```

# SpringBoot中的异步任务

## 开启异步任务

1. 在启动类上加`@EnableAsync`即可

    ```java
    @SpringBootApplication
    @EnableScheduling
    @EnableAsync
    public class TaskApplication {
        public static void main(String[] args){
            SpringApplication.run(TaskApplication.class,args);
        }
    }
    ```

2. 在异步任务类中加上`@Async`注解，也可以具体到某个方法上

## 测试任务不带返回值

### 任务类

```java
@Component
@Slf4j
@Async
public class AsyncTask {

    public void task1() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(1000L);
        long end = System.currentTimeMillis();
        log.info("任务1耗时= " + (end - start) + "毫秒");
    }

    public void task2() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(2000L);
        long end = System.currentTimeMillis();
        log.info("任务2耗时= " + (end - start) + "毫秒");
    }

    public void task3() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(3000L);
        long end = System.currentTimeMillis();
        log.info("任务3耗时= " + (end - start) + "毫秒");
    }

    public void task4() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(4000L);
        long end = System.currentTimeMillis();
        log.info("任务4耗时= " + (end - start) + "毫秒");
    }
}
```

### 测试类

```java
@RestController
@RequestMapping("async")
@Slf4j
public class AsyncTaskController {
    @Autowired
    private AsyncTask task;

    /**
     * 测试任务不带返回值
     *
     * @return json
     * @throws InterruptedException
     */
    @GetMapping("test1")
    public ResponseEntity<Long> asyncTask1() throws InterruptedException {
        //http://localhost:8080/async/test1
        long start = System.currentTimeMillis();
        task.task1();
        task.task2();
        task.task3();
        task.task4();
        long end = System.currentTimeMillis();
        long result = end - start;
        log.info("测试任务不带返回值全部完成，耗时" + result + "毫秒");
        return ResponseEntity.ok(result);
    }
}
```

## 测试任务带返回值

### 任务类

```java
@Component
@Slf4j
@Async
public class AsyncTask {

    public Future<String> task5() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(1000L);
        long end = System.currentTimeMillis();
        log.info("任务5耗时= " + (end - start) + "毫秒");
        return new AsyncResult<>("任务5");
    }

    public Future<String> task6() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(2000L);
        long end = System.currentTimeMillis();
        log.info("任务6耗时= " + (end - start) + "毫秒");
        return new AsyncResult<>("任务6");
    }

    public Future<String> task7() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(3000L);
        long end = System.currentTimeMillis();
        log.info("任务7耗时= " + (end - start) + "毫秒");
        return new AsyncResult<>("任务7");
    }

    public Future<String> task8() throws InterruptedException {
        long start = System.currentTimeMillis();
        Thread.sleep(4000L);
        long end = System.currentTimeMillis();
        log.info("任务8耗时= " + (end - start) + "毫秒");
        return new AsyncResult<>("任务8");
    }
}
```

### 测试类

```java
@RestController
@RequestMapping("async")
@Slf4j
public class AsyncTaskController {
    @Autowired
    private AsyncTask task;
    /**
     * 测试任务带返回值
     *
     * @return json
     * @throws InterruptedException
     */
    @GetMapping("test2")
    public ResponseEntity<Long> asyncTask2() throws InterruptedException {
        //http://localhost:8080/async/test2
        long start = System.currentTimeMillis();
        Future<String> task5 = task.task5();
        Future<String> task6 = task.task6();
        Future<String> task7 = task.task7();
        Future<String> task8 = task.task8();
        for (; ; ) {
            if (task5.isDone() && task6.isDone() && task7.isDone() && task8.isDone()) {
                break;
            }
        }
        long end = System.currentTimeMillis();
        long result = end - start;
        log.info("测试任务带返回值全部完成，耗时" + result + "毫秒");
        return ResponseEntity.ok(result);
    }
}
```