---
layout: post
title: 多线程学习笔记-基础篇
date: 2020-04-28
Author: tzcqupt
tags: [Java, 多线程]
comments: true
toc: true
---

# 相关概念

## 进程和线程

### 进程

进程就是**应用程序在内存中分配的空间，也就是正在运行的程序**

### 线程

让一个线程执行一个子任务，这样一个进程就包含了多个线程，每个线程负责一个单独的子任务。

### 区别

1. 他们两个本质的区别是**是否单独占有内存地址空间及其它系统资源（比如I/O）**
2. **进程是操作系统进行资源分配的基本单位，而线程是操作系统进行调度的基本单位**，即CPU分配时间的单位 。

### 上下文切换

上下文切换（有时也称做进程切换或任务切换）是指 CPU 从一个进程（或线程）切换到另一个进程（或线程）。上下文是指**某一时间点 CPU 寄存器和程序计数器的内容。**任务从保存到再加载的过程就是一次上下文切换。

#### 寄存器

寄存器是cpu内部的少量的速度很快的闪存，通常存储和访问计算过程的中间值提高计算机程序的运行速度。

#### 程序计数器

程序计数器一个专用的寄存器，用于表明指令序列中 CPU 正在执行的位置，存的值为正在执行的指令的位置或者下一个将要被执行的指令的位置，具体实现依赖于特定的系统。

## 线程组和线程优先级

### 线程组ThreadGroup

每个Thread必然存在于一个ThreadGroup中，Thread不能独立于ThreadGroup存在。执行main()方法线程的名字是main，如果在new Thread时没有显式指定，那么默认将父线程（当前执行new Thread的线程）线程组设置为自己的线程组。

#### 常用方法

##### 获取当前线程组的名字

~~~java
Thread.currentThread().getThreadGroup().getName()
~~~

##### 复制线程组

~~~java
// 复制一个线程数组到一个线程组
Thread[] threads = new Thread[threadGroup.activeCount()];
TheadGroup threadGroup = new ThreadGroup();
threadGroup.enumerate(threads);
~~~

线程组里可以有其他线程组。

### 线程优先级

- Java中线程优先级可以指定，范围是1~10。Java只是给操作系统一个优先级的**参考值**，线程最终**在操作系统的优先级**是多少还是由操作系统决定。

- Java中的优先级来说不是特别的可靠，**Java程序中对线程所设置的优先级只是给操作系统一个建议，操作系统不一定会采纳。而真正的调用顺序，是由操作系统的线程调度算法决定的**。

- Java提供一个**线程调度器**来监视和控制处于**RUNNABLE状态**的线程。线程的调度策略采用**抢占式**，优先级高的线程比优先级低的线程会有更大的几率优先执行。在优先级相同的情况下，按照“先到先得”的原则。每个Java程序都有一个默认的主线程，就是通过JVM启动的第一个线程main线程。

- 还有一种线程称为**守护线程（Daemon）**，守护线程默认的优先级比较低。如果某线程是守护线程，那如果所有的非守护线程结束，这个守护线程也会自动结束。一个线程默认是非守护线程，可以通过Thread类的setDaemon(boolean on)来设置。

## 线程的6个状态

### NEW

处于NEW状态的线程此时尚未启动。这里的尚未启动指的是还没调用Thread实例的start()方法。

### RUNNABLE

表示当前线程正在运行中。处于RUNNABLE状态的线程在Java虚拟机中运行，也有可能在等待其他系统资源（比如I/O）。

> Java线程的**RUNNABLE**状态其实是包括了传统操作系统线程的**ready**和**running**两个状态的。

### BLOCKED

阻塞状态。处于BLOCKED状态的线程正等待锁的释放以进入同步区。

### WAITING

等待状态。处于等待状态的线程变成RUNNABLE状态需要其他线程唤醒。

调用如下3个方法会使线程进入等待状态:

- Object.wait()：使当前线程处于等待状态直到另一个线程唤醒它；

- Thread.join()：等待线程执行完毕，底层调用的是Object实例的wait方法；

* LockSupport.park()：除非获得调用许可，否则禁用当前线程进行线程调度。

### TIMED_WAITING

超时等待状态。线程等待一个具体的时间，时间到后会被自动唤醒，拥有了去**争夺锁**的资格。

调用如下方法会使线程进入超时等待状态：

- Thread.sleep(long millis)：使当前线程睡眠指定时间；

- Object.wait(long timeout)：线程休眠指定时间，等待期间可以通过notify()/notifyAll()唤醒；

- Thread.join(long millis)：等待当前线程最多执行millis毫秒，如果millis为0，则会一直执行；

- LockSupport.parkNanos(long nanos)： 除非获得调用许可，否则禁用当前线程进行线程调度指定时间；

- LockSupport.parkUntil(long deadline)：同上，也是禁止线程进行调度指定时间；

### TERMINATED

终止状态。此时线程已执行完毕。

### Java线程状态转换图

![](..\img\线程状态转换图.png)

![Java异常结构总结图](..\img\Java异常结构.png)



## 线程中断

- Thread.interrupt()：中断线程。这里的中断线程并不会立即停止线程，而是设置线程的中断状态为true（默认是flase）；

- Thread.interrupted()：测试当前线程是否被中断。线程的中断状态受这个方法的影响，意思是调用一次使线程中断状态设置为true，连续调用两次会使得这个线程的中断状态重新转为false；

- Thread.isInterrupted()：测试当前线程是否被中断。与上面方法不同的是调用这个方法并不会影响线程的中断状态。

# Java多线程创建方式

## 多线程入门类和接口

### Thread类和Runnable接口

1. 继承`Thread`类，并重写`run`方法；
2. 实现`Runnable`接口的`run`方法；

#### 继承Thread类

~~~java
public class Demo {
    public static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("MyThread");
        }
    }

    public static void main(String[] args) {
        Thread myThread = new MyThread();
        myThread.start();
    }
}
~~~

#### 实现Runnable接口

~~~java
public class Demo {
    public static class MyThread implements Runnable {
        @Override
        public void run() {
            System.out.println("MyThread");
        }
    }

    public static void main(String[] args) {
        new MyThread().start();

        // Java 8 函数式编程，可以省略MyThread类
        new Thread(() -> {
            System.out.println("Java 8 匿名内部类");
        }).start();
    }
}
~~~

#### Thread的构造方法

~~~java
Thread(Runnable target)
Thread(Runnable target, String name)
~~~

#### Thread类的常用方法

- currentThread()：静态方法，返回对当前正在执行的线程对象的引用；

- start()：开始执行线程的方法，java虚拟机会调用线程内的run()方法；

- yield()：yield在英语里有放弃的意思，同样，这里的yield()指的是**当前线程愿意让出**对当前处理器的占用。这里需要注意的是，就算当前线程调用了yield()方法，程序在调度的时候，也还有可能继续运行这个线程的；

- sleep()：静态方法，使当前线程睡眠一段时间；

- join()：使当前线程等待另一个线程执行完毕之后再继续执行，内部调用的是Object类的`wait`方法实现的；

### Callable、Future与FutureTask

使用`Runnable`和`Thread`来创建一个新的线程。但是它们有一个弊端，就是`run`方法是没有返回值的。希望开启一个线程去执行一个任务，并且这个任务执行完成后有一个返回值。

#### Callable接口

`Callable`一般是配合线程池工具`ExecutorService`来使用。

~~~java
// 自定义Callable
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        // 模拟计算需要一秒
        Thread.sleep(1000);
        return 2;
    }
    public static void main(String args[]){
        // 使用
        ExecutorService executor = Executors.newCachedThreadPool();
        Task task = new Task();
        Future<Integer> result = executor.submit(task);
        // 注意调用get方法会阻塞当前线程，直到得到结果。
        // 所以实际编码中建议使用可以设置超时时间的重载get方法。
        System.out.println(result.get()); 
    }
}
~~~

#### Future接口

~~~java
public abstract interface Future<V> {
    //是否取消成功，参数表示是否采用中断的方式取消线程执行
    public abstract boolean cancel(boolean paramBoolean);
    public abstract boolean isCancelled();
    public abstract boolean isDone();
    public abstract V get() throws InterruptedException, ExecutionException;
    public abstract V get(long paramLong, TimeUnit paramTimeUnit)
            throws InterruptedException, ExecutionException, TimeoutException;
}
~~~

`cancel`方法是试图取消一个线程的执行，**并不一定能取消成功**。因为该线程可能已完成，已取消，或者其他因素。

#### FutureTask类

FutureTask实现了RunnableFuture接口，该接口继承了Runnable和Future接口。

~~~java
public interface RunnableFuture<V> extends Runnable, Future<V> {
   void run();
}
// 自定义Callable，与上面一样
class Task implements Callable<Integer>{
    @Override
    public Integer call() throws Exception {
        // 模拟计算需要一秒
        Thread.sleep(1000);
        return 2;
    }
    public static void main(String args[]){
        // 使用
        ExecutorService executor = Executors.newCachedThreadPool();
        FutureTask<Integer> futureTask = new FutureTask<>(new Task());
        executor.submit(futureTask);
        System.out.println(futureTask.get());
    }
}
~~~

FutureTask能够在高并发环境下**确保任务只执行一次**。

##### FutureTask的几种状态值说明

~~~java
private volatile int state;
    //初始创建时的状态
    private static final int NEW          = 0;
    //当任务执行完毕，FutureTask会将执行结果设置给outcome属性，在设置之前会将FutureTask的状态修改为COMPLETING。
    private static final int COMPLETING   = 1;
    //当任务执行完毕，FutureTask会将执行结果设置给outcome属性，在设置之后会将FutureTask的状态修改为NORMAL。
    private static final int NORMAL       = 2;
    //当任务在执行的过程中抛了异常，FutureTask会将异常信息设置给outcome属性，
    //在设置之前会将FutureTask的状态修改为COMPLETING，在设置之后将状态修改为EXCEPTIONAL。
    private static final int EXCEPTIONAL  = 3;
    //当外部想要取消任务，而又不允许当任务正在执行的时候被取消时会将FutureTask的状态修改为CANCELLED。
    private static final int CANCELLED    = 4;
    //当外部想要取消任务，同时允许当任务正在执行的时候被取消时，会先将FutureTask的状态设置为INTERRUPTING，
    //然后设置执行任务的线程的中断标记位。
    private static final int INTERRUPTING = 5;
    //当外部想要取消任务，同时允许当任务正在执行的时候被取消时，会先将FutureTask的状态设置为INTERRUPTING，
    //然后设置执行任务的线程的中断标记位，最后将Future的状态设置为INTERRUPTED。
    private static final int INTERRUPTED  = 6;
~~~

FutureTask的状态流转可能流程：

- NEW—>COMPLETING—>NORMAL（任务执行正常）

- NEW—>COMPLETING—>EXCEPTIONAL（任务执行异常）

- NEW—>CANCELLED（不允许执行中的取消）

- NEW—>INTERRUPTING—>INTERRUPTED（允许执行中的取消）

# 线程间的通信

## 锁与同步

Java中，锁的概念都是基于对象的，一个锁同一时间只能被一个线程持有。线程同步是线程按照一定的顺序执行。

~~~java
//定义锁
private static final Object lock = new Object();

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            //为方法加锁
            synchronized (lock) {
                for (int i = 0; i < 100; i++) {
                    System.out.println("Thread A " + i);
                }
            }
        }
    }
    static class ThreadB implements Runnable {
            @Override
            public void run() {
                synchronized (lock) {
                    for (int i = 0; i < 100; i++) {
                        System.out.println("Thread B " + i);
                    }
                }
            }
        }

    public static void main(String[] args) throws InterruptedException {
        new Thread(new ThreadA()).start();
        //防止线程B先得到锁
        Thread.sleep(10);
        new Thread(new ThreadB()).start();
    }
//线程B一样
~~~

## 等待通知机制

假如线程A现在持有了一个锁`lock`并开始执行，它可以使用`lock.wait()`让自己进入等待状态。这个时候，`lock`这个锁是被释放了的。

这时，线程B获得了`lock`这个锁并开始执行，它可以在某一时刻，使用`lock.notify()`，通知之前持有`lock`锁并进入等待状态的线程A，说“线程A你不用等了，可以往下执行了”。

> 需要注意的是，这个时候线程B并没有释放锁`lock`，除非线程B这个时候使用`lock.wait()`释放锁，或者线程B执行结束自行释放锁，线程A才能得到`lock`锁。

~~~java
private static Object lock = new Object();

    static class ThreadA implements Runnable {
        @Override
        public void run() {
            synchronized (lock) {
                for (int i = 0; i < 5; i++) {
                    try {
                        System.out.println("ThreadA: " + i);
                        lock.notify();
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                lock.notify();
            }
        }
    }
//线程B一样
~~~

## 信号量

Java中`volitile`关键字能够保证内存的可见性，如果用`volitile`关键字声明了一个变量，在一个线程里面改变了这个变量的值，那其它线程是立马可见更改后的值的。

## 管道

JDK提供了`PipedWriter`、 `PipedReader`、 `PipedOutputStream`、 `PipedInputStream`。其中，前面两个是基于字符的，后面两个是基于字节流的。

我们一个线程需要先另一个线程发送一个信息（比如字符串）或者文件等等时，就需要使用管道通信了。

~~~java
/**
 *
 * 1.线程ReaderThread开始执行，
 * 2.线程ReaderThread使用管道reader.read()进入”阻塞“，
 * 3.线程WriterThread开始执行，
 * 4.线程WriterThread用writer.write("test")往管道写入字符串，
 * 5.线程WriterThread使用writer.close()结束管道写入，并执行完毕，
 * 6.线程ReaderThread接受到管道输出的字符串并打印，
 * 7.线程ReaderThread执行完毕。
 */
public class Pipe {
    static class ReaderThread implements Runnable {
        private PipedReader reader;

        public ReaderThread(PipedReader reader) {
            this.reader = reader;
        }

        @Override
        public void run() {
            System.out.println("this is reader");
            int receive = 0;
            try {
                while ((receive = reader.read()) != -1) {
                    System.out.print((char)receive);
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    static class WriterThread implements Runnable {

        private PipedWriter writer;

        public WriterThread(PipedWriter writer) {
            this.writer = writer;
        }

        @Override
        public void run() {
            System.out.println("this is writer");
            int receive = 0;
            try {
                writer.write("test");
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    writer.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public static void main(String[] args) throws IOException, InterruptedException {
        PipedWriter writer = new PipedWriter();
        PipedReader reader = new PipedReader();
        writer.connect(reader); // 这里注意一定要连接，才能通信

        new Thread(new ReaderThread(reader)).start();
        Thread.sleep(1000);
        new Thread(new WriterThread(writer)).start();
    }
}

// 输出：
this is reader
this is writer
test
~~~

## 其他通信相关

### join

join()方法是Thread类的一个实例方法。它的作用是让当前线程陷入“等待”状态，等join的这个线程执行完成后，再继续执行当前线程。

### sleep

sleep方法是Thread类的一个静态方法。它的作用是让当前线程睡眠一段时间。

### join和sleep区别

- sleep方法是不会释放当前的锁的，而wait方法会。

- wait可以指定时间，也可以不指定；而sleep必须指定时间。

- wait释放cpu资源，同时释放锁；sleep释放cpu资源，但是不释放锁，所以易死锁。

- wait必须放在同步块或同步方法中，而sleep可以在任意位置

### ThreadLocal

ThreadLocal是一个本地线程副本变量工具类。内部是一个**弱引用**的Map来维护。

ThreadLocal为**线程本地变量**或**线程本地存储**。严格来说，ThreadLocal类并不属于多线程间的通信，而是让每个线程有自己”独立“的变量，线程之间互不影响。它为每个线程都创建一个**副本**，每个线程可以访问自己内部的副本变量。

ThreadLocal类最常用的就是set方法和get方法。

### InheritableThreadLocal

InheritableThreadLocal类与ThreadLocal类稍有不同，Inheritable是继承的意思。它不仅仅是当前线程可以存取副本值，而且它的子线程也可以存取这个副本值。