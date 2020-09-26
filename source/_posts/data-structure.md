---
title: 数据结构和算法学习笔记
Author: tzcqupt
categories: [数据结构,算法]
tags: [数据结构]
comments: true
toc: true
date: 2020-06-20 00:00:00
---

# 概论

## 时间复杂度

时间复杂度的全称是**渐进时间复杂度**，**表示算法的执行时间与数据规模之间的增长关**

**系**。

### O(1)

一般情况下,只要算法中不存在循环/递归语句,代码再长,时间复杂度也是O(1)

~~~java
int i = 8;  
int j = 6;
int sum = i + j;
~~~

### O(logn)/O(nlogn)

~~~java
 i=1;
 while (i <= n) {   
     i = i * 2;
 }
~~~

变量i的取值为一个等比数列

![等比数列](http://img.tzcqupt.top/等比数列.jpg)

时间复杂度为O(logn) ,底数被忽略

如果⼀段代码的时间复杂度是O(logn)，我们循环执行n遍，时间复杂度就是O(nlogn)了.

如归并排序/快速排序的时间复杂度都是O(nlogn)

### **O(m+n)、O(m\*n)**

代码中有两个不同的数据规模,m和n

## 空间复杂度

时间复杂度的全称是**渐进时间复杂度**，**表示算法的执行时间与数据规模之间的增长关**

**系**。



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

# 数据结构

## 数组

### 稀疏数组

当一个数组中大部分元素为０，或者为同一个值的数组时，可以使用稀疏数组来保存该数组。

#### 处理方法

1. 记录数组一共有几行几列，有多少个不同的值
2. 把具有不同值的元素的行列及值记录在一个小规模的数组中，从而缩小程序的规模

#### 举例说明

![](http://img.tzcqupt.top/img/20200620234609.png)

#### 代码实现

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

### 自定义数组

1. 数组中的元素支持泛型

2.  数组自动扩容/缩容操作

3. 新增/查找/修改/删除元素

   [代码查看](https://gitee.com/tzcqupt/JavaLearn/blob/develop/BaseSkill/src/main/java/com/tang/base/datastructure/array/Array.java)

## 链表

### 相关操作

#### 链表中间插入元素

![](http://img.tzcqupt.top/img/20200620232644.png)

#### 链表元素的删除

![](http://img.tzcqupt.top/img/20200620233236.png)

## 栈

特点:先进后出

入栈 push()

出栈 pop()

## 队列

入队enqueue()

出队dequeue()

### 常规队列

特点:先进先出

### 循环队列

![循环队列](http://img.tzcqupt.top/loop-queue.png)

# 算法

## 经典排序算法

### 冒泡排序

#### 解释

是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。

#### 算法描述

1. **`i`从0开始，`i`与`i+1`比较，如果`i>i+1`，那么就互换** 
2. **`i`不断增加，直到`i（n是数组元素的个数，n-1`是数组已经最后一个元素） ，一趟下来，可以让数组元素中最大值排在数组的最后面**

#### 图解

![](http://img.tzcqupt.top//bubble-sort.gif)

#### 算法实现

~~~java

  public void bubbleSort(int[] array) {
    //外层循环是排序的趟数
    for (int i = 0; i < array.length; i++) {
      //内层循环是当前趟数需要比较的次数
      for (int j = 0; j < array.length - 1 - i; j++) {
        if (array[j + 1] < array[j]) {
          // 交换,将大的放在后面
          int temp = array[j + 1];
          array[j + 1] = array[j];
          array[j] = temp;
     
        }
      }
    }

  }
~~~

### 选择排序

#### 解释

它的工作原理是每一次从待排序的数据元素中选出最小(或最大)的一个元素，存放在序列的起始(末尾)位置，直到全部待排序的数据元素排完。选择排序是不稳定的排序方法（比如序列[5， 5， 3]第一次就将第一个[5]与[3]交换，导致第一个5挪动到第二个5后面）。

稳定排序的好处

> 如果我们只对一串数字排序，那么稳定与否确实不重要，因为一串数字的属性是单一的，就是数字值的大小。但是排序的元素往往不只有一个属性，例如我们对一群人按年龄排序，但是人除了年龄属性还有身高体重属性，在年龄相同时如果不想破坏原先身高体重的次序，就必须用稳定排序算法.

#### 图解

![](http://img.tzcqupt.top//select-sort.gif)

#### 算法实现

~~~java
public void chooseSort(int[] array){
    //记录当前趟数的最大值的角标
    int pos ;
    // 临时值
    int temp;
    for (int i = 0; i < array.length; i++) {
      // 新的一趟
      pos= 0;
      for (int j = 0; j < array.length-i; j++) {
        if (array[j]>array[pos]){
          pos=j;
        }
      }
      //交换
      temp = array[pos];
      array[pos] = array[array.length - 1 - i];
      array[array.length - 1 - i] = temp;
    }
  }
~~~

### 插入排序

#### 解释

插入排序的基本操作就是将一个数据插入到已经排好序的有序数据中，从而得到一个新的、个数加一的有序数据，算法适用于少量数据的排序，时间复杂度为O(n^2)。是稳定的排序方法。

#### 算法描述

不知道数组元素的情况下,**把数组的第一个元素作为已经排好序的有序数据**

#### 图解

![](http://img.tzcqupt.top//insert-sort.gif)

#### 算法实现

~~~java
/** 插入排序 将一个数据插入到已经排好序的有序数据中 */
  public void insertSort() {
    int temp;
    for (int i = 0; i < array.length; i++) {
      temp = array[i];
      int j = i - 1;
      // 如果前一位(已排序的数据)比当前数据要大
      while (j >= 0 && array[j] > temp) {
        // 交换当前数据和前一位数据
        array[j + 1] = array[j];
        // 继续循环
        j--;
      }
      // 前者循环是j--,如果不满足while条件,则j+1为该元素的正确位置
      array[j + 1] = temp;
    }
  }
~~~

### 快速排序

#### 解释

通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以**递归**进行，以此达到整个数据变成有序序列。

#### 算法解释

在数组中找一个支点(任意),经过一趟排序后，支点左边的数都要比支点小，支点右边的数都要比支点大！

#### 图解

![](http://img.tzcqupt.top/quick-sort.gif)

#### 算法实现

~~~java
 public void quickSortImpl(int[] arr,int left,int right){
    int i=left;
    int j= right;
    // 支点
    int pivot = arr[(i+j)/2];
    // 左右两边进行扫描,只要没交替,就一直扫描
    while (i<=j){
      //寻找直到比支点大的数
      while (pivot > arr[i]){
        i++;
      }

      //寻找直到比支点小的数
      while (pivot < arr[j]){
        j--;
      }
      //此时已经分别找到了比支点小的数(右边)、比支点大的数(左边)，它们进行交换
      if (i <= j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
        i++;
        j--;
      }
      //“左边”再做排序，直到左边剩下一个数(递归出口)
      if (left < j){
        quickSortImpl(arr, left, j);
      }
      //“右边”再做排序，直到右边剩下一个数(递归出口)
      if (i < right){
        quickSortImpl(arr, i, right);
      }
    }
  }
~~~

