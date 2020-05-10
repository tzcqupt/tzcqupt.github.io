---
layout: post
title: 消息队列入门学习笔记
date: 2020-05-10
Author: tzcqupt
tags: [RabbitMQ]
categories: [消息队列]
comments: true
toc: true
---
# AMQP和JMS区别

MQ是消息通信的模型，并不是具体实现。现在实现MQ的有两种主流方式：AMQP、JMS。

## **AMQP**

AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级[消息](https://baike.baidu.com/item/消息/1619218)队列协议，是[应用层](https://baike.baidu.com/item/应用层/4329788)协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/[中间件](https://baike.baidu.com/item/中间件/452240)不同产品，不同的开发语言等条件的限制。[Erlang](https://baike.baidu.com/item/Erlang)中的实现有[RabbitMQ](https://baike.baidu.com/item/RabbitMQ)等。

## **JMS**

![JMS概念](https://gitee.com/tzcqupt/blog-image/raw/master/img/JMS解释.PNG)

### **区别和联系**

- JMS是定义了统一的接口，来对消息操作进行统一；AMQP是通过规定协议来统一数据交互的格式
- JMS限定了必须使用Java语言；AMQP只是协议，不规定实现方式，因此是跨语言的。
- JMS规定了两种消息模型；而AMQP的消息模型更加丰富
- 基于JMS产品:ActiveMQ/RocketMQ,基于AMQP:RabbitMQ,erlang语言开发,稳定性好

<span id="rabbitmq"></span>

# RabbitMQ

MQ全称为Message Queue，即消息队列， RabbitMQ是由erlang语言开发，基于AMQP（Advanced Message Queue 高级消息队列协议）协议实现的消息队列，它是一种应用程序之间的通信方法，消息队列在分布式系统开 发中应用非常广泛。[RabbitMQ官方地址](http://www.rabbitmq.com/)

## 安装

1. [Erlang/OTP 20.3版本](http://erlang.org/download/otp_win64_20.3.exe)和[RabbitMQ3.7.3版本](https://github.com/rabbitmq/rabbitmq-server/releases/tag/v3.7.3)

2. erlang安装完之后配置环境变量ERLANG_HOME=D:\soft\erl9.3,添加到path中

3. 在rabbitmq的sbin目录安装管理插件`rabbitmq-plugins.bat enable rabbitmq_management`

4. 重新启动服务，访问[http://localhsot:15672](http://localhsot:15672)  guest/guest

   >当卸载重新安装时会出现RabbitMQ服务注册失败，此时需要进入注册表清理erlang
   >搜索RabbitMQ、ErlSrv，将对应的项全部删除。

5. 安装`rabbitmq-delayed-message-exchange `插件，可以更方便的实现延迟队列
   1. 选择对应版本的插件下载。下载地址：https://www.rabbitmq.com/community-plugins.html
   2. 下载完成后，拷贝到RabbitMQ安装目录里的plugins文件夹下。
   3. 使用``rabbitmq-plugins enable rabbitmq_delayed_message_exchange` 开启插件，在使用命令 `rabbitmq-plugins list` 查询安装的所有插件，确认安装是否成功。

## 名词解释

Broker：消息队列服务进程，此进程包括两个部分：Exchange和Queue。 

Exchange：消息队列交换机，按一定的规则将消息路由转发到某个队列，对消息进行过虑。

Queue：消息队列，存储消息的队列，消息到达队列并转发给指定的消费方。 

Producer：消息生产者，即生产方客户端，生产方客户端将消息发送到MQ。

Consumer：消息消费者，即消费方客户端，接收MQ转发的消息。

## 消息发布接收流程

### **发送消息**

1. 生产者和Broker建立TCP连接
2. 生产者和Broker建立通道Channel
3. 生产者通过通道消息发送给Broker,由Exchange将消息转发
4. Exchange将消息转发到指定的Queue(队列)

### **接收消息**

1. 消费者和Broker建立TCP连接
2. 消费者和Broker建立通道Channel

3. 消费者监听指定的Queue
4. 当有消息到达Queue时Broker默认将消息推送给消费者
5. 消费者接收到消息

### **消息确认机制ACK**

自动ACK：消息一旦被接收,消费者自动发送ACK

手动ACK：消息接收后，不会发送ACK，需要手动调用

如何选择：看消息的重要性

## 交换器类型

### fanout

它会把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中，即**无视RoutingKey和BindingKey的匹配规则**。

### direct

它会把消息路由到那些BindingKey和RoutingKey**完全匹配**的队列中。

### topic

对direct的扩展，direct规则是严格意义上的匹配，换言之Routing Key必须与Binding Key相匹配的时候才将消息传送给Queue，那么topic这个规则就是模糊匹配，可以通过通配符满足一部分规则就可以传送。

~~~java
rabbitTemplate.convertAndSend(topicExchange,"bind001.topic.key", goJson.toJSONString());

rabbitTemplate.convertAndSend(topicExchange,"bind002.topic.key", goJson.toJSONString());

BindingBuilder.bind(AMessage).to(topicExchange).with("*.bind.key");

//队列AMessage使用*.bind.key绑定到topicExchange交换机上,这样生产的两条消息都会在消息队列A中
~~~

### headers

根据发送的消息内容中的headers属性进行匹配。

## 工作模式

#### Work queues

1. 一条消息只会被一个消费者接收；
2. rabbit采用轮询的方式将消息是平均发送给消费者的；
3. 消费者在处理完某条消息后，才会收到下一条消息。

### Publish/Subscribe 发布订阅模式

两者区别：

1. work queues不用定义交换机Exchange，Publish/Subscribe需要定义交换机
2. Publish/Subscribe的生产方面是面向交换机发送消息，work queues的生产方面是面向队列发送消息（底层使用默认的交换机）
3. Publish/Subscribe需要设置队列和交换机的绑定，work queues不需要设置（实质上会绑定到默认的交换机）

相同点

实现的发布/订阅效果是一样，多个消费端监听同一个队列不会重复消费消息

1. Routing

2. Topics

3. Header

4. RPC

## 整合SpringBoot

### 添加启动器

~~~xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
~~~

### 在配置文件中添加rabbitMQ地址

~~~yaml
spring:
  rabbitmq:
    host: 127.0.0.1
    username: leyou
    password: leyou
    virtual-host: /leyou
~~~

### 监听者

~~~java
@Component
public class Listener {
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "spring.test.queue", durable = "true"),
            exchange = @Exchange(
                    value = "spring.test.exchange",
                    ignoreDeclarationExceptions = "true",
                    type = ExchangeTypes.TOPIC
            ),
            key = {"#.#"}))
    public void listen(String msg){
        System.out.println("接收到消息：" + msg);
    }
}
~~~

- @Componet：类上的注解，注册到Spring容器

- @RabbitListener：方法上的注解，声明这个方法是一个消费者方法，需要指定下面的属性：

  - bindings：指定绑定关系，可以有多个。值是`@QueueBinding`的数组。`@QueueBinding`包含下面属性：

  - value：这个消费者关联的队列。值是`@Queue`，代表一个队列

    durable表示是否持久化

  - exchange：队列所绑定的交换机，值是`@Exchange`类型

  - key：队列和交换机绑定的`RoutingKey`

### 发送消息

利用Spring提供的`AmqpTemplate`的`convertAndSend(String exchange,String routingKey,Object message)`方法
