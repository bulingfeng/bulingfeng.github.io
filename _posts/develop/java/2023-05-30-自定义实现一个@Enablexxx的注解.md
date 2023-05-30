---
title: "自定义实现一个@Enablexxx的注解"
subtitle: "自定义实现一个@Enablexxx的注解"
layout: post
author: "bulingfeng"
header-style: text
tags:
- develop
- SpringBoot
- SpringFrameWork
---

# 自定义实现一个@Enablexxx的注解

@Enablexxx的注解的目的是为了当此注解和@Compnent或者其派生注解，比如@Configuration等等，联合使用的时候，就会把自己想要注入到spring中的类注入。从而方便开发人员使用自动装备的类。

总而言之，只要是@Enablexxx注解的所在类能够注入到spring容器中，那么自己想要装配的类也就能装配到spring容器中。

## 实现思路

1. 写一个自定义注解，并且使用@Component注解来修饰；
2. 实现**ImportSelector**接口；
3. 使用@Import注解来把对应的ImportSelector实现类给配置进来，或者使用@Import把对应想要注入的类配置进来；
4. 使用@Enablexxx注解，并保证@Enablexxx注解的类能够注入到spring容器中。

## 代码实现

自定义注解

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
@Import(value = {HelloWorldConfig.class}) // 注解的方式来实现
//@Import(value = {HelloWorldSelector.class}) // 接口的方式实现
public @interface EnableHelloWorld {
    String value() default "";
}
```

实现ImportSelector接口

```java
/**
 * 使用接口的方式来实现激活对应的bean，这种方式比使用注解的方式更加灵活,这里可以注入你想要的任何类
 */
public class HelloWorldSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        // 数组中的内容为你想要注入到容器中的bean
        return new String[]{HelloWorldConfig.class.getName()};
    }
}
```

使用@EnableHelloWorld注解

```java
@EnableHelloWorld
public class InterfaceApplication {
    public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext=new SpringApplicationBuilder(InterfaceApplication.class)
                .web(WebApplicationType.NONE)
                .run(args);

        // 验证HelloWorldConfig是否注入成功
        HelloWorldConfig helloWorldConfig =
                applicationContext.getBean(HelloWorldConfig.class);
        System.out.println("HelloWorldConfig:"+ helloWorldConfig);
        // 进行查找对应的bean
        String beanName = applicationContext.getBean("beanName", String.class);
        System.out.println("bean:"+beanName);


        // 关闭上下文
        applicationContext.close();
    }
}
```

## 主要要点

1. 自定义的@Enablexxx注解，有两种实现方案：
   - 实现@Import导入实现ImportSelector接口的类（这种方式更加灵活）
   - 使用@Import直接把想要注入的对象给写进去
2. 自定义的注解@Enablexxx只需要加载到类上，并且这个类能够注入到spring容器即可。