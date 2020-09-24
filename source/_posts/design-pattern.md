---
title: 设计模式学习比较
Author: tzcqupt
categories: [设计模式]
tags: [设计模式]
comments: true
toc: true
date: 2020-08-28 00:00:00
---

# 概论

## 类与类的交互关系

1. 泛化

   ​	继承关系

2. 实现

   ​	接口和实现类

3. 聚合

   包含关系,A类对象包含B类对象,B类对象的生命周期可以不依赖A类对象的生命周期,如课程和学生之间的关系

   ~~~java
   public class A{
   private B b;
       public A(B b){
           this.b=b;
       }
   }
   ~~~

4. 组合

   包含关系,A类对象包含B类对象,B类对象的生命周期依赖A类对象的生命周期,B类对象不能单独存在,如鸟和翅膀的关系.

   ~~~java
   public class A{
   private B b;
       public A(){
           this.b=new B();
       }
   }
   ~~~

   