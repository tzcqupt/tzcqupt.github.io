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

需要延迟执行的任务，使用场景：

1. 红包 24 小时未被查收，需要延迟执退还业务；
2. 每个月账单日，需要给用户发送当月的对账单；
3. 订单下单之后 30 分钟后，用户如果没有付钱，系统需要自动取消订单。

## 实现思路分析

在某个时间节点执行某个任务。

1. 一直死循环判断当前时间节点有没有要执行的任务

2. 使用JDK或者其他工具类

   JDK工具：DelayQueue、ScheduledExecutorService

   其他工具：Redis、MQ、Netty

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
        log.info("程序启动时间：" + LocalDateTime.now());
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
        log.info("开始时间：" +  DateFormat.getDateTimeInstance().format(new Date()));
        while (!delayQueue.isEmpty()){
            // 执行延迟任务
            log.info(delayQueue.take().toString());
        }
        log.info("结束时间：" +  DateFormat.getDateTimeInstance().format(new Date()));
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
                log.info("执行任务：" + entry.getKey() +
                        " ，执行时间：" + LocalDateTime.now());
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
            () -> log.info("正在执行任务,执行时间：" + LocalDateTime.now()), 2, 2, TimeUnit.SECONDS);

}
```

### `DelayQueue`实现延迟任务

`DelayQueue` 是一个支持延时获取元素的无界阻塞队列，队列中的元素必须实现 Delayed 接口，并重写 `getDelay(TimeUnit)` 和 `compareTo(Delayed)` 方法。

```java
static class DelayElement implements Delayed{
    // 延迟截止时间（单面：毫秒）
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
                log.info("消费：" + item);
            }
            // 删除已经执行的任务
            jedis.zremrangeByScore(DELAY_QUEUE_KEY, lastSecond, nowSecond);
            Thread.sleep(1000); // 每秒轮询一次
        }
    }
}
```

### `RabbitMQ`实现延迟任务

`RabbitMQ` 实现延迟队列的方式有两种：

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
        //参数二为类型：必须是x-delayed-message
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
        log.info("消息发送时间：" + Instant.now());

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
        log.info("消息内容：" + msg);
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

corn在线表达式：http://cron.qqe2.com/

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

fixedRate：固定速率，单位是毫秒。表示一个**任务开始后**间隔多长时间才执行下一个执行

fixedRateString：从配置文件读取的固定速率。

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

fixedDelay：固定延迟，单位是毫秒。表示一个任务**执行结束后**间隔多长时间才执行下一个执行

fixedDelayString：从配置文件读取的固定延迟

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

initialDelay：服务启动后立即延迟指定时间执行，单位是毫秒

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