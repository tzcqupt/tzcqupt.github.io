layout: post
title: 多线程学习笔记-JDK工具
Author: tzcqupt
tags:
  - Java
  - 多线程
comments: true
toc: true
date: 2020-04-28 00:00:00
---
# 线程池原理

Java中的线程池顶层接口是`Executor`接口，`ThreadPoolExecutor`是这个接口的实现类。

线程池中有两类线程：核心线程和非核心线程。

核心线程一直存在，闲置不会被销毁。非核心线程长时间闲置会被销毁。

## `ThreadPoolExecutor`

### 构造方法中相关参数

- **int corePoolSize**：该线程池中**核心线程数最大值**.

- **int maximumPoolSize**：该线程池中**线程总数最大值** 。

- **long keepAliveTime**：**非核心线程闲置超时时长**。

- **TimeUnit unit**：keepAliveTime的单位。枚举类，包括`NANOSECONDS `,`MICROSECONDS` 、`MILLISECONDS` 、`SECONDS` 、`MINUTES` 、`MINUTES` 、`DAYS`。

- **BlockingQueue workQueue**：阻塞队列，维护着**等待执行的Runnable任务对象**。

  - **LinkedBlockingQueue**

    链式阻塞队列，底层数据结构是链表，默认大小是`Integer.MAX_VALUE`，也可以指定大小。

  - **ArrayBlockingQueue**

    数组阻塞队列，底层数据结构是数组，需要指定队列的大小。

  - **SynchronousQueue**

    同步队列，内部容量为0，每个put操作必须等待一个take操作，反之亦然。

  - **DelayQueue**

    延迟队列，该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。

- **ThreadFactory threadFactory**创建线程的工厂 ，用于批量创建线程，统一在创建线程时设置一些参数，如是否守护线程、线程的优先级等。如果不指定，会新建一个默认的线程工厂。

- **RejectedExecutionHandler handler** **拒绝处理策略**，线程数量大于最大线程数就会采用拒绝处理策略，四种拒绝处理的策略为 ：
  - **ThreadPoolExecutor.AbortPolicy**：**默认拒绝处理策略**，丢弃任务并抛出RejectedExecutionException异常。
  - **ThreadPoolExecutor.DiscardPolicy**：丢弃新来的任务，但是不抛出异常。
  - **ThreadPoolExecutor.DiscardOldestPolicy**：丢弃队列头部（最旧的）的任务，然后重新尝试执行程序（如果再次失败，重复此过程）。
  - **ThreadPoolExecutor.CallerRunsPolicy**：由调用线程处理该任务。

### ThreadPoolExecutor的策略

线程池本身有一个调度线程，这个线程就是用于管理布控整个线程池里的各种任务和事务，例如创建线程、销毁线程、任务队列管理、线程队列管理等等。

#### 线程池的状态

`ThreadPoolExecutor`类中定义了一个`volatile int`变量**runState**来表示线程池的状态 ，分别为RUNNING、SHUTDOWN、STOP、TIDYING 、TERMINATED。

- 线程池创建后处于**RUNNING**状态。

- 调用`shutdown()`方法后处于**SHUTDOWN**状态，线程池不能接受新的任务，清除一些空闲worker,会等待阻塞队列的任务完成。

- 调用`shutdownNow()`方法后处于**STOP**状态，线程池不能接受新的任务，中断所有线程，阻塞队列中没有被执行的任务全部丢弃。此时，poolsize=0,阻塞队列的size也为0。

- 当所有的任务已终止，ctl记录的”任务数量”为0，线程池会变为**TIDYING**状态。接着会执行`terminated()`函数。

	> `ThreadPoolExecutor`中有一个控制状态的属性叫`ctl`，它是一个`AtomicInteger`类型的变量。

- 线程池处在TIDYING状态时，**执行完`terminated()`方法之后**，就会由 **TIDYING -> TERMINATED**， 线程池被设置为TERMINATED状态。

### 线程池主要的任务处理流程

~~~java
// JDK 1.8 
public void execute(Runnable command) {
    if (command == null)
        throw new NullPointerException();   
    int c = ctl.get();
    // 1.当前线程数小于corePoolSize,则调用addWorker创建核心线程执行任务
    if (workerCountOf(c) < corePoolSize) {
       if (addWorker(command, true))
           return;
       c = ctl.get();
    }
    // 2.如果不小于corePoolSize，则将任务添加到workQueue队列。
    if (isRunning(c) && workQueue.offer(command)) {
        int recheck = ctl.get();
        // 2.1 如果isRunning返回false(状态检查)，则remove这个任务，然后执行拒绝策略。
        if (! isRunning(recheck) && remove(command))
            reject(command);
            // 2.2 线程池处于running状态，但是没有线程，则创建线程
        else if (workerCountOf(recheck) == 0)
            addWorker(null, false);
    }
    // 3.如果放入workQueue失败，则创建非核心线程执行任务，
    // 如果这时创建非核心线程失败(当前线程总数不小于maximumPoolSize时)，就会执行拒绝策略。
    else if (!addWorker(command, false))
         reject(command);
}
~~~

#### 二次检查线程池的状态`ctl.get()`?

在多线程的环境下，线程池的状态是时刻发生变化的。很有可能刚获取线程池状态后线程池状态就改变了。判断是否将`command`加入`workqueue`是线程池之前的状态。倘若没有二次检查，万一线程池处于非**RUNNING**状态（在多线程环境下很有可能发生），那么`command`永远不会执行。

#### 处理流程

1. 线程总数量 < corePoolSize，无论线程是否空闲，都会新建一个核心线程执行任务（让核心线程数量快速达到corePoolSize，在核心线程数量 < corePoolSize时）。**注意，这一步需要获得全局锁。**

2. 线程总数量 >= corePoolSize时，新来的线程任务会进入任务队列中等待，然后空闲的核心线程会依次去缓存队列中取任务来执行（体现了**线程复用**）。 

3. 当缓存队列满了，说明这个时候任务已经多到爆棚，需要一些“临时工”来执行这些任务了。于是会创建非核心线程去执行这个任务。**注意，这一步需要获得全局锁。**

4. 缓存队列满了， 且总线程数达到了maximumPoolSize，则会采取上面提到的拒绝策略进行处理。

![](../img/线程池的处理流程.png)

### ThreadPoolExecutor如何做到线程复用的

//todo 后面再阅读

https://redspider.gitbook.io/concurrent/di-san-pian-jdk-gong-ju-pian/12#12-2-4-threadpoolexecutor-ru-he-zuo-dao-xian-cheng-fu-yong-de

## 4 种常见的线程池

### newCachedThreadPool

~~~java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
~~~



#### 运行流程

1. 提交任务进线程池。

2. 因为**corePoolSize**为0的关系，不创建核心线程，线程池最大为Integer.MAX_VALUE。

3. 尝试将任务添加到**SynchronousQueue**队列。

4. 如果SynchronousQueue入列成功，等待被当前运行的线程空闲后拉取执行。如果当前没有空闲线程，那么就创建一个非核心线程，然后从SynchronousQueue拉取任务并在当前线程执行。

5. 如果SynchronousQueue已有任务在等待，入列操作将会阻塞。

#### 使用场景

当需要执行很多**短时间**的任务时，CacheThreadPool的线程复用率比较高， 会显著的**提高性能**。而且线程60s后会回收，意味着即使没有任务进来，CacheThreadPool并不会占用很多资源。

### newFixedThreadPool

~~~java
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
}
~~~

#### 说明

核心线程数量和总线程数量相等，都是传入的参数nThreads，所以只能创建核心线程，不能创建非核心线程。因为LinkedBlockingQueue的默认大小是Integer.MAX_VALUE，故如果核心线程空闲，则交给核心线程处理；如果核心线程不空闲，则入列等待，直到核心线程空闲。

#### 与CachedThreadPool的区别

- 因为 corePoolSize == maximumPoolSize ，所以FixedThreadPool只会创建核心线程。 而CachedThreadPool因为corePoolSize=0，所以只会创建非核心线程。

- 在 getTask() 方法，如果队列里没有任务可取，线程会一直阻塞在 LinkedBlockingQueue.take() ，线程不会被回收。 CachedThreadPool会在60s后收回。

- 由于线程不会被回收，会一直卡在阻塞，所以**没有任务的情况下， FixedThreadPool占用资源更多**。 

- 都几乎不会触发拒绝策略，但是原理不同。FixedThreadPool是因为阻塞队列可以很大（最大为Integer最大值），故几乎不会触发拒绝策略；CachedThreadPool是因为线程池很大（最大为Integer最大值），几乎不会导致线程数量大于最大线程数，故几乎不会触发拒绝策略。

### newSingleThreadExecutor

~~~java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
~~~

有且仅有一个核心线程（ corePoolSize == maximumPoolSize=1），使用了LinkedBlockingQueue（容量很大），所以，**不会创建非核心线程**。所有任务按照**先来先执行**的顺序执行。如果这个唯一的线程不空闲，那么新来的任务会存储在任务队列里等待执行。

### newScheduledThreadPool

~~~java
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

//ScheduledThreadPoolExecutor():
public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
~~~

创建一个定长线程池，支持定时及周期性任务执行。

# 阻塞队列

## 阻塞队列的由来

生产者-消费者模式：生产者将生产的资源存进缓冲池中，消费者从缓冲池中拿到资源进行消费。

实现该模式，需要让**多个线程操作共享变量**（即资源），所以很容易引发**线程安全问题**，造成**重复消费**和**死锁**，尤其是生产者和消费者存在多个的情况。另外，当缓冲池空了，我们需要阻塞消费者，唤醒生产者；当缓冲池满了，我们需要阻塞生产者，唤醒消费者，这些个**等待-唤醒**逻辑都需要自己实现。

JDK中实现该逻辑的数据结构`BlockingQueue`,只管往里面存、取就行，而不用担心多线程环境下存、取共享变量的线程安全问题。

## BlockingQueue

BlockingQueue是Java util.concurrent包下重要的数据结构，区别于普通的队列，BlockingQueue提供了**线程安全的队列访问方式**，并发包下很多高级同步类的实现都是基于BlockingQueue实现的。BlockingQueue一般用于生产者-消费者模式，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。**BlockingQueue就是存放元素的容器**。

### BlockingQueue的操作方法

阻塞队列提供了四组不同的方法用于插入、移除、检查元素：

| 方法\处理方式 | 抛出异常  | 返回特殊值 | 一直阻塞   | 超时退出           |
| ------------- | --------- | ---------- | ---------- | ------------------ |
| 插入方法      | add(e)    | offer(e)   | **put(e)** | offer(e,time,unit) |
| 移除方法      | remove()  | poll()     | **take()** | poll(time,unit)    |
| 检查方法      | element() | peek()     | -          |                    |

- 抛出异常：如果试图的操作无法立即执行，抛异常。当阻塞队列满时候，再往队列里插入元素，会抛出IllegalStateException(“Queue full”)异常。当队列为空时，从队列里获取元素时会抛出NoSuchElementException异常 。

- 返回特殊值：如果试图的操作无法立即执行，返回一个特殊值，通常是true / false。

- 一直阻塞：如果试图的操作无法立即执行，则一直阻塞或者响应中断。

- 超时退出：如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功，通常是 true / false。

>不能往阻塞队列中插入null,会抛出空指针异常。
>
>可以访问阻塞队列中的任意元素，调用remove(o)可以将队列之中的特定对象移除，但并不高效，尽量避免使用。

### BlockingQueue的实现类

#### ArrayBlockingQueue

由**数组**结构组成的**有界**阻塞队列。内部结构是数组，故具有数组的特性。

~~~java
public ArrayBlockingQueue(int capacity, boolean fair){
    //..省略代码
}
~~~

可以初始化队列大小， 且一旦初始化不能改变。构造方法中的fair表示控制对象的内部锁是否采用公平锁，默认是**非公平锁**。

#### LinkedBlockingQueue

由**链表**结构组成的**有界**阻塞队列。内部结构是链表，具有链表的特性。默认队列的大小是`Integer.MAX_VALUE`，也可以指定大小。此队列按照**先进先出**的原则对元素进行排序。

#### DelayQueue

该队列中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素 。注入其中的元素必须实现 java.util.concurrent.Delayed 接口。 

DelayQueue是一个没有大小限制的队列，因此往队列中插入数据的操作（生产者）永远不会被阻塞，而只有获取数据的操作（消费者）才会被阻塞。 

#### PriorityBlockingQueue

基于优先级的无界阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定），内部控制线程同步的锁采用的是公平锁。

> **PriorityBlockingQueue**不会阻塞数据生产者（因为队列是无界的），而只会在没有可消费的数据时，阻塞数据的消费者。因此使用的时候要特别注意，**生产者生产数据的速度绝对不能快于消费者消费数据的速度，否则时间一长，会最终耗尽所有的可用堆内存空间。**对于使用默认大小的**LinkedBlockingQueue**也是一样的。

#### SynchronousQueue

这个队列比较特殊，**没有任何内部容量**，甚至连一个队列的容量都没有。并且每个 put 必须等待一个 take，反之亦然

## 阻塞队列的原理

利用了Lock锁的多条件（Condition）阻塞控制。

ArrayBlockingQueue JDK 1.8的源码。

### 构造器

除了初始化队列的大小和是否是公平锁之外，还对同一个锁（lock）初始化了两个监视器，分别是notEmpty和notFull。这两个监视器的作用目前可以简单理解为标记分组，当该线程是put操作时，给他加上监视器notFull,标记这个线程是一个生产者；当线程是take操作时，给他加上监视器notEmpty，标记这个线程是消费者。

~~~java
//数据元素数组
final Object[] items;
//下一个待取出元素索引
int takeIndex;
//下一个待添加元素索引
int putIndex;
//元素个数
int count;
//内部锁
final ReentrantLock lock;
//消费者监视器
private final Condition notEmpty;
//生产者监视器
private final Condition notFull;  

public ArrayBlockingQueue(int capacity, boolean fair) {
    //..省略其他代码
    lock = new ReentrantLock(fair);
    notEmpty = lock.newCondition();
    notFull =  lock.newCondition();
}
~~~

### put操作

~~~java
public void put(E e) throws InterruptedException {
    checkNotNull(e);
    final ReentrantLock lock = this.lock;
    // 1.自旋拿锁
    lock.lockInterruptibly();
    try {
        // 2.判断队列是否满了
        while (count == items.length)
            // 2.1如果满了，阻塞该线程，并标记为notFull线程，
            // 等待notFull的唤醒，唤醒之后继续执行while循环。
            notFull.await();
        // 3.如果没有满，则进入队列
        enqueue(e);
    } finally {
        lock.unlock();
    }
}
private void enqueue(E x) {
    // assert lock.getHoldCount() == 1;
    // assert items[putIndex] == null;
    final Object[] items = this.items;
    items[putIndex] = x;
    if (++putIndex == items.length)
        putIndex = 0;
    count++;
    // 4 唤醒一个等待的线程
    notEmpty.signal();
}
~~~

### put流程总结

1. 所有执行put操作的线程竞争lock锁，拿到了lock锁的线程进入下一步，没有拿到lock锁的线程自旋竞争锁。

2. 判断阻塞队列是否满了，如果满了，则调用await方法阻塞这个线程，并标记为notFull（生产者）线程，同时释放lock锁,等待被消费者线程唤醒。

3. 如果没有满，则调用enqueue方法将元素put进阻塞队列。注意这一步的线程还有一种情况是第二步中阻塞的线程被唤醒且又拿到了lock锁的线程。

4. 唤醒一个标记为notEmpty（消费者）的线程。

### take操作

~~~java
public E take() throws InterruptedException {
    final ReentrantLock lock = this.lock;
    lock.lockInterruptibly();
    try {
        while (count == 0)
            notEmpty.await();
        return dequeue();
    } finally {
        lock.unlock();
    }
}
private E dequeue() {
    // assert lock.getHoldCount() == 1;
    // assert items[takeIndex] != null;
    final Object[] items = this.items;
    @SuppressWarnings("unchecked")
    E x = (E) items[takeIndex];
    items[takeIndex] = null;
    if (++takeIndex == items.length)
        takeIndex = 0;
    count--;
    if (itrs != null)
        itrs.elementDequeued();
    notFull.signal();
    return x;
}
~~~

### take流程总结

1. 所有执行take操作的线程竞争lock锁，拿到了lock锁的线程进入下一步，没有拿到lock锁的线程自旋竞争锁。

2. 判断阻塞队列是否为空，如果是空，则调用await方法阻塞这个线程，并标记为notEmpty（消费者）线程，同时释放lock锁,等待被生产者线程唤醒。

3. 如果没有空，则调用dequeue方法。注意这一步的线程还有一种情况是第二步中阻塞的线程被唤醒且又拿到了lock锁的线程。

4. 唤醒一个标记为notFull（生产者）的线程。

### 注意事项

1. put和take操作都需要**先获取锁**，没有获取到锁的线程会被挡在第一道大门之外自旋拿锁，直到获取到锁。

2. 就算拿到锁了之后，也**不一定**会顺利进行put/take操作，需要判断**队列是否可用**（是否满/空），如果不可用，则会被阻塞，**并释放锁**。

3. 在第2点被阻塞的线程会被唤醒，但是在唤醒之后，**依然需要拿到锁**才能继续往下执行，否则，自旋拿锁，拿到锁了再while判断队列是否可用（这也是为什么不用if判断，而使用while判断的原因）。

## 示例和使用场景

### 生产者-消费者模型

~~~java
package com.tz.juc.block;

import java.util.concurrent.ArrayBlockingQueue;

/**
 * 利用阻塞队列实现的生产者和消费者模式
 * @author tzcqupt
 */
public class BlockQueueProducerConsumer {
    private int queueSize = 10;
    private ArrayBlockingQueue<Integer> queue = new ArrayBlockingQueue<Integer>(queueSize);

    public static void main(String[] args)  {
        BlockQueueProducerConsumer blockQueueProducerConsumer = new BlockQueueProducerConsumer();
        Producer producer = blockQueueProducerConsumer.new Producer();
        Consumer consumer = blockQueueProducerConsumer.new Consumer();

        producer.start();
        consumer.start();
    }
    class Consumer extends Thread{

        @Override
        public void run() {
            consume();
        }
        private void consume() {
            while(true){
                try {
                    queue.take();
                    System.out.println("从队列取走一个元素，队列剩余"+queue.size()+"个元素");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    class Producer extends Thread{

        @Override
        public void run() {
            produce();
        }
        private void produce() {
            while(true){
                try {
                    queue.put(1);
                    System.out.println("向队列取中插入一个元素，队列剩余空间："+(queueSize-queue.size()));
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
~~~

# 锁接口和类

## synchronized的缺点

- 如果临界区是只读操作，其实可以多线程一起执行，但使用synchronized的话，**同一时间只能有一个线程执行**。

- synchronized无法知道线程有没有成功获取到锁

- 使用synchronized，如果临界区因为IO或者sleep方法等原因阻塞了，而当前线程又没有释放锁，就会导致**所有线程等待**。

## 锁的分类

### 可重入锁和非可重入锁

重入锁，就是支持重新进入的锁，也就是说这个锁支持一个**线程对资源重复加锁**。

synchronized关键字就是使用的重入锁。比如说，你在一个synchronized实例方法里面调用另一个本实例的synchronized实例方法，它可以重新进入这个锁，不会出现任何异常。

如果我们自己在继承AQS实现同步器的时候，没有考虑到占有锁的线程再次获取锁的场景，可能就会导致线程阻塞，那这个就是一个“非可重入锁”。

`ReentrantLock`的中文意思就是可重入锁。

### 公平锁与非公平锁

公平==>先来后到，FIFO

如果对一个锁来说，先对锁获取请求的线程一定会先被满足，后对锁获取请求的线程后被满足，那这个锁就是公平的。

一般情况下，**非公平锁能提升一定的效率。但是非公平锁可能会发生线程饥饿（有一些线程长时间得不到锁）的情况**。

`ReentrantLock`支持非公平锁和公平锁两种。

### 读写锁和排它锁

**排它锁**：同一时刻只允许一个线程进行访问。`synchronized`用的锁和`ReentrantLock`都是排它锁。

**读写锁**可以再同一时刻允许多个读线程访问。Java提供了`ReentrantReadWriteLock`类作为读写锁的默认实现，内部维护了两个锁：一个读锁，一个写锁。通过分离读锁和写锁，使得在“读多写少”的环境下，大大地提高了性能。

> 注意，即使用读写锁，在写线程访问时，所有的读线程和其它写线程均被阻塞。

## JDK 中接口和类

### 抽象类AQS/AQLS/AOS

**AQS**（AbstractQueuedSynchronizer）是在JDK 1.5 发布的，提供了一个“队列同步器”的基本功能实现。而AQS里面的“资源”是用一个`int`类型的数据来表示的，有时候我们的业务需求资源的数量超出了`int`的范围，所以在JDK 1.6 中，多了一个**AQLS**（AbstractQueuedLongSynchronizer）。它的代码跟AQS几乎一样，只是把资源的类型变成了`long`类型。

AQS和AQLS都继承了一个类叫**AOS**（AbstractOwnableSynchronizer）。这个类也是在JDK 1.6 中出现的。这个类只有几行简单的代码。它是用于表示锁与持有者之间的关系（独占模式）。

### 接口Condition/Lock/ReadWriteLock

juc.locks包下共有三个接口：`Condition`、`Lock`、`ReadWriteLock`。其中，Lock和ReadWriteLock从名字就可以看得出来，分别是锁和读写锁的意思。Lock接口里面有一些获取锁和释放锁的方法声明，而ReadWriteLock里面只有两个方法，分别返回“读锁”和“写锁”：

~~~java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
~~~

Lock接口中有一个方法是可以获得一个`Condition`。

每个对象都可以用继承自`Object`的**wait/notify**方法来实现**等待/通知机制**。而Condition接口也提供了类似Object监视器的方法，通过与**Lock**配合来实现等待/通知模式。

Condition和Object的wait/notify基本相似。其中，Condition的await方法对应的是Object的wait方法，而Condition的**signal/signalAll**方法则对应Object的notify/notifyAll()。

### ReentrantLock

`ReentrantLock`是一个非抽象类，它是`Lock`接口的JDK默认实现，实现了锁的基本功能。从名字上看，它是一个**可重入锁**，从源码上看，它内部有一个抽象类`Sync`，是继承了`AQS`，自己实现的一个同步器。同时，`ReentrantLock`内部有两个非抽象类`NonfairSync`和`FairSync`，它们都继承了`Sync`。从名字上看得出，分别是**非公平同步器**和**公平同步器**的意思。这意味着`ReentrantLock`可以支持**公平锁**和**非公平锁**

通过看着两个同步器的源码可以发现，它们的实现都是**独占**的。都调用了AOS的`setExclusiveOwnerThread`方法，所以`ReentrantLock`的锁的”独占“的，也就是说，它的锁都是**排他锁**，不能共享。

在`ReentrantLock`的构造方法里，可以传入一个`boolean`类型的参数，来指定它是否是一个公平锁，默认情况下是非公平的。这个参数一旦实例化后就不能修改，只能通过`isFair()`方法来查看。

### ReentrantReadWriteLock

它是R`eadWriteLock`接口的JDK默认实现。它与`ReentrantLock`的功能类似，同样是**可重入**的，支持**非公平锁**和**公平锁**。不同的是，它还支持**读写锁**。

~~~java
// 内部结构
private final ReentrantReadWriteLock.ReadLock readerLock;
private final ReentrantReadWriteLock.WriteLock writerLock;
final Sync sync;
abstract static class Sync extends AbstractQueuedSynchronizer {
    // 具体实现
}
static final class NonfairSync extends Sync {
    // 具体实现
}
static final class FairSync extends Sync {
    // 具体实现
}
public static class ReadLock implements Lock, java.io.Serializable {
    private final Sync sync;
    protected ReadLock(ReentrantReadWriteLock lock) {
            sync = lock.sync;
    }
    // 具体实现
}
public static class WriteLock implements Lock, java.io.Serializable {
    private final Sync sync;
    protected WriteLock(ReentrantReadWriteLock lock) {
            sync = lock.sync;
    }
    // 具体实现
}

// 构造方法，初始化两个锁
public ReentrantReadWriteLock(boolean fair) {
    sync = fair ? new FairSync() : new NonfairSync();
    readerLock = new ReadLock(this);
    writerLock = new WriteLock(this);
}

// 获取读锁和写锁的方法
public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
~~~

ReentrantReadWriteLock实现了读写锁，但它有一个小弊端，就是在“写”操作的时候，其它线程不能写也不能读。我们称这种现象为“写饥饿”。

### StampedLock

来自Java 8  它没有实现`Lock`接口和`ReadWriteLoc`k接口，但它其实是实现了“读写锁”的功能，并且性能比`ReentrantReadWriteLock`更高。`StampedLock`还把读锁分为了**乐观读锁**和**悲观读锁**两种。

官方代码：

~~~java
class Point {
   private double x, y;
   private final StampedLock sl = new StampedLock();

   // 写锁的使用
   void move(double deltaX, double deltaY) {
     long stamp = sl.writeLock(); // 获取写锁
     try {
       x += deltaX;
       y += deltaY;
     } finally {
       sl.unlockWrite(stamp); // 释放写锁
     }
   }

   // 乐观读锁的使用
   double distanceFromOrigin() {
     long stamp = sl.tryOptimisticRead(); // 获取乐观读锁
     double currentX = x, currentY = y;
     if (!sl.validate(stamp)) { // //检查乐观读锁后是否有其他写锁发生，有则返回false
        stamp = sl.readLock(); // 获取一个悲观读锁
        try {
          currentX = x;
          currentY = y;
        } finally {
           sl.unlockRead(stamp); // 释放悲观读锁
        }
     }
     return Math.sqrt(currentX * currentX + currentY * currentY);
   }

   // 悲观读锁以及读锁升级写锁的使用
   void moveIfAtOrigin(double newX, double newY) {
     long stamp = sl.readLock(); // 悲观读锁
     try {
       while (x == 0.0 && y == 0.0) {
         // 读锁尝试转换为写锁：转换成功后相当于获取了写锁，转换失败相当于有写锁被占用
         long ws = sl.tryConvertToWriteLock(stamp); 

         if (ws != 0L) { // 如果转换成功
           stamp = ws; // 读锁的票据更新为写锁的
           x = newX;
           y = newY;
           break;
         }
         else { // 如果转换失败
           sl.unlockRead(stamp); // 释放读锁
           stamp = sl.writeLock(); // 强制获取写锁
         }
       }
     } finally {
       sl.unlock(stamp); // 释放所有锁
     }
   }
}}
~~~

# 并发集合容器

## 并发容器

常用容器类架构

![](../img/常用容器类.png)

### 并发Map

#### `ConcurrentHashMap`类

##### ConcurrentMap接口

~~~java
public interface ConcurrentMap<K, V> extends Map<K, V> {

    //插入元素
    //如果插入的key相同，则不替换原有的value值；
    V putIfAbsent(K key, V value);

    //移除元素
    //增加了对value的判断，如果要删除的key-value不能与Map中原有的key-value对应上，则不会删除该元素;
    boolean remove(Object key, Object value);

    //替换元素
    //增加了对value值的判断，如果key-oldValue能与Map中原有的key-value对应上，才进行替换操作；
    boolean replace(K key, V oldValue, V newValue);

    //替换元素
    //如果key存在则直接替换
    V replace(K key, V value);

}
~~~
##### `ConcurrentHashMap`原理

`ConcurrentHashMap`提供了一种粒度更细的加锁机制来实现在多线程下更高的性能，这种机制叫分段锁(Lock Striping)。

就是**将数据分段，对每一段数据分配一把锁**。当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

`ConcurrentHashMap`是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁ReentrantLock，HashEntry则用于存储键值对数据。

一个`ConcurrentHashMap`里包含一个Segment数组，Segment的结构和HashMap类似，是一种数组和链表结构， 一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构（同HashMap一样，它也会在长度达到8的时候转化为红黑树）的元素， 每个Segment守护者一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得它对应的Segment锁。

#### `ConcurrentSkipListMap`类

##### ConcurrentNavigableMap接口

`ConcurrentNavigableMap`接口继承了`NavigableMap`接口，这个接口提供了针对给定搜索目标返回最接近匹配项的导航方法。

##### `ConcurrentSkipListMap`原理

`ConcurrentNavigableMap`接口的主要实现类是`ConcurrentSkipListMap`类。从名字上来看，它的底层使用的是跳表（SkipList）的数据结构。跳表是一种**空间换时间**的数据结构，可以使用CAS来保证并发安全性。

### 并发Queue

JDK提供了对队列和双端队列的线程安全的类：`ConcurrentLinkedDeque`和`ConcurrentLinkedQueue`。这两个类是使用CAS来实现线程安全的。

### 并发Set

JDK提供了`ConcurrentSkipListSet`，是线程安全的有序的集合。底层是使用`ConcurrentSkipListMap`实现。

google的guava框架的线程安全set `Set<String> s = Sets.newConcurrentHashSet();`

# CopyOnWrite

## 概念

CopyOnWrite是计算机设计领域中的一种优化策略，也是一种在并发场景下常用的设计思想——写入时复制思想。

就是当有多个调用者同时去请求一个资源数据的时候，有一个调用者出于某些原因需要对当前的数据源进行修改，这个时候系统将会复制一个当前数据源的副本给调用者修改。

CopyOnWrite容器即**写时复制的容器**,当我们往一个容器中添加元素的时候，不直接往容器中添加，而是将当前容器进行copy，复制出来一个新的容器，然后向新容器中添加我们需要的元素，最后将原容器的引用指向新容器。

可以在并发的场景下对容器进行"读操作"而不需要"加锁"，从而达到读写分离的目的。

## CopyOnWriteArrayList

### 优点

CopyOnWriteArrayList经常被用于“读多写少”的并发场景，是因为CopyOnWriteArrayList无需任何同步措施，大大增强了读的性能。

### 缺点

1. CopyOnWriteArrayList每次执行写操作都会将原容器进行拷贝了一份，数据量大的时候，内存会存在较大的压力，可能会引起频繁Full GC（ZGC因为没有使用Full GC）
2. opyOnWriteArrayList由于实现的原因，写和读分别作用在不同新老容器上，在写操作执行过程中，读不会阻塞，但读取到的却是老容器的数据。

> 如果我们希望写入的数据马上能准确地读取，请不要使用CopyOnWrite容器。

# 线程通信工具类

## Semaphore

### 介绍

作用：限制线程的数量。

提供的功能就是多个线程彼此“打信号”。而这个“信号”是一个`int`类型的数据，也可以看成是一种“资源”。

可以在构造函数中传入初始资源总数，以及是否使用“公平”的同步器。默认情况下，是非公平的。

~~~java
// 默认情况下使用非公平
public Semaphore(int permits) {
    sync = new NonfairSync(permits);
}

public Semaphore(int permits, boolean fair) {
    sync = fair ? new FairSync(permits) : new NonfairSync(permits);
}
~~~

最主要的方法是acquire方法和release方法。acquire()方法会申请一个permit，而release方法会释放一个permit。当然，你也可以申请多个acquire(int permits)或者释放多个release(int permits)。

每次acquire，permits就会减少一个或者多个。如果减少到了0，再有其他线程来acquire，那就要阻塞这个线程直到有其它线程release permit为止。

### 案例

Semaphore往往用于资源有限的场景中，去限制线程的数量。

~~~java
public class SemaphoreDemo {
    static class MyThread implements Runnable {

        private int value;
        private Semaphore semaphore;

        public MyThread(int value, Semaphore semaphore) {
            this.value = value;
            this.semaphore = semaphore;
        }

        @Override
        public void run() {
            try {
                semaphore.acquire(); // 获取permit
                System.out.println(String.format("当前线程是%d, 还剩%d个资源，还有%d个线程在等待",
                        value, semaphore.availablePermits(), semaphore.getQueueLength()));
                // 睡眠随机时间，打乱释放顺序
                Random random =new Random();
                Thread.sleep(random.nextInt(1000));
                semaphore.release(); // 释放permit
                System.out.println(String.format("线程%d释放了资源", value));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);
        for (int i = 0; i < 10; i++) {
            new Thread(new MyThread(i, semaphore)).start();
        }
    }
}
~~~

### 原理

Semaphore内部有一个继承了AQS的同步器Sync，重写了`tryAcquireShared`方法。在这个方法里，会去尝试获取资源。

如果获取失败（想要的资源数量小于目前已有的资源数量），就会返回一个负数（代表尝试获取资源失败）。然后当前线程就会进入AQS的等待队列。

## Exchanger

Exchanger类用于两个线程交换数据。它支持泛型，也就是说你可以在两个线程之间传送任何数据。

### 案例

~~~java
public class ExchangerDemo {
    public static void main(String[] args) throws InterruptedException {
        Exchanger<String> exchanger = new Exchanger<>();

        new Thread(() -> {
            try {
                System.out.println("这是线程A，得到了另一个线程的数据："
                        + exchanger.exchange("这是来自线程A的数据"));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        System.out.println("这个时候线程A是阻塞的，在等待线程B的数据");
        Thread.sleep(1000);

        new Thread(() -> {
            try {
                System.out.println("这是线程B，得到了另一个线程的数据："
                        + exchanger.exchange("这是来自线程B的数据"));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
~~~

**输出：**

> 这个时候线程A是阻塞的，在等待线程B的数据 
>
> 这是线程B，得到了另一个线程的数据：这是来自线程A的数据 
>
> 这是线程A，得到了另一个线程的数据：这是来自线程B的数据

## CountDownLatch

线程等待直到计数器减为0时开始工作。CountDown代表计数递减，Latch表示屏障。

假设某个线程在执行任务之前，需要等待其它线程完成一些前置任务，必须等所有的前置任务都完成，才能开始执行本线程的任务。

### 案例

玩游戏的时候，在游戏真正开始之前，一般会等待一些前置任务完成，比如“加载地图数据”，“加载人物模型”，“加载背景音乐”等等。只有当所有的东西都加载完成后，玩家才能真正进入游戏。

~~~java
public class CountDownLatchDemo {
    // 定义前置任务线程
    static class PreTaskThread implements Runnable {

        private String task;
        private CountDownLatch countDownLatch;

        public PreTaskThread(String task, CountDownLatch countDownLatch) {
            this.task = task;
            this.countDownLatch = countDownLatch;
        }

        @Override
        public void run() {
            try {
                Random random = new Random();
                Thread.sleep(random.nextInt(1000));
                System.out.println(task + " - 任务完成");
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) {
        // 假设有三个模块需要加载
        CountDownLatch countDownLatch = new CountDownLatch(3);

        // 主任务
        new Thread(() -> {
            try {
                System.out.println("等待数据加载...");
                System.out.println(String.format("还有%d个前置任务", countDownLatch.getCount()));
                countDownLatch.await();
                System.out.println("数据加载完成，正式开始游戏！");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // 前置任务
        new Thread(new PreTaskThread("加载地图数据", countDownLatch)).start();
        new Thread(new PreTaskThread("加载人物模型", countDownLatch)).start();
        new Thread(new PreTaskThread("加载背景音乐", countDownLatch)).start();
    }
}
~~~

### 原理

内部定义了一个AQS的实现类Sync。

> 构造器中的**计数值（count）实际上就是闭锁需要等待的线程数量**。这个值只能被设置一次，而且CountDownLatch**没有提供任何机制去重新设置这个计数值**。

## CyclicBarrier

理解：循环的屏障，作用跟CountDownLatch类似，但是可以重复使用。

### 案例

如果玩一个游戏有多个“关卡”，那使用CountDownLatch显然不太合适，那需要为每个关卡都创建一个实例。那我们可以使用CyclicBarrier来实现每个关卡的数据加载等待功能。

~~~java
public class CyclicBarrierDemo {
    static class PreTaskThread implements Runnable {

        private String task;
        private CyclicBarrier cyclicBarrier;

        public PreTaskThread(String task, CyclicBarrier cyclicBarrier) {
            this.task = task;
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            // 假设总共三个关卡
            for (int i = 1; i < 4; i++) {
                try {
                    Random random = new Random();
                    Thread.sleep(random.nextInt(1000));
                    System.out.println(String.format("关卡%d的任务%s完成", i, task));
                    cyclicBarrier.await();
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
                cyclicBarrier.reset(); // 重置屏障
            }
        }
    }

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
            System.out.println("本关卡所有前置任务完成，开始游戏...");
        });

        new Thread(new PreTaskThread("加载地图数据", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载人物模型", cyclicBarrier)).start();
        new Thread(new PreTaskThread("加载背景音乐", cyclicBarrier)).start();
    }
}
~~~



输出：

>关卡1的任务加载人物模型完成
>关卡1的任务加载背景音乐完成
>关卡1的任务加载地图数据完成
>本关卡所有前置任务完成，开始游戏...
>关卡2的任务加载人物模型完成
>关卡2的任务加载背景音乐完成
>关卡2的任务加载地图数据完成
>本关卡所有前置任务完成，开始游戏...
>关卡3的任务加载背景音乐完成
>关卡3的任务加载人物模型完成
>关卡3的任务加载地图数据完成
>本关卡所有前置任务完成，开始游戏...

### 原理

`CyclicBarrier`没有分为`await()`和`countDown()`，而是只有单独的一个`await()`方法。

一旦调用await()方法的线程数量等于构造方法中传入的任务总量（这里是3），就代表达到屏障了。`CyclicBarrier`允许我们在达到屏障的时候可以执行一个任务，可以在构造方法传入一个`Runnable`类型的对象。上述案例就是在达到屏障时，输出“本关卡所有前置任务完成，开始游戏...”。

`CyclicBarrier`内部使用的是Lock + Condition实现的等待/通知模式

## Phaser

JDK 1.7出现，移相器，增强的`CyclicBarrier`.

`CyclicBarrier`，可以发现它在构造方法里传入“任务总量”`parties`之后，就不能修改这个值了，并且每次调用`await()`方法也只能消耗一个`parties`计数。但Phaser可以动态地调整任务总量！

### 变量解释

- party：对应一个线程，数量可以通过register或者构造参数传入;

- arrive：对应一个party的状态，初始时是unarrived，当调用`arriveAndAwaitAdvance()`或者 `arriveAndDeregister()`进入arrive状态，可以通过`getUnarrivedParties()`获取当前未到达的数量;

- register：注册一个party，每一阶段必须所有注册的party都到达才能进入下一阶段;

- deRegister：减少一个party。

- phase：阶段，当所有注册的party都arrive之后，将会调用Phaser的`onAdvance()`方法来判断是否要进入下一阶段。

### 案例

假设我们游戏有三个关卡，但只有第一个关卡有新手教程，需要加载新手教程模块。但后面的第二个关卡和第三个关卡都不需要。我们可以用Phaser来做这个需求。

~~~java
public class PhaserDemo {
    static class PreTaskThread implements Runnable {

        private String task;
        private Phaser phaser;

        public PreTaskThread(String task, Phaser phaser) {
            this.task = task;
            this.phaser = phaser;
        }

        @Override
        public void run() {
            for (int i = 1; i < 4; i++) {
                try {
                    // 第二次关卡起不加载NPC，跳过
                    if (i >= 2 && "加载新手教程".equals(task)) {
                        continue;
                    }
                    Random random = new Random();
                    Thread.sleep(random.nextInt(1000));
                    System.out.println(String.format("关卡%d，需要加载%d个模块，当前模块【%s】",
                            i, phaser.getRegisteredParties(), task));

                    // 从第二个关卡起，不加载NPC
                    if (i == 1 && "加载新手教程".equals(task)) {
                        System.out.println("下次关卡移除加载【新手教程】模块");
                        phaser.arriveAndDeregister(); // 移除一个模块
                    } else {
                        phaser.arriveAndAwaitAdvance();
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) {
        Phaser phaser = new Phaser(4) {
            @Override
            protected boolean onAdvance(int phase, int registeredParties) {
                System.out.println(String.format("第%d次关卡准备完成", phase + 1));
                return phase == 3 || registeredParties == 0;
            }
        };

        new Thread(new PreTaskThread("加载地图数据", phaser)).start();
        new Thread(new PreTaskThread("加载人物模型", phaser)).start();
        new Thread(new PreTaskThread("加载背景音乐", phaser)).start();
        new Thread(new PreTaskThread("加载新手教程", phaser)).start();
    }
}
~~~

**输出**：

>关卡1，需要加载4个模块，当前模块【加载地图数据】
>关卡1，需要加载4个模块，当前模块【加载背景音乐】
>关卡1，需要加载4个模块，当前模块【加载人物模型】
>关卡1，需要加载4个模块，当前模块【加载新手教程】
>下次关卡移除加载【新手教程】模块
>第1次关卡准备完成
>关卡2，需要加载3个模块，当前模块【加载人物模型】
>关卡2，需要加载3个模块，当前模块【加载地图数据】
>关卡2，需要加载3个模块，当前模块【加载背景音乐】
>第2次关卡准备完成
>关卡3，需要加载3个模块，当前模块【加载人物模型】
>关卡3，需要加载3个模块，当前模块【加载地图数据】
>关卡3，需要加载3个模块，当前模块【加载背景音乐】
>第3次关卡准备完成

> Phaser不在意具体有哪些线程arrive，只要达到它当前阶段的parties值，就触发屏障。是没有分辨具体是哪个线程的功能的，它在意的只是数量。

### 原理

内部使用了两个基于Fork-Join框架的原子类辅助。

# Fork/Join框架

## 概念

Fork/Join框架是一个实现了ExecutorService接口的多线程处理器，它专为那些可以通过递归分解成更细小的任务而设计，最大化的利用多核处理器来提高应用程序的性能。

与其他ExecutorService相关的实现相同的是，Fork/Join框架会将任务分配给线程池中的线程。而与之不同的是，Fork/Join框架在执行任务时使用了**工作窃取算法**。

## 工作窃取算法

工作窃取算法指的是在多线程执行不同任务队列的过程中，某个线程执行完自己队列的任务后从其他线程的任务队列里窃取任务来执行。

## Fork/Join的具体实现

Fork/Join框架简单来讲就是对任务的分割与子任务的合并，所以要实现这个框架，先得有**任务**。在Fork/Join框架里提供了抽象类`ForkJoinTask`来实现任务。

### ForkJoinTask

ForkJoinTask是一个类似普通线程的实体，但是比普通线程轻量得多。

#### fork()方法

使用线程池中的空闲线程异步提交任务

~~~java
// 本文所有代码都引自Java 8
public final ForkJoinTask<V> fork() {
    Thread t;
    // ForkJoinWorkerThread是执行ForkJoinTask的专有线程，由ForkJoinPool管理
    // 先判断当前线程是否是ForkJoin专有线程，如果是，则将任务push到当前线程所负责的队列里去
    if ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread)
        ((ForkJoinWorkerThread)t).workQueue.push(this);
    else
         // 如果不是则将线程加入队列
        // 没有显式创建ForkJoinPool的时候走这里，提交任务到默认的common线程池中
        ForkJoinPool.common.externalPush(this);
    return this;
}
~~~



**把任务推入当前工作线程的工作队列里**。

#### join()方法

等待处理任务的线程处理完毕，获得返回值。

~~~java
public final V join() {
    int s;
    // doJoin()方法来获取当前任务的执行状态
    if ((s = doJoin() & DONE_MASK) != NORMAL)
        // 任务异常，抛出异常
        reportException(s);
    // 任务正常完成，获取返回值
    return getRawResult();
}

/**
 * doJoin()方法用来返回当前任务的执行状态
 **/
private int doJoin() {
    int s; Thread t; ForkJoinWorkerThread wt; ForkJoinPool.WorkQueue w;
    // 先判断任务是否执行完毕，执行完毕直接返回结果（执行状态）
    return (s = status) < 0 ? s :
    // 如果没有执行完毕，先判断是否是ForkJoinWorkThread线程
    ((t = Thread.currentThread()) instanceof ForkJoinWorkerThread) ?
        // 如果是，先判断任务是否处于工作队列顶端（意味着下一个就执行它）
        // tryUnpush()方法判断任务是否处于当前工作队列顶端，是返回true
        // doExec()方法执行任务
        (w = (wt = (ForkJoinWorkerThread)t).workQueue).
        // 如果是处于顶端并且任务执行完毕，返回结果
        tryUnpush(this) && (s = doExec()) < 0 ? s :
        // 如果不在顶端或者在顶端却没未执行完毕，那就调用awitJoin()执行任务
        // awaitJoin()：使用自旋使任务执行完成，返回结果
        wt.pool.awaitJoin(w, this, 0L) :
    // 如果不是ForkJoinWorkThread线程，执行externalAwaitDone()返回任务结果
    externalAwaitDone();
}
~~~

ForkJoinPool.join()会使线程免于阻塞。

![](../img/forkJoin流程图.png)

#### 实例

创建任务的时候我们一般不直接继承ForkJoinTask，而是继承它的子类**RecursiveTask/RecursiveAction**

**RecursiveAction**可以看做是无返回值的**ForkJoinTask**

**RecursiveTask是有返回值的ForkJoinTask**。

### ForkJoinPool

ForkJoinPool是用于执行ForkJoinTask任务的执行（线程）池。

#### 源码

~~~java
@sun.misc.Contended
public class ForkJoinPool extends AbstractExecutorService {
    // 任务队列 双端队列，ForkJoinTask存放在这里。
    volatile WorkQueue[] workQueues;   

    // 线程的运行状态 ForkJoinPool的运行状态。SHUTDOWN状态用负数表示，其他用2的幂次表示。
    volatile int runState;  

    // 创建ForkJoinWorkerThread的默认工厂，可以通过构造函数重写
    public static final ForkJoinWorkerThreadFactory defaultForkJoinWorkerThreadFactory;

    // 公用的线程池，其运行状态不受shutdown()和shutdownNow()的影响
    static final ForkJoinPool common;

    // 私有构造方法，没有任何安全检查和参数校验，由makeCommonPool直接调用
    // 其他构造方法都是源自于此方法
    // parallelism: 并行度，
    // 默认调用java.lang.Runtime.availableProcessors() 方法返回可用处理器的数量
    private ForkJoinPool(int parallelism,
                         ForkJoinWorkerThreadFactory factory, // 工作线程工厂
                         UncaughtExceptionHandler handler, // 拒绝任务的handler
                         int mode, // 同步模式
                         String workerNamePrefix) { // 线程名prefix
        this.workerNamePrefix = workerNamePrefix;
        this.factory = factory;
        this.ueh = handler;
        this.config = (parallelism & SMASK) | mode;
        long np = (long)(-parallelism); // offset ctl counts
        this.ctl = ((np << AC_SHIFT) & AC_MASK) | ((np << TC_SHIFT) & TC_MASK);
    }

}
~~~