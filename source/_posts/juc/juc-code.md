---
layout: post
title: 多线程入门总结-代码篇
date: 2020-04-27
Author: tzcqupt
tags: [多线程]
categories: [多线程]
comments: true
toc: true
---

# 多线程基础

## 使用多线程

### 继承Thread

~~~java
public class MyThread extends Thread{
    @Override
    public void run(){
        System.out.println(this.getClass().getSimpleName());
        try {
            for (int i = 0; i < 10; i++) {
                int time = (int) (Math.random() * 1000);
                Thread.sleep(time);
                System.out.println("run= "+Thread.currentThread().getName());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
    public static void main(String[] args){
        try{
            MyThread myThread = new MyThread();
            myThread.start();
            for (int i = 0; i < 10; i++) {
                int time = (int) (Math.random() * 1000);
                Thread.sleep(time);
                System.out.println("run= "+Thread.currentThread().getName());
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~



### 实现Runnable接口

~~~java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println(this.getClass().getSimpleName());
    }
    public static void main(String[] args) {
        MyRunnable myRunnable = new MyRunnable();
        Thread thread = new Thread(myRunnable);
        thread.start();
        System.out.println("运行结束");
    }
}
~~~



### 实现Callable接口

**需结合线程池使用**

~~~java
public class MyCallable implements Callable<Integer> {
    @Override
    public Integer call() throws Exception {
        // 模拟计算需要一秒
        Thread.sleep(1000);
        return 2;
    }
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        // 使用
        ExecutorService executor = Executors.newCachedThreadPool();
        MyCallable myCallable = new MyCallable();
        Future<Integer> result = executor.submit(myCallable);
        // 注意调用get方法会阻塞当前线程，直到得到结果。
        // 所以实际编码中建议使用可以设置超时时间的重载get方法。
        System.out.println(result.get());
    }
}
~~~

## 线程常用API

- `currentThread`

  返回代码正在那个线程调用的详细信息

- `Thread.currentThread().getThreadGroup()`

  返回当前线程的所属线程组

- `getPriority()`/`setPriority(int)`

  获得当前线程的优先级

  > 设置线程组和线程的优先级不一样时，线程<=线程组

- `getState`

  获得当前线程状态NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED。

- `isAlive`

  判断当前的线程是否处于活动状态

- `sleep`

  在指定的毫秒数内让当前**『正在执行的线程』**暂停执行。

- `getId`

  获取当前线程的唯一标识

## 停止线程

**用到的类**

~~~java
public static class T1 extends Thread {
        private boolean flag = true;
        private int i =0;
        @Override
        public void run() {
            while (flag) {
                if (i==5){
                    stopThread();
                }
                super.run();
                System.out.println(String.format("当前执行的线程是：%s，优先级：%d",
                        Thread.currentThread().getName(),
                        Thread.currentThread().getPriority()));
                i++;
            }

        }

        public void stopThread() {
            flag = false;
        }
    }
~~~



### 使用退出标志

使线程正常退出，也就是当run方法完成后线程终止。

~~~java
/**
  * 测试使用标志停止线程
  */
@Test
public void testStopThread() throws InterruptedException {
    T1 t = new T1();
    t.start();
    Thread.sleep(2000L);
    t.stopThread();
}
~~~



### `stop`方法强制结束

该方法已废弃。如果强制让线程停止有可能使一些清理性的工作得不到完成。另外一个原因是对锁定的对象进行**『解锁』**，导致数据得不到同步的处理，出现数据不一致性的问题。

### `interrupt`方法中断线程

```java
@Test
public void testStopThreadWithInterrupt() {
    T1 t = new T1();
    t.start();
    t.interrupt();
}
```

>1. 调用`interrupt`方法不会真正的结束线程，只是在当前线程中打上一个**【停止】**的标记。Thread类提供了interrupted方法（`Thread.interrupted()`）测试当前线程是否中断，`isInterrupted`方法测试线程是否已经中断。
>
>2. 如果线程中有sleep代码，不管是否进入到sleep状态，调用了`interrupt`方法都会产生异常。

~~~java
@Test
    public void testStopThreadWithInterrupt() {
        T1 t = new T1();
        t.start();
        t.interrupt();
        boolean interrupted1 = t.isInterrupted();
        System.out.println("第一次调用t.interrupt()方法\n" + t.getName() + "中断了吗 " + interrupted1);
        boolean interrupted2 = t.isInterrupted();
        System.out.println("第二次调用t.interrupt()方法\n" + t.getName() + "中断了吗 " + interrupted2);
        boolean b = Thread.interrupted();
        System.out.println(Thread.currentThread().getName() + " 中断了吗 " + b);
    }
~~~

> 1. `t.isInterrupted()`方法是检查【T1】是否被打上停止的标记
>
> 2. 连续两次调用`t.isInterrupted()`方法，线程的中断状态由方法清除，第二次将返回false。
>
> 3. `Thread.interrupted()`方法是检查**【主线程】**是否被打上停止的标记。

## 暂停线程

#### `suspend`

1. 暂停线程使用`suspend`，重启线程使用`resume`

2. 如果在一个同步方法`synchronized`同步块中使用`suspend`暂停线程，其他线程无法访问该方法。
3. `suspend`下`println`不会释放同步锁
4. `suspend`会造成共享对象数据不同步。

#### `yield`

`yield`方法的作用是放弃当前的CPU资源，将资源让给其它的任务去占用CPU执行。但放弃时间不确定，有可能刚刚放弃，马上又获取CPU时间片。

## 守护线程

在Java线程中有两种线程，一种是用户线程，另一种是守护线程。

守护线程是一种特殊的线程，特殊指的是当进程中不存在用户线程时，守护线程会自动销毁。典型的守护线程的例子就是垃圾回收线程，当进程中没有用户线程，垃圾回收线程就没有存在的必要了，会自动销毁。

~~~java
T1 t = new T1();
t.setDaemon(true);//设置线程为守护线程,需在start()方法执行前设置
t.start();
~~~

# 线程的同步机制

## `synchronized`同步方法

多个线程同时访问下：

1. 局部变量（如方法内的变量）线程安全

2. 成员变量非线程安全
3. `synchronized`锁的是整个对象(可能为这个对象的实例(this/非 static方法)，也可能就是这个对象(this.getClass/static方法))

4. 支持锁重入。若调用的`synchronized`方法里面可以调用另一个`synchronized`方法。

   可重入锁也支持父子类继承的环境中。

   ~~~java
   	/**
        * 测试synchronized的锁重入特性
        */
       @Test
       public void testSynchronized(){
           Thread thread = new T2();
           thread.start();
       }
       class T2 extends Thread{
           @Override
           public void run() {
               /*TestService service = new TestService();
               service.foo1();*/
               System.out.println("测试父子类继承");
               TestServiceB serviceB = new TestServiceB();
               serviceB.foo();
           }
       }
       class TestService{
           synchronized public void foo1(){
               System.out.println("foo1方法");
               foo2();
           }
           synchronized public void foo2(){
               System.out.println("foo2方法");
               foo3();
           }
           synchronized public void foo3(){
               System.out.println("foo3");
           }
       }
   //子类
    class TestServiceB extends TestService{
           synchronized public void foo(){
               System.out.println("foo方法");
               super.foo1();
           }
       }
   ~~~

5. 锁的自动释放

   当一个线程执行的代码出现了异常，其所持有的锁会自动释放。

6. 同步不具有继承性。`synchronized`特性不会从父类的方法继承。

## `synchronized`同步语句块

一个方法内，可能部分逻辑不需要同步，可以只用`synchronized`包围需要同步的代码块。

`synchronized(this){...}`锁定的是当前实例对象，`synchronized(this.getClass())`锁定的是当前Class对象

可以用`synchronized`锁定任意对象，这样其他线程也可以执行。

~~~java
public static void main(String[] args){
    DemoService service = new DemoService();
    Thread thread1 = new DemoThread(service);
    thread1.start();
    Thread thread2 = new DemoThread(service);
    thread2.start();
}
class DemoService{
    //锁定任意对象
    private Object lockObject = new Object();

    public void foo(){
        try {
            //不会争foo2()的this锁
            synchronized (lockObject) {
                System.out.println(Thread.currentThread().getName() + "开始于" + System.currentTimeMillis());
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + "结束于" + System.currentTimeMillis());
            }
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
    
    synchronized public void foo2(){
        System.out.println("foo2方法开始" + System.currentTimeMillis());
        System.out.println("foo2方法结束" + System.currentTimeMillis());
    }
}
class DemoThread extends Thread{
    private DemoService service;
    public DemoThread(DemoService service){
        this.service = service;
    }

    @Override
    public void run() {
        service.foo();
    }
}
~~~

### 避免使用synchronized(String)

避免使用字符串类型作为对象锁，而应该使用其它类型，例如new Object()，对象就不会放入缓存 。

### synchronized无限等待

使用`synchronized`锁住任意对象即可。使线程可以访问其他的同步方法。

~~~java
 private Object lockObject1 = new Object();
    /*synchronized */public void foo1(){
        synchronized (lockObject1) {
            System.out.println("foo1方法开始执行");
            boolean isContinue = true;
            while (isContinue) {

            }
            System.out.println("foo1方法执行结束");
        }
    }
    private Object lockObject2 = new Object();
    /*synchronized */public void foo2(){
        synchronized (lockObject2) {
            System.out.println("foo2方法开始执行");
            System.out.println("foo2方法执行结束");
        }
    }
~~~

### 死锁

产生死锁的必要条件：

1. 互斥条件：线程要求对所分配的资源进行排它性控制，即在一段时间内某个资源仅为一个线程所占用。

2. 请求和操持条件：当线程因请求资源而阻塞时，对已经获得的资源保持不放。

3. 不剥夺条件：线程已经获得的资源在未使用之前，不能剥夺，只能在使用完时由自己来释放。

4. 环路等待条件：在发生死锁时，必然存在一个线程等待另一个线程的环形链(lock1=>lock2=>lock1)

### 锁对象内容的改变

~~~java
class User{
    private String username;
    private String addr;
}
class DemoService{
    public void foo(User user){
          try {
            synchronized (user) {
                System.out.println(Thread.currentThread().getName() + "开始" + System.currentTimeMillis());
                user.username = "b";
                user.password = "bb";
                Thread.sleep(2000);
                System.out.println(Thread.currentThread().getName() + "结束" + System.currentTimeMillis());
            }

        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }
}
~~~

> 只要对象不变，即使对象的属性被改变，synchronized依然有效

## volatile

使变量可以在多个线程可见。

~~~java
public class Demo26 {
    public static void main(String[] args) throws InterruptedException {
        Demo26Service service = new Demo26Service();
        service.start();
        Thread.sleep(100);
        System.out.println(Thread.currentThread().getName() + "准备停止foo方法的循环");
        service.flag = false;
    }
}

class Demo26Service extends Thread{
    // 标志，控制循环
    volatile public boolean flag = true;

    public void run(){
        foo();
    }

    public void foo(){
        System.out.println("foo开始运行");
        while(flag) {

        }
        System.out.println("foo运行结束");
    }
}
~~~



启动线程后，变量`public boolean flag=true`存在在公共堆栈及线程的私有堆栈中。当使用服务器模式时为了线程运行的效率，线程一直在私有堆栈中取得flag的值一直都是true。当执行`service.flag=false`时，虽然被执行，更新的却是在公共堆栈中的flag，所以一直是死循环的状态。使用volatile修饰成员变量后，强制虚拟机从公共堆栈中获取变量的值。



# 线程间的通信

## wait与notify

### wait

wait方法的作用是使当前正在执行的线程进入等待状态，wait方法是Object类的方法，该方法用来将当前线程放入到『预执行队列』中，并且在wait所在的代码行进行停止执行，直到接到通知或被中断为止。在调用wait方法之前，线程必须获取得该对象的对象锁，也就是说只能在同步方法或同步代码块中调用wait方法。如果在执行wait方法后，当前线程锁会自动释放。当wait方法返回线程与其它线程**重新竞争获得锁**。

`wait(long ms)`等待某一时间内是否有线程对象锁进行唤醒，如果超过这个等待时间线程会自动唤醒。

### notify

在执行notify方法后，当前线程**不会马上释放该对象锁**，**wait状态的线程也不能马上获取取该对象锁**，要等到执行notify方法的线程将**任务执行完成**后，也就是退出synchronize代码块后，当前线程才会释放锁，wait状态线程才可以获取到锁。

调用`notify`方法一次只随机通知一个线程进行唤醒，可以使用`notifyAll()`唤醒全部线程。

### wait方法遇到intterrupt方法

当线程呈wait状态时，调用线程对象的interrupt方法会产生InterruptedException异常。

## join

一个线程等待另一个线程执行完之后继续执行。可以使用`join(long ms)`设置等待时间

## ThreadLocal

每个线程拥有自己的共享变量，通过set和get方法来设置和获取。

# Lock

## ReentrantLock

简单使用

~~~java
//获取锁
private Lock lock = new ReentrantLock();
//公平锁 true 非公平锁false 构造函数传递,默认非公平锁
private Lock fairLock = new ReentrantLock(true);
//获取condition
private Condition condition = lock.newCondition();
lock.lock();//加上同步锁
lock.unlock();//解开同步锁
condition.await();//Object.wait();
condition.singnal();//Object.notify();
condition.singnalAll();//Object.notifyAll();

~~~

### 实现线程的顺序执行

~~~java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class OrderDemo {
    private static Lock lock = new ReentrantLock();
    private static Condition conditionA = lock.newCondition();
    private static Condition conditionB = lock.newCondition();
    private static Condition conditionC = lock.newCondition();
    volatile private static int nextPrintWho = 1;

    public static void main(String[] args) {
        Thread t1 = new Thread(){
            @Override
            public void run() {
                try{
                    lock.lock();
                    while (nextPrintWho != 1){
                        conditionA.await();
                    }
                    for (int i = 0; i < 3; i++) {
                        System.out.println("ThreadA打印第" + (i + 1) + "次");
                    }
                    nextPrintWho = 2;
                    conditionB.signalAll();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }

            }
        };
        Thread t2 = new Thread(){
            @Override
            public void run() {
                try{
                    lock.lock();
                    while (nextPrintWho != 2){
                        conditionB.await();
                    }
                    for (int i = 0; i < 3; i++) {
                        System.out.println("ThreadB打印第" + (i + 1) + "次");
                    }
                    nextPrintWho = 3;
                    conditionC.signalAll();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }

            }
        };
        Thread t3 = new Thread(){
            @Override
            public void run() {
                try{
                    lock.lock();
                    while (nextPrintWho != 3){
                        conditionC.await();
                    }
                    for (int i = 0; i < 3; i++) {
                        System.out.println("ThreadC打印第" + (i + 1) + "次");
                    }
                    nextPrintWho = 1;
                    conditionA.signalAll();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }finally {
                    lock.unlock();
                }

            }
        };

        Thread[] threadsA = new Thread[5];
        Thread[] threadsB = new Thread[5];
        Thread[] threadsC = new Thread[5];
        for (int i = 0; i < threadsA.length; i++) {
            threadsA[i] = new Thread(t1);
            threadsB[i] = new Thread(t2);
            threadsC[i] = new Thread(t3);

            threadsA[i].start();
            threadsB[i].start();
            threadsC[i].start();
        }
    }
}
~~~

## ReentrantReadWriteLock

读写锁表示有两个锁，一个是读操作也称作共享锁，另一个是写操作也叫排他锁。也就是多个读锁之间不互斥，读锁与写锁是互斥，写锁与写锁之间也是互斥的。在没有线程进入写入操作时，进行读取操作的多个线程都可以获取 读取锁，而进行写入操作时只有在获取写锁后才能进行写入操作。即**可以有多个线程同时进行读取操作，但同一时刻内只能有一个线程进行写操作**。