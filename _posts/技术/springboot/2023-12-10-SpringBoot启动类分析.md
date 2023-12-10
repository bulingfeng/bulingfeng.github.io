SpringBoot在使用的过程中，大家一定会使用到`@SpringBootApplication`注解，因为每个SpringBoot的主引导类都会标注此注解，这样SpringBoot才会正常启动成功。

那我们接下来点进去`@SpringBootApplication`注解来一探究竟。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```

这里有很多注解，我们注重关注3个注解：

> @SpringBootConfiguration
>
> @EnableAutoConfiguration
>
> @ComponentScan

其中`@SpringBootConfiguration`这个注解可以当做一个普通的`@Configuration`注解来处理。而`@ComponentScan`这个注解就是一个包扫描包注解，我们着重来观察下`@EnableAutoConfiguration`这个注解。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
```

这里有两个比较重要的注解`@AutoConfigurationPackage`和`@Import(AutoConfigurationImportSelector.class)`；

我们首先来看下`@AutoConfigurationPackage`这个注解；

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
```

