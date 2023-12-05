---
title: "SpringBoot的一些基本特性"
subtitle: "SpringBoot的一些基本特性"
layout: post
author: "bulingfeng"
header-style: text
tags:
- SpringBoot
- 技术
---

## 关于版本号的简化

SpringBoot大大简化了java开发人员的配置，而这就涉及到一些配置的规约。比如当我们想使用springboot的web功能的时候，那么只需要引入：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

那么springmvc等功能都就都具备了。并且这个maven的坐标还没有说明版本号。这里就涉及到最顶层一个parent依赖了：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.1.2</version>
</parent>
```

当我们点进去`spring-boot-starter-parent`的时候，发现这个包里面都是些的一些开发者信息，打包信息等。并且并没有规定一些包的版本号之类的，但是再看它也有一个父类：

```xml
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>3.1.2</version>
</parent>
```

`spring-boot-dependencies`这个坐标规定了很多关于一些常用的包版本控制。下面是部分版本号代码:

```properties
<properties>
    <activemq.version>5.18.2</activemq.version>
    <angus-mail.version>1.1.0</angus-mail.version>
    <artemis.version>2.28.0</artemis.version>
    <aspectj.version>1.9.19</aspectj.version>
    <assertj.version>3.24.2</assertj.version>
    <awaitility.version>4.2.0</awaitility.version>
    <brave.version>5.15.1</brave.version>
    <build-helper-maven-plugin.version>3.3.0</build-helper-maven-plugin.version>
    <byte-buddy.version>1.14.5</byte-buddy.version>
    <cache2k.version>2.6.1.Final</cache2k.version>
    .....
 <properties/>
```

而当我们搜索`spring-boot-starter-web`的时候，我们发现了一个相关的版本定义：

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <version>3.1.2</version>
</dependency>
```

## 自动装配

关于自动装配，自己要关注两个依赖:`spring-boot-starter`和`spring-boot-starter`中的`spring-boot-autoconfig`

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <version>3.1.2</version>
  <scope>compile</scope>
</dependency>
```

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-autoconfigure</artifactId>
  <version>3.1.2</version>
  <scope>compile</scope>
</dependency>
```

当我们查看spring-boot-autoconfig包中的代码的时候，会发现有很多springboot描述的内容，其中很多内容都是飘红的。这是因为没有引入jar包，当引入jar包的时候，这些飘红代码就会消失，也就是说，这些自动装配的逻辑就会生效。

