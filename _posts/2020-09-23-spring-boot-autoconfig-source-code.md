---
title: SpringBoot自动配置
tags: 源码分析 spring-boot 
categories: spring-boot
---

* TOC
{:toc}

## 简介

在使用Springboot的过程中，有些配置不满足我们的业务需求时候，我们就需要人为的修改组件的配置。那么就涉及到该如何配置，配置的参数有哪些，参数的作用是什么。这就对SpringBoot的自动配置有足够清醒的了解，然后游刃有余的灵活配置（不是凡事找谷歌，这样会很低效）。

## SpringBoot自动配置类的加载流程

### 1、寻找加载类位置

```
我们从Springboot的main启动程序中开始。
@SpringBootApplication -> @EnableAutoConfiguration -> @Import(AutoConfigurationImportSelector.class)
```

### 2、分析源代码

1、源代码入口位置：org.springframework.boot.autoconfigure.AutoConfigurationImportSelector#getAutoConfigurationEntry

``` java
/**
	 * Return the {@link AutoConfigurationEntry} based on the {@link AnnotationMetadata}
	 * of the importing {@link Configuration @Configuration} class.
	 * @param annotationMetadata the annotation metadata of the configuration class
	 * @return the auto-configurations that should be imported
	 */
	protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);// 这个方法是获取加载自动配置方法。进入getCandidateConfigurations
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```

```java
/**
	 * Return the auto-configuration class names that should be considered. By default
	 * this method will load candidates（申请） using {@link SpringFactoriesLoader} with
	 * {@link #getSpringFactoriesLoaderFactoryClass()}.
	 * @param metadata the source metadata
	 * @param attributes the {@link #getAttributes(AnnotationMetadata) annotation
	 * attributes}
	 * @return a list of candidate configurations
	 */
	protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
// 进入loadFactoryNames
List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```

```
/**
	 * Load the fully qualified class names of factory implementations of the
	 * given type from {@value #FACTORIES_RESOURCE_LOCATION}, using the given
	 * class loader.(加载了jar包内位于FACTORIES_RESOURCE_LOCATION中需要自动配置类，靠使用classloarder)
	 * @param factoryType the interface or abstract class representing the factory
	 * @param classLoader the ClassLoader to use for loading resources; can be
	 * {@code null} to use the default
	 * @throws IllegalArgumentException if an error occurs while loading factory names
	 * @see #loadFactories
	 */
	public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}

	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
			// 加载了FACTORIES_RESOURCE_LOCATION 内部的所有配置的类
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					String factoryTypeName = ((String) entry.getKey()).trim();
					for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
						result.add(factoryTypeName, factoryImplementationName.trim());
					}
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```

2、加载配置文件的位置

```
public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";
```

3、那些jar包中会有spring.factories配置文件

```
spring-boot
spring-boot-autoconfigure
```

下面展示spring-boot jar包中META-INF/spring.factories。比如spring-boot-autoconfigure可以自行查看其中启动配置的内容。

```
# PropertySource Loaders
org.springframework.boot.env.PropertySourceLoader=\
org.springframework.boot.env.PropertiesPropertySourceLoader,\
org.springframework.boot.env.YamlPropertySourceLoader

# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener

# Error Reporters
org.springframework.boot.SpringBootExceptionReporter=\
org.springframework.boot.diagnostics.FailureAnalyzers
```

## 如何配置配置文件

不管是使用properties还是yml文件都有提示功能，那么如果知道某个自动配置都有哪些配置，并且具体的配置的含义是什么呢？

那么来到**spring-boot-autoconfig**包中看对对于配置类。如图:
![spring-boot-autoconfig类](https://bulingfeng.com/static/img/springboot/circle-queue.jpg)

比如想查看elasticsearch中的配置，打开ElasticsearchRestClientProperties.class。

```java
@ConfigurationProperties(prefix = "spring.elasticsearch.rest")
public class ElasticsearchRestClientProperties {

	/**
	 * Comma-separated list of the Elasticsearch instances to use.
	 */
	private List<String> uris = new ArrayList<>(Collections.singletonList("http://localhost:9200"));

	/**
	 * Credentials username.
	 */
	private String username;

	/**
	 * Credentials password.
	 */
	private String password;
	.....

}
```

```yml
spring:
  elasticsearch:
    rest: 
```

可以发现prefix的值和该类的熟悉就是配置文件yml中可以配置的属性，以及各个属性的含义。其他自动化配置也是一样，大多数是在*Properties和Configurations中。

