---
title: 【Spring Boot-技巧】API返回值去除为NULL的字段
date: 2018-07-26 10:10:10
tags:
- Spring Boot
categories:
- Spring Boot
---

> 在前后端分离的微服务时代，后端API需要良好的规范。本篇主要将一个数据返回时的一个小技巧-- 过滤为空字段

<!--more-->

## 简介
在前后端分离的微服务时代，后端API需要良好的规范。本篇主要将一个数据返回时的一个小技巧-- **过滤为空字段**
**解决痛点**：将有效解决数据传输过程中的流量浪费。
## 组件简介

### Jackson ObjectMapper

通过自定义配置该组件可以选择性序列化返回的JSON。

### 官方解释

 Spring MVC（客户端和服务器端）用于HttpMessageConverters在HTTP交换中协商内容转换。如果Jackson在类路径上，您已经获得了提供的默认转换器 ` Jackson2ObjectMapperBuilder ` ，其中一个实例是为您自动配置的。
Spring Boot还具有一些功能，可以更轻松地自定义此行为。

## 实战代码

### 创建配置类

首先创建一个配置类，加入定义为： `JacksonConfig`

### 代码清单

```java
import com.fasterxml.jackson.annotation.JsonInclude;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;

/**
 * @author Han Yahong<hyhvpn@126.com>
 * @program 51-baojiadan-service
 * @description 返回json空值去掉null和""
 * @create 2018-07-26 11:04
 */
@Configuration
public class JacksonConfig {
    @Bean
    @Primary
    @ConditionalOnMissingBean(ObjectMapper.class)
    public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return objectMapper;
    }
}

```
**代码解释**：以上就是全部代码，通过注解**@Configuration** 注入后可自动配置。

**关键点**：`objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);`
通过该方法对mapper对象进行设置，所有序列化的对象都将按改规则进行系列化。
其中枚举属性：`JsonInclude.Include.NON_NULL 有以下选择：


| 属性 | 使用场景 |
| ------ | ------ |
| `Include.Include.ALWAYS` | 默认 |
| `Include.NON_DEFAULT` | 属性为默认值不序列化 |
| `Include.NON_EMPTY` | 属性为 空（""） 或者为 NULL 都不序列化 |
| `Include.NON_NULL` | 属性为NULL 不序列化 |

**替换非空**

可通过自定义替换原有制定值。代码如下：

```java
public ObjectMapper jacksonObjectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper objectMapper = builder.createXmlMapper(false).build();
        // 字段保留，将null值转为""
        objectMapper.getSerializerProvider().setNullValueSerializer(new JsonSerializer<Object>() {
            @Override
            public void serialize(Object o, JsonGenerator jsonGenerator,
                                  SerializerProvider serializerProvider)
                    throws IOException, JsonProcessingException {
                jsonGenerator.writeString("");
            }
        });
        return objectMapper;
    }
```

### 本文结束

BLOG官网地址：[https://www.hanyahong.com](https://www.hanyahong.com)
