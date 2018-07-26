---
title: 【微服务】之二：从零开始，轻松搞定SpringCloud微服务系列--注册中心（一）
date: 2017-12-04 20:02 
tags:
- 微服务
categories:
- Spring Cloud
---

>微服务体系，有效解决项目庞大、互相依赖的问题。目前SpringCloud体系有强大的一整套针对微服务的解决方案。本文中，重点对微服务体系中的服务发现注册中心进行详细说明。本篇中的注册中心，采用Netflix 公司的Eureka。

<!--more-->
本系列教程列表：
[【微服务】之一：从零开始，轻松搞定SpringCloud微服务系列--开山篇（spring boot 小demo）](http://www.jianshu.com/p/46529b6f1489)
【微服务】之二：从零开始，轻松搞定SpringCloud微服务系列--注册中心（一）
## 注册中心简介
Netflix Eureka：云端负载均衡，一个基于 REST 的服务，用于定位服务，以实现云端的负载均衡和中间层服务器的故障转移。他包含很多功能，本文重点讲解它的服务注册中心。

**官方解释**：
Eureka is a REST (Representational State Transfer) based service that is primarily used in the AWS cloud for locating services for the purpose of load balancing and failover of middle-tier servers. We call this service, the Eureka Server. Eureka also comes with a Java-based client component,the Eureka Client, which makes interactions with the service much easier. The client also has a built-in load balancer that does basic round-robin load balancing.

Eureka支持服务动态扩容、缩容、失效剔除。

Eureka提供了完整的Service Registry和Service Discovery实现，与现有Spring Cloud完美融合。

##  注册中心服务原理
![enter image description here](http://upload-images.jianshu.io/upload_images/3565867-3ab1b53a026e62cb?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

由上图可以看出，蓝色部分为注册中心，黄色部分为一个生产者、消费者。所有应用都被同一个注册中心纳入管理。通常有Register（服务注册）、Renew（服务续约）、Cancel（服务下线）等操作。

## 环境清单
JDK: 1.8
Maven:3.x+
IDE:idea
## 开始起飞

为了后续博文的开展，我们约定所有子系统都放置在一个父类项目下，采用Idea的模块开发机制，对所有子系统进行统一仓库管理，方便交流学习。

### 创建父类项目
使用idea创建一个maven项目（创建之前设置好maven、jdk版本）。
然后在pom.xml文件中加入以下超级父类依赖。

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
```



### 创建子模块
![模块开发](http://upload-images.jianshu.io/upload_images/3565867-61b7f9869c33052d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

通过上面箭头提示的Module创建模块项目，流程与新建项目类似。
创建完成以后在子项目pom文件中加入以下依赖。

#### 配置pom文件

```xml
<!--依赖管理-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <!--资源库管理-->
    <repositories>
        <repository>
            <id>spring-snapshots</id>
            <name>Spring Snapshots</name>
            <url>https://repo.spring.io/libs-snapshot</url>
            <snapshots>
                <enabled>true</enabled>
            </snapshots>
        </repository>
    </repositories>
    <!--依赖管理中心-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>Edgware.RELEASE</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!--构建中心-->
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <!-- defined in spring-cloud-starter-parent pom (as documentation hint),
                    but needs to be repeated here -->
                <configuration>
                    <requiresUnpack>
                        <dependency>
                            <groupId>com.netflix.eureka</groupId>
                            <artifactId>eureka-core</artifactId>
                        </dependency>
                        <dependency>
                            <groupId>com.netflix.eureka</groupId>
                            <artifactId>eureka-client</artifactId>
                        </dependency>
                    </requiresUnpack>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

设置完成以后，开始主体设置，创建包>>创建主方法>>配置yml文件
**项目目录结构**：
![文件目录](http://upload-images.jianshu.io/upload_images/3565867-bbba6a0aa2a3412e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 新建Application.java文件

```java

/**
 * @Description : Eureka 服务发现server 启动类
 * @Author hanyahong
 * @Date 2017/12/4- 16:14
 */
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) throws Exception {
        SpringApplication.run(Application.class, args);
    }

}
```
说明：
**@EnableEurekaServer** :服务发现服务端注解，设置以后将作为服务注册中心。

**@SpringBootApplication**：springboot启动主程序注解。

### 配置application.yml文件
在位于resources的文件夹下面创建该文件，该文件作为服务的配置文件，可以配置相关子项目参数。
```yml
#server 端口号设置
server:
  port: 8081

#注册中心设置，server本身不被发现
eureka:
  client:
    registerWithEureka: false
    fetchRegistry: false
  server:
    waitTimeInMsWhenSyncEmpty: 0
```

至此，一个完整的服务注册中心基本搭建完毕。可以进行启动测试。

![启动程序](http://upload-images.jianshu.io/upload_images/3565867-6ce0259d2a982c43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 验证是否成功
我们可以在Terminal看到以下信息，说明启动成功。
```git

 .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.5.9.RELEASE)
...
2017-12-04 18:04:30.780  INFO 32476 --- [      Thread-11] e.s.EurekaServerInitializerConfiguration : Started Eureka Server
2017-12-04 18:04:30.823  INFO 32476 --- [           main] s.b.c.e.t.TomcatEmbeddedServletContainer : Tomcat started on port(s): 8081 (http)
2017-12-04 18:04:30.825  INFO 32476 --- [           main] .s.c.n.e.s.EurekaAutoServiceRegistration : Updating port to 8081
2017-12-04 18:04:30.830  INFO 32476 --- [           main] com.hyh.Application                      : Started Application in 14.784 seconds (JVM running for 15.647)
2017-12-04 18:05:30.759  INFO 32476 --- [a-EvictionTimer] c.n.e.registry.AbstractInstanceRegistry  : Running the evict task with compensationTime 0ms


```
通过浏览器访问 **http://localhost:8081/** 可以看到已经启动正常。
![image.png](http://upload-images.jianshu.io/upload_images/3565867-a0c1d9710fd25e59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 使用Maven 插件进行打包

**提醒**：在打包之前需要本地已经安装好Git/Maven并配置好环境变量。博主因为不喜欢使用Idea自带的JDK/MAVEN/GIT,都是单独安装的。这个看个人喜好吧。
另外，博主喜欢使用命令进行驱动打包，因此对Idea的Terminal进行了重新配置，将Win下面CMD替换成了Git.bash，如果有同学感兴趣可以自行体验。
### IDEA 默认Terminal 修改
打开 File>>Settings，找到修改配置选项
![Settings](http://upload-images.jianshu.io/upload_images/3565867-517c391f8bb29543.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后将默认的Shell地址改成Git.
![配置默认Shell](http://upload-images.jianshu.io/upload_images/3565867-dc8bcccb34b0325a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

设置完成以后进行保存。回到我们界面可以通过**Alt + F12** 快速打开shell窗口。
![shell窗口](http://upload-images.jianshu.io/upload_images/3565867-b8240f643f9fd732.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





### 正式开始打包

首先使用``` pwd ``` 查看所在的目录，应该是父类项目的根目录，```ls``` 命令或者```dir``` 查看，该目录下面的文件与文件夹，我们需要跳转到子目录下面,进行子项目（服务注册中心）打包。所以使用 ```cd ``` 命令跳转子目录
使用命令```mvn clean package``` 就可以实现maven打包。可能会先下载一些依赖，完了就会开始打包操作。


![跳转打包目录](http://upload-images.jianshu.io/upload_images/3565867-7840ee841873a00f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**Maven打包**
```shell
mvn clean package
```
执行命令后出发maven构建，如下。
```shell
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building cloud-hyh-discovery-eureka V1.0.0
[INFO] ------------------------------------------------------------------------
[INFO]
[INFO] --- maven-clean-plugin:2.6.1:clean (default-clean) @ cloud-hyh-discovery-eureka ---
[INFO] Deleting E:\workplace\spring-cloud-microservice\cloud-hyh-discovery-eureka\target
[INFO]
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ cloud-hyh-discovery-eureka ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] Copying 0 resource
[INFO]
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ cloud-hyh-discovery-eureka ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to E:\workplace\spring-cloud-microservice\cloud-hyh-discovery-eureka\target\classes
[INFO]
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ cloud-hyh-discovery-eureka ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory E:\workplace\spring-cloud-microservice\cloud-hyh-discovery-eureka\src\test\resources
[INFO]
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ cloud-hyh-discovery-eureka ---
[INFO] Nothing to compile - all classes are up to date
[INFO]
[INFO] --- maven-surefire-plugin:2.18.1:test (default-test) @ cloud-hyh-discovery-eureka ---
[INFO] No tests to run.
[INFO]
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ cloud-hyh-discovery-eureka ---
[INFO] Building jar: E:\workplace\spring-cloud-microservice\cloud-hyh-discovery-eureka\target\cloud-hyh-discovery-eureka-V1.0.0.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:1.5.9.RELEASE:repackage (default) @ cloud-hyh-discovery-eureka ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 4.984 s
[INFO] Finished at: 2017-12-04T19:24:35+08:00
[INFO] Final Memory: 37M/326M
[INFO] ------------------------------------------------------------------------
```

至此，一个完整的服务注册中心，我们就开发并打包完毕。可以通过**target ** 文件夹找到刚刚生成的jar文件。进行独立启动了。
![image.png](http://upload-images.jianshu.io/upload_images/3565867-8c4d9942b92cef0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

使用命令可以启动独立的jar保程序。

```shell
java -jar XXX.jar
```
Linux环境下，后台启动命令
```shell
nohup java -jar xxx.jar &
```

**备注：** Linux查看端口对于的启动程序命令,XXXX代表端口号
```shell
netstat -anp|grep XXXX
```
通过以上命令查到程序对于的信息（包括PID）。如果想停止，可以通过PID（进程ID）杀死相关进程。
```shell
kill -9 PID
```

## 博客源码地址

Github地址：[https://github.com/hanyahong/spring-cloud-microservice](https://github.com/hanyahong/spring-cloud-microservice)

码云地址：[https://gitee.com/hyhvpn/spring-cloud-microservice](https://gitee.com/hyhvpn/spring-cloud-microservice)
