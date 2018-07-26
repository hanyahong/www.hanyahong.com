---
title: 【微服务】之三：从零开始，轻松搞定SpringCloud微服务-配置中心
date:  2017-12-07 13:58
tags:
- 微服务
categories:
- Spring Cloud
---

>在整个微服务体系中，除了注册中心具有非常重要的意义之外，还有一个注册中心。注册中心作为管理在整个项目群的配置文件及动态参数的重要载体服务。Spring Cloud体系的子项目中，Spring Cloud Config子项目就是该注册中心。在整个分布式框架系统中，充当重要角色。

<!--more-->

## 官方解释

Spring Cloud provides tools for developers to quickly build some of the common patterns in distributed systems (e.g. configuration management, service discovery, circuit breakers, intelligent routing, micro-proxy, control bus, one-time tokens, global locks, leadership election, distributed sessions, cluster state). Coordination of distributed systems leads to boiler plate patterns, and using Spring Cloud developers can quickly stand up services and applications that implement those patterns. They will work well in any distributed environment, including the developer's own laptop, bare metal data centres, and managed platforms such as Cloud Foundry.

## 本系列博文目录

[【微服务】从零开始，轻松搞定SpringCloud微服务目录 ](http://www.jianshu.com/p/c0168b369cd6)

说明：本系列源码持续更新，开始本篇之前先了解前面几篇文章。
## 开始起飞

基本思路：本文采用Git仓库作为配置文件的存放地址，通过创建一个配置中心服务器启动服务，然后再通过创建一个配置中心的客户端进行测试是否正常运转。

### 创建配置中心仓库

在原有的父类项目下创建一个普通的子项目，可以删除无关的文件，只留下空白项目。然后再创建一个测试的配置文件。
![image.png](http://upload-images.jianshu.io/upload_images/3565867-25f32db74b9be57b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

配置文件中加入测试数据
``` yml
#随意设置的一个参数
myblog:
  name: 千万之路刚开始-author-hyh
  url: http://www.hanyahong.com
  location: BeiJing
```
### 创建配置中心服务端

#### 创建子项目

![image.png](http://upload-images.jianshu.io/upload_images/3565867-d0cef6178d0af2f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### POM文件配置
在pom.xml文件中做一下配置
``` xml

<dependencies>
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-config-server</artifactId>
   </dependency>

   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-test</artifactId>
       <scope>test</scope>
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
#### 配置项目resource配置文件

**注**：整个博客是对各个子项目整合，因此加入了服务注册中心的相关配置

在resources文件夹下创建application.yml文件。并加入以下配置：
``` xml
#服务端口
server:
  port: 8082

#服务注册中心配置
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8081/eureka/

#spring设置
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/hanyahong/spring-cloud-microservice.git
          searchPaths: cloud-hyh-config-repo


```

#### 创建主方法类
在创建完包以后，创建主方法。
``` java

@EnableDiscoveryClient
@SpringBootApplication
@EnableConfigServer
public class ConfigApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigApplication.class, args);
    }
}
```

至此，服务端配置完毕，启动服务器即可，等待客户端验证。

### 创建客户端
创建一个配置客户端，对刚刚的服务进行测试。
@EnableDiscoveryClient:  服务发现客户端注解，用于被发现。
@EnableConfigServer: 开启配置中心服务器配置。
#### 创建客户端子项目
![image.png](http://upload-images.jianshu.io/upload_images/3565867-f62fbe22fbdcdd4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 配置pom文件
在子项目pom.xml中加入一下依赖及插件。
```xml
 <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
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

#### 创建配置文件
在子项目中resources文件夹下，创建bootstrap.yml文件。加入一下配置。
``` yml
spring:
  application:
    #本项目名称
    name: config-client
  cloud:
    config:
      #配置中心服务器地址配置
      uri: http://localhost:8082/
      profile: default
      label: master
      retry:
        # 配置重试次数，默认为6
        max-attempts: 6
        # 间隔乘数 默认1.1
        multiplier: 1.1
        # 初始重试间隔时间，默认1000ms
        initial-interval: 1000
        # 最大间隔时间，默认2000ms
        max-interval: 2000

server:
  port: 8091
#服务发现配置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8081/eureka/
```

#### 创建程序入口
创建默认包以后创建ConfgClientApplication.java 文件。
```java
/** 
 * @Description :配置中心启动类
 * @Author hanyahong
 * @Date 2017/12/6- 14:06
 */


@SpringBootApplication
public class ConfigClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigClientApplication.class, args);
    }
}

```

#### 创建测试API
创建一个测试API文件，TestApi.java。
```java
/**
 * @Description :配置中心-客户端展示API
 * @Author hanyahong
 * @Date 2017/12/6- 16:39
 */
@RefreshScope
@RestController

public class TestApi {

    @Value("${myblog.name}")
    private String name;
    @Value("${myblog.url}")
    private String url;
    @Value("${myblog.location}")
    private String location;
    @RequestMapping("/blog-info")
    public String getBlogInfo() {
        return "从Github仓库中获取得到我博客信息：【"+location+","+","+url+","+name+"】";
    }
}
```
@RefreshScope:开启刷新

至此，配置中心测试的客户端基本完毕。
对于子项目来说有三个子项目：
**cloud-hyh-config**  端口号：8082
**cloud-hyh-config-client** 端口号：8091
**cloud-hyh-config-repo** 纯存储使用，该文档下面的配置文件一定要上传到仓库后，才可以远程获取。

### 启动项目并测试

对服务注册中心（上篇有写）、服务配置中心、服务客户端分别进行启动。
可以通过注册中心查看是否都已经被纳入管理。
![image.png](http://upload-images.jianshu.io/upload_images/3565867-8fcbb0c27fd5e3bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**测试一：** 注册中心测试
首先通过访问配置中心服务器的地址可以进行测试是否获取成功。
访问 http://localhost:8082/config-client.yml 对仓库中资源文件 测试是否返回结果。
另外也可以通过 http://localhost:8082/config-client/master 进行访问。浏览器显示返回结果：
``` 
"name":"config-client","profiles":["master"],"label":null,"version":"7169e90f628c85d582f3f9d5fceda36696ebd751","state":null,"propertySources":[{"name":"https://github.com/hanyahong/spring-cloud-microservice.git/cloud-hyh-config-repo/config-client.yml","source":{"myblog.name":"千万之路刚开始-author-hyh","myblog.url":"http://www.hanyahong.com","myblog.location":"BeiJing","config-client.name":"test"}}]}
```
**测试二：** 客户端访问API测试
通过客户端访问API  http://localhost:8091/blog-info 显示结果：
``` 从Github仓库中获取得到我博客信息：【BeiJing-Customs,,http://www.hanyahong.com,千万之路刚开始-author-hyh】```
测试成功！

**测试三：** 动态更新参数测试
配置中心一个重要的功能就是你无须重启去生效一些参数配置，系统可以通过访问/refresh 进行动态刷新，将参数生效。
1. 修改配置文件信息，上传git仓库。
2. 使用PostMan 或其他工具进行一次POST请求 API:``` http://localhost:8091/refresh ```   （一定要看清楚，POST请求，浏览器直接访问无效，会报Request method 'GET' not supported 错误）。
3.  再一次访问 http://localhost:8091/blog-info ，可以看到已在未重启的情况下，配置动态更新。
	
### 后续说明
因配置中心涉及很多数据的更新，不可能每次通过这种方式去动态更新，后续会有专门消息总线模块的讲解，将通过消息总线的机制去进行配置的传输。

## 源码地址

GitHub：https://github.com/hanyahong/spring-cloud-microservice
