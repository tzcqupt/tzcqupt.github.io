---
title: Kafka学习
Author: tzcqupt
categories: [Kafka]
tags: [Kafka]
comments: true
toc: true
date: 2022-02-13 00:00:00
---

# Kafka的安装

1. 安装jdk8 `sudo apt install openjdk-8-jdk-headless`

2. zookeeper安装&启动

   1. 解压后得到 `apache-zookeeper-3.5.7-bin` 文件夹,进入`conf`目录,拷贝`zoo_sample.cfg`为`zoo.cfg`

   2. 编辑zoo.cfg

      ~~~
      dataDir=/home/tzcqupt/logs/zookeeper
      clientPort=2181
      ~~~

   3. 进入bin目录,执行`./zkServer.sh start`开启zookeeper服务

3. 安装kafka

   1. 进入解压后的目录`kafka_2.11-2.4.0`的config目录,编辑`server.properties`文件

      ~~~
      #替换为本机ip
      listeners=PLAINTEXT://192.168.50.28:9092
      advertised.listeners=PLAINTEXT://192.168.50.28:9092
      log.dirs=/home/tzcqupt/logs/kafka
      zookeeper.connect=localhost:2181
      ~~~

   2. 启动Kafka

      ~~~bash
      bin/kafka-server-start.sh config/server.properties &
      ~~~

   3. 停止Kafka

      ~~~
      bin/kafka-server-stop.sh
      ~~~

      

4. kafka常见命令

   1. 创建topic

      ~~~
      bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic tzcqupt-topic
      ~~~

   2. 查看已经创建的Topic信息

      ~~~
      bin/kafka-topics.sh --list --zookeeper localhost:2181
      ~~~

   3. 发送消息

      ~~~
      bin/kafka-console-producer.sh --broker-list 192.168.50.28:9092 --topic tzcqupt-topic
      ~~~

   4. 接收消息

      ~~~
      bin/kafka-console-consumer.sh --bootstrap-server 192.168.50.28:9092 --topic tzcqupt-topic --from-beginning
      ~~~

      

