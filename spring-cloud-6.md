---
title: 【微服务】之六：轻松搞定SpringCloud微服务-API网关zuul
date: 2017.12.24 10:51
tags:
	- 微服务
categories:
	- Spring Cloud
---

>通过前面几篇文章的介绍，我们可以轻松搭建起来微服务体系中比较重要的几个基础构建服务。那么，在本篇博文中，我们重点讲解一下，如何将所有微服务的API同意对外暴露，这个就设计API网关的概念。

<!--more-->

## 本系列教程
[【微服务】轻松搞定SpringCloud微服务目录](http://www.hanyahong.com/categories/Spring-Cloud/)
本系列为连载文章，阅读本文之前强烈建议您先阅读前面几篇。


## 网关名称解释
网关(Gateway)又称网间连接器、协议转换器。网关在网络层以上实现网络互连，是最复杂的网络互连设备，仅用于两个高层协议不同的网络互连。网关既可以用于广域网互连，也可以用于局域网互连。 网关是一种充当转换重任的计算机系统或设备。使用在不同的通信协议、数据格式或语言，甚至体系结构完全不同的两种系统之间，网关是一个翻译器。
## API网关名词解释
API Gateway（API GW / API 网关），顾名思义，是出现在系统边界上的一个面向API的、串行集中式的强管控服务，这里的边界是企业IT系统的边界。
在微服务流行之前，API GW的实体就已经诞生了，这时的主要应用场景是OpenAPI，也就是开放平台，面向的是企业外部合作伙伴，对于这个应用场景，相信接触的人会比较多。当在微服务概念流行起来之后，API网关似乎成了在上层应用层集成的标配组件。
## Spring Cloud Zuul 简介

Zuul 是Netflix 提供的一个开源组件,致力于在云平台上提供动态路由，监控，弹性，安全等边缘服务的框架。也有很多公司使用它来作为网关的重要组成部分。Spring Cloud 体系收录的该模块，主要用于提供动态路由、监控、安全控制、限流配额等，可以将内部微服务API同意暴露。

## 博文系统架构
根据我们微服务整体的一个架构设计，本系列博文也主要是对整个微服务架构进行落地示范。通过一组架构比较可以更加深入理解网关的概念。
### 传统互联网架构
在没有微服务的架构体系当中，我们一般使用Nginx作为负载分发、反向代理，形成一个API网关的一个构建。架构图如下图所示：
![image.png](http://upload-images.jianshu.io/upload_images/3565867-a4967e589f2ae477.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 微服务下网关模式
在spring cloud 体系当中，我们将内部的服务全部进行隐藏，对外只有一个对外暴露的机制，这就是spring cloud zuul 网关。架构图如下所示：
![image.png](http://upload-images.jianshu.io/upload_images/3565867-f8825ee11eadcd6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果我们在深入一些，去兼容一些以前公司旧的非微服务系统也可以是这样：
![image.png](http://upload-images.jianshu.io/upload_images/3565867-942f025b0c050593.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 开始起飞

同样，起飞之前，建议阅读前面几篇文章，方便代码理解与使用。
### 创建网关子项目
我们在原来的父类项目下创建一个子项目。（可以参考我GitHub代码结构）
![image.png](http://upload-images.jianshu.io/upload_images/3565867-3b2df1b9ac549313.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 配置POM文件
```xml
<dependencies>
 <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-zuul</artifactId>
 </dependency>
 <dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-eureka</artifactId>
 </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```
#### 创建配置文件application.yml
resources 文件夹下面创建文件application.yml,配置清单如下：
```yml
#服务注册中心配置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8081/eureka/
  instance:
    appname: api-gateway

#设置网关端口号
server:
  port: 8080

spring:
  application:
    name: api-getway
```

#### 创建程序入口application.java

为了方便，命名为GatewayApplication
```java
/**
 * Created by Administrator on 2017/12/17.
 * 网关启动入口
 */
@SpringBootApplication
@EnableZuulProxy
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}

```

**@EnableZuulProxy** 注解说明：@EnableZuulProxy简单理解为@EnableZuulServer的增强版，当Zuul与Eureka、Ribbon等组件配合使用时，我们使用@EnableZuulProxy。



#### 启动并测试
从上面的配置中，我们已经完成了基本的网关配置，那么我们就来做一个简单的测试。
本次测试选择我微服务项目群中有三个服务：
分别是：

| 子项目名称 | 服务名称 | 端口号 | 源码 
| --- | :---  | :-  | :-: |
| cloud-hyh-service-1 |cloud-service |8071 | github |
| cloud-hyh-discovery-eureka | discovery-service | 8081 | github |
| cloud-hyh-api-gateway-zuul | api-gateway | 8080 | github |

##### 启动子服务
分别启动注册中心、测试服务、api网关三个服务。我们可以观察服务注册中心已经可以看到其他两个服务会被注册。
![image.png](http://upload-images.jianshu.io/upload_images/3565867-cdd07b4e400fb263.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 访问API测试
我们在cloud-service子项目中，之前已经做过一个测试的API,

子微服务|地址|路径|请求方式|
----|----|--|--
cloud-service|http://localhost:8071|/ribbon/name|GET


在没有网关之前，我们外部如果想要调用服务的花，就可以通过微服务自身的地址访问。但是有了API网关之后，我们可以将我们的服务地址对外暴露出去。

子微服务|地址|路径|请求方式|
----|----|--|--
cloud-service|http://localhost:8080|/ribbon/name|GET

访问地址： 网关地址/服务名/访问地址/
http://localhost:8080/cloud-service/ribbon/name

测试成功
浏览器中返回：
**千万之路刚开始-www.hanyahong.com-beijing该服务器端口8071**
****
说明：针对API网关还有很多参数需要配置，例如持久化、负载分发等。针对这一块我们后续还会进行专题细致深入。

## 源码

本文出处：[http://www.hanyahong.com/](http://www.hanyahong.com/)
Github源码：[https://github.com/hanyahong/spring-cloud-microservice ](https://github.com/hanyahong/spring-cloud-microservice )
转发请注明出处！
