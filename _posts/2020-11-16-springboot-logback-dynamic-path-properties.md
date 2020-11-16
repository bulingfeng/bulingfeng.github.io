---
title: springboot-logback日志动态路径
tags:  日志
categories: spring-boot
---

* TOC
{:toc}


## 1、简介

> springboot常常使用logback来做日志。大多数日志的路径配置到logback.xml中也没有什么问题。但是在特定的需求下，比如需要灵活的改动日志的位置，这个时候日志路径写入到logback.xml就不太合适了。最好的方法是配置的到application.yml中。这样修改好之后重启程序即可。

## 2、路径配置

- 关于如何配置logbak。请参考下面文章。

  [logback配置](https://bulingfeng.com/spring-boot/spring-boot-logback-proerties/)

- 配置日志路径

> log:
>
>   path: /home/logs

- logback-spring.xml中添加配置

```xml
<!--contextName一定需要和scope一致，这样才能会让配置中的生效-->
    <contextName>context</contextName>
    <springProperty scope="context" name="log.path" source="log.path"/>
    <!-- name的值是变量的名称，value的值时变量定义的值。通过定义的值会被插入到logger上下文中。定义后，可以使“${}”来使用变量。 -->
```

## 3、log.path__IS_UNDEFINED问题

> 通过以上配置就可以达到日志的灵活文件目录灵活配置的目的。但是会出现一个问题，就是启动jar包位置总会出现一个log.path__IS_UNDEFINED。经过个人实验，如果单独只引入spring-boot-starter-webflux包进行测试是没有任何问题。但是往往真实项目中需要引入大量的jar，那么就避免不了其中使用了logback日志。这就那些不能从配置文件中获取到日志目录的变量。

**解决方案**

1. 修改logback.xml文件名

```
按照自己需求修改。比如我的就是logback-blf.xml
```

2. 指定加载的logback配置文件

```yml
# 指定某个特定的配置文件
logging:
  config: classpath:logback-blf.xml
```

