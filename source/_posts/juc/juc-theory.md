---
layout: post
title: 深入浅出多线程学习笔记-原理篇
date: 2020-04-28
Author: tzcqupt
tags: [Java, 多线程]
categories: [编程书籍]
comments: true
toc: true
---

# Java内存模型

## Java内存模型的抽象结构

### 运行时内存的划分

对于每一个线程来说，**栈都是私有的，堆是共有的。**

线程**私有**的数据区：**虚拟机栈、本地方法栈、程序计数器**。

所有线程**共享**的数据区：**方法区、堆**。

在**栈**中的变量（局部变量、⽅法定义参数、异常处理器参数）不会在线程 

之间共享。**堆中**的变量是共享的，内存可见性针对的是**共享变量**。

### 堆中的内存不可见问题

JMM ==>Java内存模型

![](..\img\JMM抽象示意图.PNG)

1. 线程**A**⽆法直接访问线程**B**的⼯作内存，线程间通信必须经过主内存。

2. 根据JMM的规定，线程对共享变量的所有操作都必须在⾃⼰的本地内存中进⾏，不能直接从主内存中读取。

   > 先在本地内存中找到共享变量，发现共享变量更新了，就从主内存中拷贝新的值到本地内存，最后从本地内存读取。

3. **JMM**通过控制主内存与每个线程的本地内存之间的交互，来提供内存可⻅性保证。

# 重排序与happens-before

## 重排序

计算机执行程序时，为了提高性能，编译器和处理器对指令做出重排。

指令重排可以保证串⾏语义⼀致，但是没有义务保证多线程间的语义也⼀致。

## 顺序一致性模型和JMM保证

### 数据竞争与顺序一致性

数据竞争：在⼀个线程中写⼀个变量，在另⼀个线程读同⼀个变量，并且写和读没有通过同步来排序。 

JMM保证：如果程序是正确同步的，程序的执⾏将具有顺序⼀致性。

> 正确使用`volatile`、`final`、`synchronized`等关键字来实现**多线程下的同步**。

## happens-before

程序员只要遵循happens-before规则，那他写的程序就能保证在JMM中具有强的内存可见性。

如果操作A happens-before操作B，那么操作A在内存上所做的操作对操作B都是可见的，不管它们在不在一个线程。

### 天然的happens-before

- 程序顺序规则：一个线程中的每一个操作，happens-before于该线程中的任意后续操作。

- 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。

- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。

- 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。

- start规则：如果线程A执行操作ThreadB.start()启动线程B，那么A线程的ThreadB.start（）操作happens-before于线程B中的任意操作、

- join规则：如果线程A执行操作ThreadB.join（）并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。

# volatitle

## 相关概念回顾

### 内存可见性

**线程之间的可见性，当一个线程修改了共享变量时，另一个线程可以读取到这个修改后的值**。

### 重排序

为优化程序性能，对原有的指令执行顺序进行优化重新排序。重排序可能发生在多个阶段，比如编译重排序、CPU重排序等。

### happens-before规则

是一个给程序员使用的规则，只要程序员在写代码的时候遵循happens-before规则，JVM就能保证指令在多线程之间的顺序性符合程序员的预期。

## volatile的内存语义

1. 保证变量的**内存可见性**
2. 禁止volatile变量与普通变量**重排序**。

### **JMM**内存屏障插⼊策略

- 在每个volatile写操作前插⼊⼀个StoreStore屏障； 

- 在每个volatile写操作后插⼊⼀个StoreLoad屏障； 

- 在每个volatile读操作后插⼊⼀个LoadLoad屏障； 

- 在每个volatile读操作后再插⼊⼀个LoadStore屏障。

### volatile与普通变量的重排序规则

1. 如果第⼀个操作是volatile读，那⽆论第⼆个操作是什么，都不能重排序； 

2. 如果第⼆个操作是volatile写，那⽆论第⼀个操作是什么，都不能重排序； 

3. 如果第⼀个操作是volatile写，第⼆个操作是volatile读，那不能重排序。 

> 第一个操作是普通变量读，第二个操作是volatile变量读，可以重排序

## volatile用途

功能上锁比volatile强大，性能上，volatile更有优势。

# synchronzied与锁

**Java**多线程的锁都是基于对象的，Java中的每⼀个对象都可以作为⼀个锁。 

## synchronzied关键字

### 3种形式

~~~java
// 关键字在实例方法上，锁为当前实例
public synchronized void instanceLock() {
    // code
}

// 关键字在静态方法上，锁为当前Class对象
public static synchronized void classLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象
public void blockLock() {
    Object o = new Object();
    synchronized (o) {
        // code
    }
}
~~~

### 临界区

指的是某一块代码区域，它同一时刻只能由一个线程执行。

### 等价的写法

#### 锁为当前实例

~~~java
// 关键字在实例方法上，锁为当前实例
public synchronized void instanceLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象，也就是当前实例
public void blockLock() {
    synchronized (this) {
        // code
    }
}
~~~

#### 锁为Class对象

~~~java
// 关键字在静态方法上，锁为当前Class对象
public static synchronized void classLock() {
    // code
}

// 关键字在代码块上，锁为括号里面的对象，为Class对象
public void blockLock() {
    synchronized (this.getClass()) {
        // code
    }
}
~~~

## 几种锁

Java 6 为了减少获得锁和释放锁带来的性能消耗，引入了“偏向锁”和“轻量级锁“。在Java 6 以前，所有的锁都是”重量级“锁。

Java 6后，一个对象有4种锁状态。

无锁状态、偏向锁状态、轻量级锁状态、重量级锁状态。

锁降级情况，参考JVM学习。锁降级发生在Stop The World期间，当JVM进入安全点的时候，会检查是否有闲置的锁，然后进行降级。

### Java对象头

每个Java对象都有对象头。如果是非数组类型，则用2个字宽来存储对象头，如果是数组，则会用3个字宽来存储对象头。在32位处理器中，一个字宽是32位；在64位虚拟机中，一个字宽是64位。

| 长度     | 内容                   | 说明                         |
| -------- | ---------------------- | ---------------------------- |
| 32/64bit | Mark Word              | 存储对象的hashCode或锁信息等 |
| 32/64bit | Class Metadata Address | 存储到对象类型数据的指针     |
| 32/64bit | Array length           | 数组的长度（如果是数组）     |

当对象状态为偏向锁时，`Mark Word`存储的是偏向的线程ID；当状态为轻量级锁时，`Mark Word`存储的是指向线程栈中`Lock Record`的指针；当状态为重量级锁时，`Mark Word`为指向堆中的monitor对象的指针。

### 偏向锁

### 轻量级锁

### 重量级锁

每一个对象都可以当做一个锁，当多个线程同时请求某个对象锁时，对象锁会设置几种状态用来区分请求的线程

~~~
Contention List：所有请求锁的线程将被首先放置到该竞争队列
Entry List：Contention List中那些有资格成为候选人的线程被移到Entry List
Wait Set：那些调用wait方法被阻塞的线程被放置到Wait Set
OnDeck：任何时刻最多只能有一个线程正在竞争锁，该线程称为OnDeck
Owner：获得锁的线程称为Owner
!Owner：释放锁的线程
~~~

当一个线程尝试获得锁时，如果该锁已经被占用，则会将该线程封装成一个`ObjectWaiter`对象插入到Contention List的队列的队首，然后调用`park`函数挂起当前线程。

当线程释放锁时，会从Contention List或EntryList中挑选一个线程唤醒，被选中的线程叫做`Heir presumptive`即假定继承人，假定继承人被唤醒后会尝试获得锁，但`synchronized`是非公平的，所以假定继承人不一定能获得锁。这是因为对于重量级锁，线程先自旋尝试获得锁，这样做的目的是为了减少执行操作系统同步操作带来的开销。如果自旋不成功再进入等待队列。这对那些已经在等待队列中的线程来说，稍微显得不公平，还有一个不公平的地方是自旋线程可能会抢占了Ready线程的锁。

果线程获得锁后调用`Object.wait`方法，则会将线程加入到WaitSet中，当被`Object.notify`唤醒后，会将线程从WaitSet移动到Contention List或EntryList中去。需要注意的是，当调用一个锁对象的`wait`或`notify`方法时，**如当前锁的状态是偏向锁或轻量级锁则会先膨胀成重量级锁**。

### 锁的升级流程

每一个线程在准备获取共享资源时。

1. 检查MarkWord里面是不是放的自己的ThreadId ,如果是，表示当前线程是处于 “偏向锁” 。

2. 如果MarkWord不是自己的ThreadId，锁升级，这时候，用CAS来执行切换，新的线程根据MarkWord里面现有的ThreadId，通知之前线程暂停，之前线程将Markword的内容置为空。

3. 两个线程都把锁对象的HashCode复制到自己新建的用于存储锁的记录空间，接着开始通过CAS操作， 把锁对象的MarKword的内容修改为自己新建的记录空间的地址的方式竞争MarkWord。

4. 第三步中成功执行CAS的获得资源，失败的则进入自旋 。

5. 自旋的线程在自旋过程中，成功获得资源(即之前获的资源的线程执行完成并释放了共享资源)，则整个状态依然处于 轻量级锁的状态，如果自旋失败 。

6. 进入重量级锁的状态，这个时候，自旋的线程进行阻塞，等待之前线程执行完成并唤醒自己。

# CAS与原子操作

## 乐观锁和悲观锁

### 悲观锁

对于悲观锁来说，它总是认为每次访问共享资源时会发生冲突，所以必须对每次数据操作加上锁，以保证临界区的程序同一时间只能有一个线程在执行。

多用于”写多读少“的环境，避免频繁失败和重试影响性能。

### 乐观锁

观锁总是假设对共享资源的访问没有冲突，线程可以不停地执行，无需加锁也无需等待。而一旦多个线程发生冲突，乐观锁通常是使用一种称为CAS的技术来保证线程执行的安全性。

没有锁存在，不可能出现死锁的情况。

多用于“读多写少“的环境，避免频繁加锁影响性能。

## CAS

CAS全称：比较并交换（Compare And Swap）。

CAS种有3个值：V：要更新的变量(var)、E：预期值(expected)、N：新值(new)。

比较并交换过程如下：

判断V是否等于E，如果等于，将V的值设置为N；如果不等，说明已经有其它线程更新了V，则当前线程放弃更新，什么都不做。

> **预期值E本质上指的是“旧值”**。

**CAS是一种原子操作**，它是一种系统原语，是一条CPU的原子指令，从CPU层面保证它的原子性。

>当多个线程同时使用CAS操作一个变量时，只有一个会胜出，并成功更新，其余均会失败，但失败的线程并不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。

### ABA问题

一个值原来是A，变成了B，又变回了A。

解决思路：在变量前面追加**版本号或者时间戳**。JDK1.5开始，atomic包里提供了一个类`AtomicStampedReference`类来解决ABA问题。

这个类的`compareAndSet`方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果二者都相等，才使用CAS设置为新的值和标志。

### 循环时间长开销大

CAS多与自旋结合。如果自旋CAS长时间不成功，会占用大量的CPU资源。

解决思路：让JVM支持处理器提供的**pause指令**。

pause指令能让自旋失败时cpu睡眠一小段时间再继续自旋，从而使得读操作的频率低很多,为解决内存顺序冲突而导致的CPU流水线重排的代价也会小很多。

### 只能保证一个共享变量的原子操作

1. 使用JDK 1.5开始就提供的`AtomicReference`类保证对象之间的原子性，把多个变量放到一个对象里面进行CAS操作；
2. 使用锁。锁内的临界区代码可以保证只有当前线程能操作。

# AQS

## AQS简介

**AQS**是`AbstractQueuedSynchronizer`的简称，即`抽象队列同步器`。

- 抽象：抽象类，只实现一些主要逻辑，有些方法由子类实现；

- 队列：使用先进先出（FIFO）队列存储数据；

- 同步：实现了同步的功能。

AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的同步器，比如我们提到的ReentrantLock，Semaphore，ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。

## AQS数据结构

AQS内部使用了一个volatile的变量state来作为资源的标识。同时定义了几个获取和改变state的protected方法，子类可以覆盖这些方法来实现自己的逻辑：`getState()、setState()、compareAndSetState()`

AQS类本身实现的是一些排队和阻塞的机制，比如具体线程等待队列的维护（如获取资源失败入队/唤醒出队等）。它内部使用了一个先进先出（FIFO）的双端队列，并使用了两个指针head和tail用于标识队列的头部和尾部。

## 资源共享模式

- 独占模式（Exclusive）：资源是独占的，一次只能一个线程获取。如`ReentrantLock`。

- 共享模式（Share）：同时可以被多个线程获取，具体的资源个数可以通过参数指定。如`Semaphore/CountDownLatch`。

同时实现两种模式的同步类，如`ReadWriteLock`。

AQS源码定义:

~~~java
static final class Node {
    // 标记一个结点（对应的线程）在共享模式下等待
    static final Node SHARED = new Node();
    // 标记一个结点（对应的线程）在独占模式下等待
    static final Node EXCLUSIVE = null; 

    // waitStatus的值，表示该结点（对应的线程）已被取消
    static final int CANCELLED = 1; 
    // waitStatus的值，表示后继结点（对应的线程）需要被唤醒
    static final int SIGNAL = -1;
    // waitStatus的值，表示该结点（对应的线程）在等待某一条件
    static final int CONDITION = -2;
    /*waitStatus的值，表示有资源可用，新head结点需要继续唤醒后继结点（共享模式下，多线程并发释放资源，而head唤醒其后继结点后，需要把多出来的资源留给后面的结点；设置新的head结点时，会继续唤醒其后继结点）*/
    static final int PROPAGATE = -3;

    // 等待状态，取值范围，-3，-2，-1，0，1
    volatile int waitStatus;
    volatile Node prev; // 前驱结点
    volatile Node next; // 后继结点
    volatile Thread thread; // 结点对应的线程
    Node nextWaiter; // 等待队列里下一个等待条件的结点


    // 判断共享模式的方法
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    Node(Thread thread, Node mode) {     // Used by addWaiter
        this.nextWaiter = mode;
        this.thread = thread;
    }

}

// AQS里面的addWaiter私有方法
private Node addWaiter(Node mode) {
    // 使用了Node的这个构造函数
    Node node = new Node(Thread.currentThread(), mode);
    // 其它代码省略
}
~~~

