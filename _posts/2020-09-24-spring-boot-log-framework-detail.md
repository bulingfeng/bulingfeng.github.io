---
title: SpringBoot日志框架整合
tags: 日志 
categories: spring-boot
---

* TOC
{:toc}

## 1、简介

```
spring默认使用的日志框架为：JCL。
Springboot默认使用的日志框架为：SLF4J和Logback。
```

```
由于slf4j和logback都是同一个开发者开发，所以项目中的日志框架使用前两者来搭配就比较简单。但是如果也想用slf4j来做日志门面，而用其他日志框架来做实现。可以参考官方文档：
http://www.slf4j.org/manual.html
```

日志的配置策略如下图:

![concrete-bindings](https://bulingfeng.com/static/img/log/concrete-bindings.png)

比如上图想使用slf4j做日志门面，而日志框架使用log4j。那么值需要在再导入一个适配的两者的jar，即slf4j-log412.jar。

## 2、日志的配置

每一个日志框架都有各自的日志配置文件，**使用slf4j后，配置文件还是使用日志实现框架的配置文件即可。**

## 3、统一框架中所使用的日志框架

### 问题

日常开发的过程中避免不了引用别的框架，比如Spring，Hibernate。但是他们所使用的框架都不一样（而且容易查询日志框架冲突），那么为了统一日志框架的使用。我们需要做一些工作。下表是框架所使用的日志框架

| 集成框架名称 | 使用的日志框架 |
| ------------ | -------------- |
| Spring       | common-logging |
| Hibernate    | jboss-logging  |

## 4、统一日志框架的步骤

**步骤：**

1. 将要替换系统中的日志框架先给排除出去。
2. 使用中间包代替替换原来的日志包（总结一句话就是引入中间包）。
3. 导入实现slf4j的日志框架jar包。

**参考图示：**

**![legacy](https://bulingfeng.com/static/img/log/legacy.png)**



## 5、SpringBoot日志默认使用的日志框架

查看SpringBoot maven依赖的日志框架拓扑图如下：

![log-map](https://bulingfeng.com/static/img/log/log-map.png)

**总结：**

```
如果引入额外的框架使用的时候，只需要把该日志框架内的日志框架给移除（有pom的拓扑图知道是jul和log4j的框架），那么springboot工程就会默认使用logback框架来作为默认的日志框架。
```

## 6、参考文档

```
http://www.slf4j.org/legacy.html
```

