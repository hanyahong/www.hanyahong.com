---
title: '[高并发]Java高并发编程系列第二篇--线程同步'
date: 2017.01.10 01:50:02
tags:
	- 高并发
categories:
	- 高并发教程
---

>高并发,听起来高大上的一个词汇,在身处于互联网潮的社会大趋势下,高并发赋予了更多的传奇色彩.首先,我们可以看到很多招聘中,会提到有高并发项目者优先.高并发,意味着,你的前雇主,有很大的业务层面的需求,而且也能怎么你在整个项目中的一个处理逻辑的能力体现.那么,你真的知道什么是高并发吗?这不是一个很简单的话题.高并发,往往会牵扯到很多的问题,如何才能快速响应,如何处理各个线程之间的交互,如何完成逻辑之间的高负载运转,甚至,一个系统,如果没有做好前期高并发的合理配置,整个产品会遇到瓶颈,以及不可预期的多次后果.
那么本系列博客将重点从最基本的理论基础,线程时间,再到项目实战,讲述,一个高并发系统的完整技术栈.

<!--more-->
本文是JAVA高并发系列的基础篇第二篇--线程同步
本系列博文:
[第一篇:[高并发]Java高并发编程系列开山篇--线程实现](http://www.cnblogs.com/hyhnet/p/6250264.html)

----------

### 一 线程同步基本概述
**同步**: 什么是线程同步,可以简单认为,当有两个以上的线程,需要访问共同的一个资源的时候,我们需要确保每一个线程都能使用到资源.那么问题来了,怎么实现,这就可以使用到我们的这个概念--同步.
同步,其实关键的一点,也就是监视器,它的作用就是监视每一个线程发生的每次动作行为.下面我们看看同步到底怎么去在代码中实现.
### 二  同步实现方式
#### 	实现方式
其实,在JAVA语言中同步是简单的一件事,为什么呢?应为我们可以使用关键字`synchronized` 这个关键字去实现(实现同步的一种).用它来修饰某个方法.
我们可以使用一个小的Demo来验证一下:
**场景描述:**
一个发送者类
一个消息类
一个主方法调用类

本文Github源码:[Github源码](https://github.com/hanyahong/com-hanyahong-blog/tree/master/com-hanyahong-thread-1/src/main/java/com/hyh/synch_1)
##### 清单1_消息类
```java
package com.hyh.synch_1;

/**
 * Created by hyh on 17-1-1.
 * 测试消息类
 * @author hyh
 */
public class Message {

    //发送消息  1.加入synchronized表示同步,不加入表示不同步,可以自己切换体验
    //public  void sendMeg(String meg) {
    public synchronized void sendMeg(String meg) {
        System.out.print("接受消息:" + meg);

        try {
            Thread.sleep(1000);

        } catch (InterruptedException e) {
            System.out.println("异常");
        }
        System.out.println("-----发送成功");
    }
}
```

##### 清单二_发送者类
```java
package com.hyh.synch_1;

/**
 * Created by hyh on 17-1-1.
 * 发送者类
 * @author hyh
 */
public class From implements Runnable {

    //全局变量
    String meg;
    Message message;
    Thread thread;

    //构造函数
    public From(Message message, String meg) {
        this.message = message;
        this.meg = meg;
        thread = new Thread(this);
        thread.start();
    }

    //重写父类函数
    public void run() {
        message.sendMeg(meg);
    }
}
```
##### 清单3_主类:
```java
package com.hyh.synch_1;

/**
 * Created by hyh on 17-1-1.
 * 同步测试类
 * @author hyh
 */
public class Synch {
    public static void main(String[] args) {
        Message message = new Message();
        From from = new From(message, "我的消息:----1");
        From from2 = new From(message, "我的消息:----2");
        From from3 = new From(message, "我的消息:----3");

        try {
            from.thread.join();
            from2.thread.join();
            from3.thread.join();
        } catch (InterruptedException i) {
            System.out.println("异常");
        }
    }
}

```

到此为止,三个类就写完了,那么我们来运行一下.是什么结果.
运行j结果:

    
    接受消息:我的消息:----2-----发送成功
	接受消息:我的消息:----3-----发送成功
	接受消息:我的消息:----1-----发送成功

这里如果,你再取消清单一中的同步关键字再试一试,结果显示,如果奇效,消息出现错乱,不会同步.

### 三 同步进阶
线程同步的方式和机制理论
临界区（Critical Section）、互斥量（Mutex）、信号量（Semaphore）、事件（Event）的区别
**1、临界区：**通过对多线程的串行化来访问公共资源或一段代码，速度快，适合控制数据访问。在任意时刻只允许一个线程对共享资源进行访问，如果有多个线程试图访问公共资源，那么在有一个线程进入后，其他试图访问公共资源的线程将被挂起，并一直等到进入临界区的线程离开，临界区在被释放后，其他线程才可以抢占。
**2、互斥量：**采用互斥对象机制。 只有拥有互斥对象的线程才有访问公共资源的权限，因为互斥对象只有一个，所以能保证公共资源不会同时被多个线程访问。互斥不仅能实现同一应用程序的公共资源安全共享，还能实现不同应用程序的公共资源安全共享
**3、信号量：**它允许多个线程在同一时刻访问同一资源，但是需要限制在同一时刻访问此资源的最大线程数目
**4、事 件：** 通过通知操作的方式来保持线程的同步，还可以方便实现对多个线程的优先级比较的操作
### 四 线程同步的方式
第一种:使用synchronized关键字修饰的方法。
第二种:同步代码块 ,即有synchronized关键字修饰的语句块。
第三种:使用特殊域变量(volatile)实现线程同步
第四种:使用重入锁实现线程同步, ReentrantLock类是可重入、互斥、实现了Lock接口的锁
第五种:使用局部变量实现线程同步 
可以根据不用的功能做不同的实现.

至此(本文完)


原创文字,转发请注明出处:http://www.cnblogs.com/hyhnet/p/6257698.html
**独立站点:** http://www.hanyahong.com
**wechat:** wixf150
