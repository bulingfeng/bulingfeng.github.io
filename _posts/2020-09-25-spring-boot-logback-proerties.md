---
title: logback.xml参数配置
tags: 日志 
categories: spring-boot
---

* TOC
{:toc}


## 1、logback.xml和logback-spring.xml区别

logback.xml是logback的原始的日志配置文件。logback-spring.xml是springboot的专属logback配置文件。其中logback-spring.xml会有一些额外定制特性。

**官方建议**

> When possible, we recommend that you use the `-spring` variants for your logging configuration (for example, `logback-spring.xml` rather than `logback.xml`). If you use standard configuration locations, Spring cannot completely control log initialization.

**官方警告**

> There are known classloading issues with Java Util Logging that cause problems when running from an 'executable jar'. We recommend that you avoid it when running from an 'executable jar' if at all possible.

## 2、loback-spring.xml额外定制功能有哪些

根据不同的环境配置来输出对应日志信息

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
</springProfile>

<springProfile name="dev | staging">
    <!-- configuration to be enabled when the "dev" or "staging" profiles are active -->
</springProfile>

<springProfile name="!production">
    <!-- configuration to be enabled when the "production" profile is not active -->
</springProfile>
```

## 3、logback-spring.xml具体配置信息

[logback-spring.xml][https://bulingfeng.com/static/img/log/logback-spring.xml]

## 4、参考文档

> https://docs.spring.io/spring-boot/docs/current/reference/html/spring-boot-features.html#boot-features-logback-extensions

