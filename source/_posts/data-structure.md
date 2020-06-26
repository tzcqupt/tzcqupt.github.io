---
title: 数据结构学习笔记
Author: tzcqupt
categories: [数据结构]
tags: [结构]
comments: true
toc: true
date: 2020-06-20 00:00:00
---

# 概论

数据结构包括:线性结构和非线性结构.

## 线性结构

### 特点

数据元素之间存在一对一的线性关系

### 存储结构

#### 顺序存储结构

顺序存储的线性表称为顺序表,顺序表中的存储元素是连续的.

#### 链式存储结构

链式存储的线性表称为链表，链表中的存储元素不一定是连续的，元素节点中存放数据元素以及相邻元素的地址信息.

### 常见的线性结构

数组、队列、链表和栈

## 非线性结构

### 常见的非线性结构

二维数组，多维数组，广义表，树结构，图结构

# 数组

## 稀疏数组

当一个数组中大部分元素为０，或者为同一个值的数组时，可以使用稀疏数组来保存该数组。

### 处理方法

1. 记录数组一共有几行几列，有多少个不同的值
2. 把具有不同值的元素的行列及值记录在一个小规模的数组中，从而缩小程序的规模

### 举例说明

![](http://img.tzcqupt.top/img/20200620234609.png)

### 代码实现

```java
public class SparseArray {
  public static void main(String[] args) {
    // 原始数组
    int[][] originArray = new int[10][11];
    originArray[0][2] = 1;
    originArray[1][3] = 2;
    originArray[2][4] = 1;
    printArray(originArray);
    // 0. 得到原始数组总不为0的元素的个数sum
    int sum = getArrayValueCount(originArray, 0);
    // 1. 稀疏数组定义--sum+1,列始终为3
    int[][] sparseArray = new int[sum + 1][3];

    // 2. 给稀疏数组赋值
    sparseArray[0][0] = originArray.length;
    sparseArray[0][1] = originArray[0].length;
    sparseArray[0][2] = sum;

    sparseArray[1][0] = 0;
    sparseArray[1][1] = 2;
    sparseArray[1][2] = 1;

    sparseArray[2][0] = 1;
    sparseArray[2][1] = 3;
    sparseArray[2][2] = 2;

    sparseArray[3][0] = 2;
    sparseArray[3][1] = 4;
    sparseArray[3][2] = 1;

    System.out.println("打印转换后的稀疏数组");

    printArray(sparseArray);

    // 将稀疏数组转换为原始数组
    // 1. 得到稀疏数组的sparseArray[0][0]和sparseArray[0][1]的值初始化数组
    int[][] targetArray = new int[sparseArray[0][0]][sparseArray[0][1]];
    // 2. 给原始数组赋值
    for (int i = 1; i < sparseArray.length; i++) {
      for (int j = 0; j < sparseArray[0].length - 2; j++) {
        targetArray[sparseArray[i][j]][sparseArray[i][j + 1]] = sparseArray[i][j + 2];
      }
    }
    System.out.println("打印转换后的原始数组");
    printArray(targetArray);
  }

  /**
   * 计算某个二维数组中不为某个值的元素个数
   *
   * @param arrays 二维数组
   * @return 有效元素个数
   */
  private static int getArrayValueCount(int[][] arrays, int target) {
    int count = 0;
    for (int i = 0; i < arrays.length; i++) {
      for (int j = 0; j < arrays.length; j++) {
        if (arrays[i][j] != target) {
          count++;
        }
      }
    }
    return count;
  }
  /**
   * 打印二维数组
   *
   * @param arrays 数组
   */
  private static void printArray(int[][] arrays) {
    for (int i = 0; i < arrays.length; i++) {
      for (int j = 0; j < arrays[0].length; j++) {
        System.out.print(arrays[i][j] + "\t");
      }
      System.out.println();
    }
  }
}
```

## 自定义数组

1. 数组中的元素支持泛型

2.  数组自动扩容/缩容操作

3. 新增/查找/修改/删除元素

   [代码查看](https://gitee.com/tzcqupt/JavaLearn/blob/develop/BaseSkill/src/main/java/com/tang/base/datastructure/array/Array.java)

# 链表

## 相关操作

### 链表中间插入元素

![](http://img.tzcqupt.top/img/20200620232644.png)

### 链表元素的删除

![](http://img.tzcqupt.top/img/20200620233236.png)

# 栈

特点:先进后出

# 队列

## 常规队列

特点:先进先出

## 循环队列

![循环队列](http://img.tzcqupt.top//img/20200620232205.png)
