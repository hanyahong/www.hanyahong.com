---
title: '[高并发]Java高并发编程系列开山篇--线程实现'
date: 2017.01.05 01:50:49
tags:
	- 高并发
categories:
	- 高并发教程
---

Java是最早开始有并发的语言之一,再过去传统多任务的模式下,人们发现很难解决一些更为复杂的问题,这个时候我们就有了并发.
**引用**
> &#160; &#160; &#160; &#160;多线程比多任务更加有挑战。多线程是在同一个程序内部并行执行，因此会对相同的内存空间进行并发读写操作。这可能是在单线程程序中从来不会遇到的问题。其中的一些错误也未必会在单CPU机器上出现，因为两个线程从来不会得到真正的并行执行。然而，更现代的计算机伴随着多核CPU的出现，也就意味着不同的线程能被不同的CPU核得到真正意义的并行执行。

<!--more-->

----------
&#160; &#160; &#160; &#160;那么,要开始Java并发之路,就要开始从java线程开始说起.
&#160; &#160; &#160; &#160;本篇幅将是本系列博客的开山篇,也就是基础线程的复习.
### 线程简介
#### 线程百科
		线程，有时被称为轻量级进程(Lightweight Process，LWP），是程序执行流的最小单元。一个标准的线程由线程ID，当前指令指针(PC），寄存器集合和堆栈组成。另外，线程是进程中的一个实体，是被系统独立调度和分派的基本单位，线程自己不拥有系统资源，只拥有一点儿在运行中必不可少的资源，但它可与同属一个进程的其它线程共享进程所拥有的全部资源。
#### 线程优点
资源利用率更好
程序设计在某些情况下更简单
程序响应更快
#### 线程代价
设计更复杂
上下文切换的开销上升
增加资源消耗
### 线程的实现方式
&#160; &#160; &#160; &#160;线程我们有不同的实现方式,生产环境中用到的也有所不同,那么,我们先来通过Demo看一下每种实现方式的区别.
#### 通过Thread 实现的线程
``` java
public class Demo1 {
    public static void main(String args[]) {
        Thread thread = Thread.currentThread();
        System.out.println("当前线程:" + thread);

        thread.setName("hyh thread");//修改线程名称

        System.out.println("修改名称之后:" + thread);

        try {
            for (int a = 5; a > 0; a--) {
                System.out.println(a);

                thread.sleep(1000);
            }
        } catch (Exception e) {
            System.out.println("出现异常");
        }
    }

```
#### 通过集成Runnable接口实现

Demo2.class 清单:
```java
public class Demo2 implements Runnable {

    Thread t;

    //空构造函数
    Demo2() {
        t = new Thread(this, "测试线程");
        System.out.println("子线程" + t);
        t.start();
    }

    public void run() {
        try {
            for (int a = 5; a > 0; a--) {
                System.out.println("子线程" + a);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("异常");
        }
        System.out.println("退出子线程");
    }
}

/**
 * 主类
 */
class ThreadDemo {
    public static void main(String args[]) {
        new Demo2();//创建一个新线程
        try {
            for (int i = 5; i > 0; i--) {
                System.out.println("主线程:" + i);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("主线程异常");
        }
        //主线程退出
        System.out.println("主线程退出");

    }
}
```
#### 通过集成 Thread 重写run方法实现
```java
public class Demo3 extends Thread {
    public Demo3() {
        //创建新线程
        super("线程Demo");
        System.out.println("子线程:" + this);
        start();
    }

    @Override
    public void run() {
        try {
            for (int a = 5; a > 0; a--) {
                System.out.println("子线程:" + a);
                Thread.sleep(500);
            }
        } catch (InterruptedException e) {
            System.out.println("子程序异常");
        }
    }

}

class MasterThread {
    public static void main(String args[]) {
        new Demo3();//创建新线程

        try {
            for (int i = 5; i > 0; i--) {
                System.out.println("主线程:" + i);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("主程序异常");
        }
        System.out.println("主程序退出...");
    }
}
```
### 三个线程的实现
在日常生产中,使用线程可以通过实现Runnable接口.下面我们通过该方法实现简单的三个线程Demo

结构: 创建一个Demo4类,另外一个主类MultThreadDemo.
清单:
```java
 public class Demo4 implements Runnable {
 //全局变量
    String name;
    Thread thread;

    //构造器
    public Demo4(String th) {
        name = th;
        thread = new Thread(this, name);
        System.out.println("新线程" + thread);
        //开始线程
        thread.start();
    }

    //重写run方法
    public void run() {
        try {
            for (int a = 5; a > 0; a--) {
                System.out.println(name + ":" + a);
                Thread.sleep(1000);
            }
        } catch (InterruptedException e) {
            System.out.println("异常");
        }
        System.out.println(name + "线程结束");
    }
}

/**
 * 测试类
 *
 * @author hyh
 */
class MultThreadDemo {

    public static void main(String[] args) {
        //创建三个线程
        Demo4 thread_1 = new Demo4("线程一");
        Demo4 thread_2 = new Demo4("线程二");
        Demo4 thread_3 = new Demo4("线程三");
        //查看状态
        System.out.println("线程一状态:" + thread_1.thread.isAlive());
        System.out.println("线程二状态:" + thread_2.thread.isAlive());
        System.out.println("线程三状态:" + thread_3.thread.isAlive());

        try {
            System.out.println("等待其他线程结束");
            //使用join确保主线程最后运行
            thread_1.thread.join();
            thread_2.thread.join();
            thread_3.thread.join();
        } catch (InterruptedException e) {
            System.out.println("线程异常");

        }
        //查看状态
        System.out.println("线程一:" + thread_1.thread.isAlive());
        System.out.println("线程二:" + thread_2.thread.isAlive());
        System.out.println("线程三:" + thread_3.thread.isAlive());
    }
}
```
到此,开山Demo完成.
本文源码Github地址:
https://github.com/hanyahong/com-hanyahong-blog/tree/master/com-hanyahong-thread-1/src/main/java/com/hyh/thread
### 思考:进程与线程的比较
**进程**
资源分配的基本单位。
所有与该进程有关的资源，都被记录在进程控制块PCB中。
**进程**处理机的调度单位，拥有完整的虚拟地址空间。当进程发生调度时，不同的进程拥有不同的虚拟地址空间，而同一进程内的不同线程共享同一地址空间。
**线程**与资源分配无关，属于某一个进程，并与其他线程共享进程资源。
**线程**只由相关堆栈（系统栈或用户栈）寄存器和线程控制表TCB组成。寄存器可被用来存储线程内的局部变量，但不能存储其他线程的相关变量。
**进程**在多线程中，进程不是一个可执行的实体。

**两者比较：**
**调度和切换：**线程上下文切换比进程上下文切换要快得多。
**通信：**进程间通信IPC，线程间可以直接读写进程数据段（如全局变量）来进行通信——需要进程同步和互斥手段的辅助，以保证数据的一致性。
**地址空间和其它资源（如打开文件）：**进程间相互独立，同一进程的各线程间共享。某进程内的线程在其它进程不可见。

（本篇完）
**ＷeChat:** wixf150
原创文章，转发请注明出处：http://www.cnblogs.com/hyhnet/p/6250264.html
访问独立站点,获得更好用户体验:http://www.hanyahong.com
