---
title: "SpringBoot常用注解"
subtitle: "SpringBoot常用注解"
layout: post
author: "bulingfeng"
header-style: text
tags:
- SpringBoot
- 技术
---

## @Import

这个注解的使用条件就式在在使用的这个类上面必须是Spring-context中的bean对象。这个类可以注入你想要的任何类。并且注入的bean名称为class的全类名。

## @Conditional以及其派生注解

根据条件进行注解，并且它还有大量的派生注解。

这类注解可以使用在方法上，也可以使用在类上。如果使用在类上，则会根据条件是否把对应的对象给注入到spring上下文中。

## @DependsOn

这个注解能够保证在初始化bean的时候能够按照一定的顺序来，也就是必须在某些bean被initialization的时候这个bean才会被initialization。

需要注意的有两点:

> 1. 当依赖的bean不存在的时候会报错。
> 2. 使用@DependsOn的时候需要防止出现循环依赖(circular dependency)

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DependsOn {

	String[] value() default {};
}
```

根据java代码知道，@DependsOn是通过bean的name来进行注入的。

## @ImportResource

当下流行是springboot，而springboot是鼓励使用`类`配置，而原来的springmvc在注入bean则是使用xml的方式。

当原来的版本springmvc想要迁移到springboot并且你还不愿意把每个bean都适用类的方式来进行迁移，那么可以使用@ImportResource来把xml中的bean注入到spring的上下文中。

## @ConfigurationProperties和@EnableConfigurationProperties

@ConfigurationProperties能够通过前缀来把配置文件中的值给注入到bean，当然这个bean也需要使用@Configuration或者@Component等注解把bean注入到spring上下文中。

@EnableConfigurationProperties则可以在引导类中使用，而注入被@ConfigurationProperties修饰的类，但是这个类没有使用@Component或者@Configuration。