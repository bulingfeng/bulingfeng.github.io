---
title: 基于@Enable注解来注册Bean
tags: spring-boot
categories: spring-boot
---

## 简介

> 很多时候我们都是使用@ComponentScan注解来做bean的注入。这样的话需要配合着@Configuration,@Bean等注解来完成。如果我们不想使用@Configuration,@Bean这类注解来注入bean呢？那我们需要使用@Import来导入你想导入的指定类。

## 代码

**自定义@Enable注解**

```java
package com.example.annotation;
import com.example.configuration.HelloWorldConfig;
import org.springframework.context.annotation.Import;
import java.lang.annotation.*;
@Retention(RetentionPolicy.RUNTIME)
@Target(value= ElementType.TYPE)
@Documented
@Import({HelloWorldConfig.class})
public @interface EnableHelloWorld {

}

```

**定义配置类**

```
package com.example.configuration;
public class HelloWorldConfig {
    public String helloWorld(){
        return "hello world by Enabled annoation";
    }
}
```

>请注意，配置类HelloWorldConfig没有加任何的注解，比如@Configuration。

**上下文中获取指定类**

```java
package com.example.bootstrap;

import com.example.annotation.EnableHelloWorld;
import com.example.configuration.HelloWorldConfig;
import org.springframework.boot.WebApplicationType;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.context.ConfigurableApplicationContext;

@EnableHelloWorld
public class HelloWorldEnable {
    public static void main(String[] args) {
        ConfigurableApplicationContext context =new SpringApplicationBuilder(HelloWorldEnable.class)
                .web(WebApplicationType.NONE)
                .run(args);

        HelloWorldConfig helloWorldConfig=context.getBean(HelloWorldConfig.class);
        // 请注意 这里获取到的HelloWorldConfig 可是没有通过任何注解来完成注册的
        // 比如@Confituration @Bean.... 这类注解一概没有使用
        System.out.printf("获取到helloWorldConfig的内容为:"+helloWorldConfig.helloWorld());
    }
}
```

## 总结

> 1、如果使用@Configuration,@Compnent,@Bean等来注入bean，都需要配合@ComponentScan注解来使用。
>
> 2、使用自定义的@Enable注解，可以不表明方法1中的注解，就可以把@Import中所声明的类注册到spring上下文中。