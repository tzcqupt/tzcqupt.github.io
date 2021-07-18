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

![等比数列](http://file.tzcqupt.top/等比数列.jpg)

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

![](http://file.tzcqupt.top/img/20200620234609.png)

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

![](http://file.tzcqupt.top/img/20200620232644.png)

#### 链表元素的删除

![](http://file.tzcqupt.top/img/20200620233236.png)

### 单链表

 每个节点中有持有指向下一个节点的指针，只能按照一个方向遍历链表 

~~~java
class SingleNode{
  private Object data;//存储数据
  private SingleNode nextNode;//指向下一个节点
} 
~~~

### 双向链表

 每个节点中两个指针，分别指向当前节点的上一个节点和下一个节点 

~~~java
class DoubleNode{
  private Object data;//存储数据
  private SingleNode prevNode;//指向上一个节点
  private SingleNode nextNode;//指向下一个节点
} 
~~~

### 链表的优点

1. 可以快速定位到上一个或者下一个节点
2. 可以快速删除数据，只需改变指针的指向即可，这点比数组好

### 链表的缺点

1. 无法向数组那样，通过下标随机访问数据
2. 查找数据需从第一个节点开始遍历，不利于数据的查找，查找时间和无需数据类似，需要全遍历，最差时间是O(N)

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

![循环队列](http://file.tzcqupt.top/loop-queue.png)

## 二叉树

### 二叉查找树

 二叉树是每个结点最多有两个子树的树结构，通常子树被称作“左子树”（left subtree）和“右子树”（right subtree）。 

#### 特点

1. 每个结点都包含一个元素以及n个子树，这里0≤n≤2。
2. 左子树和右子树是有顺序的，次序不能任意颠倒，左子树的值要**小于**父结点，右子树的值要**大于**父结点。 

#### 缺点

1. 查询数据的效率不稳定，若树左右比较平衡的时，最差情况为O(logN)，如果插入数据是有序的，退化为了链表，查询时间变成了O(N)

2. 数据量大的情况下，会导致树的高度变高，如果每个节点对应磁盘的一个块来存储一条数据，需IO次数大幅增加，显然用此结构来存储数据是不可取的

### 平衡二叉树(AVL树)

 它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。 

### B-树

`B杠树`, B-树节点中可以放多个元素，主要是为了降低树的高度。 

#### 特点

1. 每个节点最多有m个孩子，m称为b树的阶
2. 除了根节点和叶子节点外，其它每个节点至少有Ceil(m/2)个孩子
3. 若根节点不是叶子节点，则至少有2个孩子
4. 所有叶子节点都在同一层，且不包含其它关键字信息
5. 每个非终端节点包含n个关键字（健值）信息
6. 关键字的个数n满足：ceil(m/2)-1 <= n <= m-1
7. ki(i=1,…n)为关键字，且关键字升序排序
8. Pi(i=1,…n)为指向子树根节点的指针。P(i-1)指向的子树的所有节点关键字均小于ki，但都大于k(i-1)

#### 缺点

不利于范围查找

### B+树

#### 特点

1. 每个结点至多有m个子女
2. 除根结点外,每个结点至少有[m/2]个子女，根结点至少有两个子女
3. 有k个子女的结点必有k个关键字
4. 父节点中持有访问子节点的指针
5. 父节点的关键字在子节点中都存在（如上面的1/20/35在每层都存在），要么是最小值，要么是最大值，如果节点中关键字是升序的方式，父节点的关键字是子节点的最小值
6. 最底层的节点是叶子节点
7. 除叶子节点之外，其他节点不保存数据，只保存关键字和指针
8. 叶子节点包含了所有数据的关键字以及data，叶子节点之间用链表连接起来，可以非常方便的支持范围查找

### B+树和B-树总结

#### 不同点

1. b+树中一个节点如果有k个关键字，最多可以包含k个子节点（k个关键字对应k个指针）；而b-树对应k+1个子节点（多了一个指向子节点的指针）
2. b+树除叶子节点之外其他节点值存储关键字和指向子节点的指针，而b-树还存储了数据，这样同样大小情况下，b+树可以存储更多的关键字
3. b+树叶子节点中存储了所有关键字及data，并且多个节点用链表连接，从上图中看子节点中数据从左向右是有序的，这样快速可以支撑范围查找（先定位范围的最大值和最小值，然后子节点中依靠链表遍历范围数据）

#### 如何选择

1. B-Tree因为非叶子结点也保存具体数据，所以在查找某个关键字的时候找到即可返回。而B+Tree所有的数据都在叶子结点，每次查找都得到叶子结点。所以在同样高度的B-Tree和B+Tree中，B-Tree查找某个关键字的效率更高。
2. 由于B+Tree所有的数据都在叶子结点，并且结点之间有指针连接，在找大于某个关键字或者小于某个关键字的数据的时候，B+Tree只需要找到该关键字然后沿着链表遍历就可以了，而B-Tree还需要遍历该关键字结点的根结点去搜索。
3. 由于B-Tree的每个结点（这里的结点可以理解为一个数据页）都存储主键+实际数据，而B+Tree非叶子结点只存储关键字信息，而每个页的大小有限是有限的，所以同一页能存储的B-Tree的数据会比B+Tree存储的更少。这样同样总量的数据，B-Tree的深度会更大，增大查询时的磁盘I/O次数，进而影响查询效率

# 算法

## 经典排序算法

### 冒泡排序

#### 解释

是一种简单的排序算法。它重复地走访过要排序的数列，一次比较两个元素，如果他们的顺序错误就把他们交换过来。走访数列的工作是重复地进行直到没有再需要交换，也就是说该数列已经排序完成。

#### 算法描述

1. **`i`从0开始，`i`与`i+1`比较，如果`i>i+1`，那么就互换** 
2. **`i`不断增加，直到`i（n是数组元素的个数，n-1`是数组已经最后一个元素） ，一趟下来，可以让数组元素中最大值排在数组的最后面**

#### 图解

![](http://file.tzcqupt.top//bubble-sort.gif)

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

![](http://file.tzcqupt.top//select-sort.gif)

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

![](http://file.tzcqupt.top//insert-sort.gif)

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

![](http://file.tzcqupt.top/quick-sort.gif)

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

