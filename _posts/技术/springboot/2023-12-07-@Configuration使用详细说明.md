---
title: "@Configuration的详细使用"
subtitle: "@Configuration的详细使用"
layout: post
author: "bulingfeng"
header-style: text
tags:
- SpringBoot
- 技术
---

在讨论使用@Configuration之前呢，我先说下我使用的相关技术版本号，springboot的版本为:3.1.2;而这个版本对应的spring的版本是6.0;

## @Configuration的基本使用

我们都知道@Configuration是一个配置的注解，通过这个注解，就是可以把对应的类给注入到spring的context中。比如如下代码：

```java
@Configuration
public class WebConfiguration {
    @Bean
    public Dish getDish(){
        return new Dish("fish",false,100, Dish.Type.FISH);
    }
}
```

我们再通过spring的上下文来获取到WebConfiguration的类。并进行打印。

```tex
org.example.config.WebConfiguration$$SpringCGLIB$$0@420a8042
```

我们通过打印的信息发现，这是一个通过CGLIB来实现的代理类。

## 一些疑问

我们看到`WebConfiguration`类中有一个getDish这个方法，这个方法被@Bean所修饰，并且每次都会返回一个Dish类。那么我们通过拿到WebConfiguration类以后，然后反复的调用getDish那么会反复的创建Dish对象吗？

```java
        ConfigurableApplicationContext applicationContext = SpringApplication.run(MybatisPlusApplication.class, args);

        WebConfiguration webConfig = applicationContext.getBean(WebConfiguration.class);
        System.out.println(webConfig);

        Dish dish1 = webConfig.getDish();
        Dish dish2 = webConfig.getDish();
        System.out.println(dish1 == dish2);
```

打印的结果为：

```
org.example.config.WebConfiguration$$SpringCGLIB$$0@3a7c678b
true
```

我们知道不管你调用几次，其实每次都是同一个对象。即使我在这个方法内创建多个方法，甚至是使用同一个bean的name，代码如下：

```java
@Bean("getDish")
    public Dish getDish(){
        return new Dish("fish",false,100, Dish.Type.FISH);
    }

    @Bean("getDish")
    public Dish getDish2(){
        return new Dish("fish",false,100, Dish.Type.FISH);
    }
```

那么得到的结果，依然是这两个bean是一样的。比如两个beanName不一样呢？比如：

```java
  @Bean
  public Dish getDish(){
      return new Dish("fish",false,100, Dish.Type.FISH);
  }

  @Bean
  public Dish getDish2(){
      return new Dish("fish",false,100, Dish.Type.FISH);
  }
```

那么结果自然是false，则两个bean是不一样的。

那么我们来看下官方文档是如何描述的把

```
In Spring, instantiated beans have a singleton scope by default. This is where the magic comes in: All @Configuration classes are subclassed at startup-time with CGLIB. In the subclass, the child method checks the container first for any cached (scoped) beans before it calls the parent method and creates a new instance.
```

如果想了解更多，可以去官方文档进行查看

```
https://docs.spring.io/spring-framework/reference/core/beans/java/configuration-annotation.html#:~:text=%40Configuration%20is%20a%20class%2Dlevel,to%20define%20inter%2Dbean%20dependencies.
```

## 关于@Configuration中的一些参数使用

在上面的例子中我们可以看到，每次调用同一个getDish方法，都是获取的同一个bean对象。那么如果你想每次调用getDish方法都获取到不同的对象呢？

第一种方法，就是不使用@Configuration注解，而使用@Component注解。

第二种方法就是使用：

```java
@Configuration(proxyBeanMethods = false)
```

`proxyBeanMethods`值的目的就是说是否使用代理类，默认为true.

而spring6.0还又多出来一个值：`enforceUniqueMethods`,这个值默认为true，说明在被@Bean修饰的方法中，是否能够运行方法重名。

```java
@Configuration(enforceUniqueMethods = true)
//@Component
public class WebConfiguration {
    @Bean
    public Dish getDish(){
        return new Dish("fish",false,100, Dish.Type.FISH);
    }
    @Bean
    public Dish getDish(String hello){
        return new Dish("fish",false,100, Dish.Type.FISH);
    }
}
```

这样的话，启动springboot的项目会直接报错。如果把值改成false，则不会报错。并且由于两个Dish都是同一个bean的name，所以这两个方法获取到的Dish对象是一样的。