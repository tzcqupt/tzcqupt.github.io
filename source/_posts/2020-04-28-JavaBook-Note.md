---
layout: post
title: 阿里开发手册阅读笔记
date: 2020-04-28
Author: tzcqupt
tags: [编码规范]
comments: true
top: True
toc: true
---
# 编程规约

## 基础约定

1. 定义数组类型和中括号紧挨 `int[] arrayDemo;`
2. 避免在子父类的成员变量之间、或者不同代码块的局部变量之间采用完全相同的命名，使可读性降低。
3. 在 long 或者 Long 赋值时，数值后使用大写的 L
4. 推荐使用 `java.util.Objects#equals` 代替Object的equals方法，避免空指针异常。

5. 所有**整型包装类对象**之间值的比较，全部使用 equals 方法比较。

6. **浮点数**之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用 equals来判断。

   1. 基本数据 指定误差范围

      ~~~java
      float a = 1.0f - 0.9f;
      float b = 0.9f - 0.8f;
      float diff = 1e-6f;
      if (Math.abs(a - b) < diff) {
       System.out.println("true");
      }
      ~~~

   2. 包装数据 用BigDecimal定义

      ~~~java
      BigDecimal a = new BigDecimal("1.0");
      BigDecimal b = new BigDecimal("0.9");
      BigDecimal c = new BigDecimal("0.8");
      BigDecimal x = a.subtract(b);
      BigDecimal y = b.subtract(c);
      if (x.equals(y)) {
       System.out.println("true");
      }
      ~~~

7. 使用BigDecimal的入参为String的构造方法新建BigDecimal对象。

8. POJO类属性使用包装数据类型。

9. 序列化类新增属性时，不修改serialVersionUID字段。

10. POJO类需要写toString方法，如果继承了另外的类，加上`super.toString()`

11. 使用索引访问用 String 的 split 方法得到的数组时，需做最后一个分隔符后有无内容.

    的检查，否则会有抛 IndexOutOfBoundsException 的风险。

12. 循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展。

13. 在 JDK8 中，针对统计时间等场景，推荐使用 Instant 类。

14. 获取某年的天数

    ~~~java
    // 获取今年的天数
    int daysOfThisYear = LocalDate.now().lengthOfYear();
    // 获取指定某年的天数
    LocalDate.of(2011, 1, 1).lengthOfYear();
    ~~~

## 集合处理

1. 重写equals就重写hashCode
2. 判断集合是否为空，使用isEmpty()方法

3. ArrayList 的 subList 结果不可强转成 ArrayList

4. 使用 Map 的方法 keySet()/values()/entrySet()返回集合对象时，不可以对其进行添加元素操作，否则会抛出 UnsupportedOperationException 异常。

5. Collections 类返回的对象，如：emptyList()/singletonList()等都是 immutable list，不可对其进行添加或者删除元素的操作。

6. 使用集合转数组的方法，必须使用集合的 toArray(T[] array)，传入的是类型完全一致、长度为 0 的空数组。

7. 在使用 Collection 接口任何实现类的 addAll()方法时，都要对输入的集合参数进行NPE（NullPointerException） 判断。

8. 使用工具类 Arrays.asList()把数组转换成集合时，不能使用其修改集合相关的方法，它的 add/remove/clear 方法会抛出 UnsupportedOperationException 异常。

9. 不要在 foreach 循环里进行元素的 remove/add 操作。remove 元素请使用 Iterator方式，如果并发操作，需要对 Iterator 对象加锁。

## 并发

1. 创建线程或线程池时请指定有意义的线程名称。

2. 线程资源必须通过线程池提供

3. 线程池不允许使用 Executors 去创建，而是通过 ThreadPoolExecutor 的方式

4. SimpleDateFormat 是线程不安全的类，一般不要定义为 static 变量，如果定义为 static，必须加锁，或者使用 DateUtils 工具类。

   > 如果是 JDK8 的应用，可以使用 Instant 代替 Date，LocalDateTime 代替 Calendar，DateTimeFormatter(线程安全)代替 SimpleDateFormat

5. 必须回收自定义的 ThreadLocal 变量

6. 获取/释放锁的方式

   ~~~java
   Lock lock = new XxxLock();
   // ...
   lock.lock();
   try {
    doSomething();
    doOthers();
   } finally {
    lock.unlock();
   }
   ~~~

7. JDK7后使用ThreadLocalRandom 提高性能