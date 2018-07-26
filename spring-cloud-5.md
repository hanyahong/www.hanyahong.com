---
title: 【微服务】之五：轻松搞定SpringCloud微服务-调用远程组件Feign
date: 2017.12.10 13:25
tags:
	- 微服务
categories:
	- Spring Cloud
---

>上一篇文章讲到了负载均衡在Spring Cloud体系中的体现，其实Spring Cloud是提供了多种客户端调用的组件,各个微服务都是以HTTP接口的形式暴露自身服务的，因此在调用远程服务时就必须使用HTTP客户端。我们可以使用JDK原生的URLConnection、Apache的Http Client、Netty的异步HTTP Client, Spring的RestTemplate。但是，用起来最方便、最优雅的还是要属Feign了。今天这一篇文章是系列教程中第五篇，也是对负载均衡的第二篇，主要对Spring Cloud Feign的实战。

<!--more-->

## 本系列教程
[【微服务】轻松搞定SpringCloud微服务目录](http://www.cnblogs.com/hyhnet/p/7998751.html)
本系列为连载文章，阅读本文之前强烈建议您先阅读前面几篇。


## Feign简介
Feign是一种声明式、模板化的HTTP客户端，也是netflix公司组件。使用feign可以在远程调用另外服务的API,如果调用本地API一样。
我们知道，阿里巴巴的doubbo采用二进制的RPC协议进行底层通讯，客户端可以使用类似本地方法一样调用。那么，虽然Feign同样可以有这种效果，但是底层还是通过HTTP协议调取restful的API的方式。
通过Feign， 我们能把HTTP远程调用对开发者完全透明，得到与调用本地方法一致的编码体验。

## Spring Cloud Feign

## 开始起飞
起飞之前，先说明一下，本篇文章还是会在之前文章的源码继续。因此同样需要服务发现中心、服务配置中心、服务、客户端等微服务，因此沿用之前的项目中的子项目。

###  创建客户端服务

为了方便，可以直接复制上一片文章中的**cloud-hyh-ribbon-test-client** 子项目。并命名为新子项目。

#### 配置客户端Pom文件
```xml
<dependencies>
	<!--服务发现eureka组件 依赖-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <!--web支持 依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--测试启动组件-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
    </dependency>
    <!--feign依赖 配置-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-feign</artifactId>
    </dependency>
</dependencies>
<!--构建-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

#### 配置文件配置
对resources文件下配置文件进行配置。

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8081/eureka/
  instance:
    appname: feign-client

server:
  port: 8093

spring:
  application:
    name: ribbon-client
```
#### 启动类编写

创建子包以后，创建主类FeignApplication.java

```java

/**
 * @Description :启动类，示范feign
 * @Author hanyahong
 * @Date 2017/12/10- 07:00
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class FeignClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(FeignClientApplication.class, args);
    }
}
```

**说明：** 
@EnableFeignClients ：该注解是开启feign的扫描。

#### 创建服务层

创建文件夹service ,然后创建TestService.java。
```java
/**
 * @Description :测试feign接口服务
 * @Author hanyahong
 * @Date 2017/12/10- 12:43
 * <p>
 * 通过在本服务的service层直接调用其他服务的api层的API接口进行相关操作。
 * cloud-service 是一个简单服务，测试返回一个获取博客名称的接口。
 */

@FeignClient("cloud-service")
public interface TestService {

    /**
     * 调取博客名称
     *
     * @return 博客名称
     */
    @GetMapping("/ribbon/name")
    String getBlogName();
}
```
说明：
@FeignClient 注解：就是你要调用的微服务接口所在的服务名。
比如A服务想调用B服务的服务。就要填写B服务的服务名。
 @GetMapping注解：就是要调用的接口rest资源路径。
#### API层接口
我们将创建一个接口，暴露出去，可以调用刚才服务层的这个接口。
```java
/**
 * @Description :博客操作API，对外暴露
 * @Author hanyahong
 * @Date 2017/12/10- 12:53
 */
@RestController
@RequestMapping(value = "/feign")
public class BlogApi {
    /**
     * 注入服务层接口
     */
    @Autowired
    TestService testService;


    /**
     * 通过Feign客户端的方式，调用之前一个服务的API
     *
     * @return
     */
    @RequestMapping("/blogName")
    public String getBlogNameByFeign() {
        return "通过feign方式得到：" + testService.getBlogName();
    }
}
```

### 目录参考
![目录参考.png](http://upload-images.jianshu.io/upload_images/3565867-77f28c9f27edd176.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 启动项目群并测试

启动注册中心、配置中心、两个相同配置服务、还有一个本文创建的客户端cloud-hyh-test-feign.

#### 查看是否启动成功

访问注册中心，查看是否全部启动。
![image.png](http://upload-images.jianshu.io/upload_images/3565867-d30b7d054e4f512d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

已经启动成功！
#### 访问测试
访问地址http://localhost:8093/feign/blogName 路径，查看是否成功返回需要的数据。

结果显示：
>通过feign方式得到：千万之路刚开始-www.hanyahong.com-beijing该服务器端口8072

## 源码

本文出处：http://www.hanyahong.com/
Github源码：https://github.com/hanyahong/spring-cloud-microservice 
转发请注明出处！
