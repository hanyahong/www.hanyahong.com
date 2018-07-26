---
title: 【微服务】之四：轻松搞定SpringCloud微服务-负载均衡Ribbon
date: 2017-12-09 11:43
tags:
	- 微服务
categories:

	- Spring Cloud
---
> 对于任何一个高可用高负载的系统来说，负载均衡是一个必不可少的名称。在大型分布式计算体系中，某个服务在单例的情况下，很难应对各种突发情况。因此，负载均衡是为了让系统在性能出现瓶颈或者其中一些出现状态下可以进行分发业务量的解决方案。在SpringCloud 体系当中，加入了Netflix公司的很多优秀产品，其中一个就是针对于服务端进行负载均衡的Ribbon。

<!--more-->


[【微服务】轻松搞定SpringCloud微服务目录](http://www.cnblogs.com/hyhnet/p/7998751.html)
本系列为连载文章，阅读本文之前强烈建议您先阅读前面几篇。
## 相关简介
### 负载均衡简介
**负载均衡**：英文名称为**Load Balance**， 建立在现有网络结构之上，它提供了一种廉价有效透明的方法扩展网络设备和服务器的带宽、增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性。其意思就是分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。
负载均衡带来的好处很明显：

### Ribbon简介
Ribbon是Netflix开源的一款用于客户端软负载均衡的工具软件。Spring Cloud对Ribbon进行了一些封装以更好的使用Spring Boot的自动化配置理念。
### Spring Cloud Ribbon 简介
Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端负载均衡的工具。它是一个基于HTTP和TCP的客户端负载均衡器。它可以通过在客户端中配置ribbonServerList来设置服务端列表去轮询访问以达到均衡负载的作用。
## 开始起飞
起飞之前，先说明一下，本项目前几篇文章中已经构建了相关子项目包括：注册中心、配置中心。本文中继续可以使用。
###  创建两个服务器
需要创建两个一模一样的服务器，让客户端按照不同的机制进行分发，达到负载均衡的效果。我们约定两个子项目名称：
**cloud-hyh-service-1**  端口号：8071
**cloud-hyh-service-2**  端口号：8072
对于服务名称设置一样：**cloud-service** ，其他业务都一样，可以复制。【端口号不一样】
#### pom.xml文件配置
``` xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
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

#### 服务器一参数配置
``` yml
#服务注册中心配置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8081/eureka/
  instance:
    appname: cloud-service
    lease-renewal-interval-in-seconds: 1

server:
  port: 8071

spring:
  application:
    name: cloud-service


```

####  服务器二参数配置
``` yml
#服务注册中心配置
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8081/eureka/
  instance:
    appname: cloud-service

server:
  port: 8072

spring:
  application:
    name: cloud-service
```
**说明**：与配置一其实基本一样，只不过将端口号配置成 8072

#### 服务器入口配置Application.yml

```java
/**
 * @Description : 
 * @Author hanyahong
 * @Date 2017/12/7- 17:35
 */
@SpringBootApplication
@EnableDiscoveryClient
public class ServiceTwoApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceTwoApplication.class, args);
    }
}
```
####  新建测试API类
``` java
/**
 * @Description :测试RibbonTest API
 * @Author hanyahong
 * @Date 2017/12/7- 17:40
 */
@RestController
@RequestMapping(value = "/ribbon")
public class RibbonTestApi {

    /**
     * 获取博客名称API
     *
     * @return 相关信息
     */
    @RequestMapping(value = "name", method = RequestMethod.GET)
    public String getMyBlogNameApi() {
        return "千万之路刚开始-www.hanyahong.com-beijing"+"该服务器端口号：8071";
    }
}


备注：两台服务器，除了返回的服务器端口号 8071 8072不同之外，其他都相同，就是为了看到效果。
```

### 创建测试客户端

创建一个子项目，cloud-hyh-ribbon-client ,主要用来测试ribbon客户端负载。

#### pom文件配置

在pom文件中加入以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
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

#### 配置文件application配置

```yml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8081/eureka/
  instance:
    appname: ribbon-client

server:
  port: 8092

spring:
  application:
    name: ribbon-client
```

#### 配置子项目启动类

``` java
/**
 * @Description :启动类，示范负载均衡服务器
 * @Author hanyahong
 * @Date 2017/12/7- 17:00
 */
@SpringBootApplication
@EnableDiscoveryClient
public class RibbonServiceApplication {

    public static void main(String[] args) {

        SpringApplication.run(RibbonServiceApplication.class, args);
    }

    /**
     * Spring提供的用于访问Rest服务的客户端
     * @return
     */
    @Bean
    @LoadBalanced
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```
说明：
>**RestTemplate**是Spring提供的用于访问Rest服务的客户端。RestTemplate提供了多种便捷访问远程Http服务的方法，能够大大提高客户端的编写效率。调用RestTemplate的默认构造函数，RestTemplate对象在底层通过使用java.net包下的实现创建HTTP 请求，可以通过使用ClientHttpRequestFactory指定不同的HTTP请求方式。
ClientHttpRequestFactory接口主要提供了两种实现方式，一种是SimpleClientHttpRequestFactory，使用J2SE提供的方式（既java.net包提供的方式）创建底层的Http请求连接，还有一种方式是使用HttpComponentsClientHttpRequestFactory方式，底层使用HttpClient访问远程的Http服务，使用HttpClient可以配置连接池和证书等信息。

**@LoadBalanced** 注解加在RestTemplate上面，这个注解会自动构造LoadBalancerClient接口的实现类并注册到Spring容器中。

#### 创建接口API

```java
/**
 * @Description : 测试客户端负载均衡的接口API
 * @Author hanyahong
 * @Date 2017/12/7- 18:01
 */
@RestController
@RequestMapping(value = "/test")
public class TestRibbonApi {
    /**
     * 注入RestTemplate
     */
    @Autowired
    RestTemplate restTemplate;


    @RequestMapping(value = "/blog/name" ,method = RequestMethod.GET)
    public String testGetNameOfBlog(){
        String url="http://CLOUD-SERVICE/ribbon/name";
        return restTemplate.getForObject(url,String.class);
    }
}
```
注意：这个代码中 url 设置的是 上面提到的服务器的服务名。

### 启动项目群进行测试

经过全面的配置，服务器全面配置完毕，包括一个注册中心、一个配置中心、两个相同配置的服务器、一台测试客户端负载均衡的测试服务器。
启动成功以后会在注册中心看到。
![image.png](http://upload-images.jianshu.io/upload_images/3565867-058efd484f6769a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过访问客户端地址：http://localhost:8092/test/name 就可以访问。效果如下：
![image.png](http://upload-images.jianshu.io/upload_images/3565867-08aecadef807486f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
刷新一次：

![image.png](http://upload-images.jianshu.io/upload_images/3565867-1e35fb73a783a1bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此所有配置成功。测试结果也成功。

## 本文源码

Github源码：https://github.com/hanyahong/spring-cloud-microservice
