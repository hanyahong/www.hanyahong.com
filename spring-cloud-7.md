---
title: 【微服务】之七：轻松搞定SpringCloud微服务-API权限控制
date: 2017.12.24 11:23
tags:
	- 微服务
categories:
	- Spring Cloud
---



> 权限控制，是一个系统当中必须的重要功能。张三只能访问输入张三的特定功能，李四不能访问属于赵六的特定菜单。这就要求对整个体系做一个完善的权限控制体系。该体系应该具备针区分用户、权限、角色等各种必须的功能。

<!--more-->


## 本系列教程


[【微服务】轻松搞定SpringCloud微服务目录-独立博客](http://www.hanyahong.com/categories/Spring-Cloud/)

本系列为连载文章，阅读本文之前强烈建议您先阅读前面几篇。
上一节我们讲到API网关zuul ,对于Spring Cloud 来说，zuul除了可以做api接口的统一暴露，还应该具备权限控制的相关功能。

## 单例应用权限控制
在没有引入Spring Cloud成套体系中，对于单体springboot 所开发的应用使用springmvc自带拦截器就可以实现对路径的拦截，截取request中特定的参数进行校验，如果合法就可以访问，如果不合法便返回403。
## Spring Cloud Zuul 过滤器简介
对于组件zuul中，其实带有权限认证的功能，那就是ZuulFilter过滤器。
ZuulFilter是Zuul中核心组件，通过继承该抽象类，覆写几个关键方法达到自定义调度请求的作用。

## 开始起飞
起飞之前，还是那句话，推荐先看前面博文。本次还是基于api网关功能的延伸，因此为了避免和前一片文章中子项目冲突，我们新建一个子项目，然后复制api-gateway-zuul项目的代码。

### 配置AccessTokenFilter文件

在新建的子项目下面，我们创建一个包config ,然后在下面创建AccessTokenFilter文件，清单如下：
```java
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import javax.servlet.http.HttpServletRequest;

/**
 * Created by Administrator on 2017/12/21.
 */
public class AccessTokenFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return "pre";//前置过滤器
    }

    @Override
    public int filterOrder() {
        return 0;//优先级为0，数字越大，优先级越低
    }

    @Override
    public boolean shouldFilter() {
        return true;//是否执行该过滤器，此处为true，说明需要过滤
    }

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        String username = request.getParameter("token");
        if (null != username && username.equals("www.hanyahong.com")) {//暂时简单化测试
            ctx.setSendZuulResponse(true);// 对该请求进行路由
            ctx.setResponseStatusCode(200);
            ctx.set("isSuccess", true);// 设值，可以在多个过滤器时使用
            return null;
        } else {
            ctx.setSendZuulResponse(false);// 过滤该请求，不对其进行路由
            ctx.setResponseStatusCode(403);// 返回错误码
            ctx.setResponseBody("{\"result\":\"Request illegal!the token is null\"}");// 返回错误内容
            ctx.set("isSuccess", false);
            return null;
        }
    }
}

```
**说明：**

**filterOrder:** filter执行顺序，通过数字指定 
**shouldFilter:** filter是否需要执行 true执行 false 不执行 
**run :**  filter具体逻辑 
**filterType :** filter类型,分为pre、error、post、	route
	> pre:请求执行之前filter 
	  route: 处理请求，进行路由 
	  post: 请求处理完成后执行的filter 
	  error:出现错误时执行的filter

官网给出一个四种类型的示意图：
![image.png](http://upload-images.jianshu.io/upload_images/3565867-66e229055cf7fad0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### 注入AccessToeknFilter

自行创建Filter需要手动加载到容器进行统一管理。在主方法**Application.java**中，可以加入以下代码：
```java
/**
     * 加载过滤器
     * @return
     */
    @Bean
    public AccessTokenFilter accessFilter() {
        return new AccessTokenFilter();
    }
```

## 启动并测试

分别启动子项目 **cloud-hyh-discovery-eureka** 、**cloud-hyh-service-1** 以及刚刚创建的api网关的新子项目。
**首先**可以访问 http://localhost:8081/ 查看服务是否已经都启动完毕。

**其次**通过网关访问service-1服务中的**/ribbon/name** 接口，查看是否允许访问。http://localhost:8080/cloud-service/ribbon/name。通过访问可以看到浏览器提示：
```json
{"result":"Request illegal!the token is null"}
```
最后，访问带有权限认证的url，http://localhost:8080/cloud-service/ribbon/name?token=www.hanyahong.com
可以看到浏览器可以通过验证，进入了子系统中的API，并返回了相关结果。
```json
千万之路刚开始-www.hanyahong.com-beijing该服务器端口8071
```

### 说明
一般token会通过一系列加密处理，另外一般是放在请求头部。如果前后端分离的话就会设计跨域的问题。这个我们会在后面开一篇专门讲解跨域访问的文章细致讲解。
另外，在实际应用中还有很多需要配置的地方，绝非这般简单的配置。这个希望在实际的项目当中，你们可以自己体会。
## 源码

本文出处：http://www.hanyahong.com/
Github源码：https://github.com/hanyahong/spring-cloud-microservice 
转发请注明出处！
