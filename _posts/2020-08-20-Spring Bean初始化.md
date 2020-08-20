---
title: Spring Bean初始化
tags: java Spring-Framework
categories: Spring-Framework
---

* TOC
{:toc}



## 1、Spring Bean初始化有以下方式：

1. @PostConstruct
2. @Bean(initMethod="")
3. 实现InitializingBean接口中的afterPropertiesSet方法

其中执行顺序为：@PostConstruct > InitializingBean#afterPropertiesSet > @Bean(initMethod="")

## 2、Spring Bean初始化demo

### 初始化的Bean

```java
package com.bulingfeng.bean.factory;

import org.springframework.beans.factory.InitializingBean;

import javax.annotation.PostConstruct;

/**
 * @Author:bulingfeng
 * @Date: 2020-08-19
 * 初始化顺序 @PostConstruct>afterPropertiesSet>initPerson
 */
public class DefaultPersonFactory implements InitializingBean {

    // 基于PostConstruct来进行bean对象的初始化，这种方式是该注解的默认方式
    @PostConstruct
    public void init(){
        System.out.println("@PostConstruct 正在进行初始化......");
    }


    public void initPerson(){
        System.out.println("initPerson 正在进行初始化......");
    }

    // 这个是必须实现的方法
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("afterPropertiesSet 正在进行初始化......");
    }
}

```

### 测试的main方法

```java
package com.bulingfeng.bean.definition;

import com.bulingfeng.bean.factory.DefaultPersonFactory;
import com.bulingfeng.bean.factory.PersonFactory;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Lazy;

/**
 * @Author:bulingfeng
 * @Date: 2020-08-20
 * 初始化bean
 */
public class InitializationBeanDemo {
    public static void main(String[] args) {
        // 1.创建beanfactory容器
        AnnotationConfigApplicationContext applicationContext=new AnnotationConfigApplicationContext();

        // 2.把类注册到ioc容器中
        applicationContext.register(InitializationBeanDemo.class);
        // 启动spring上下文
        applicationContext.refresh();

        // 通过类型来获取bean
        PersonFactory personFactory=applicationContext.getBean(PersonFactory.class);
        System.out.println("获取到的类型为:"+personFactory);
    }


    @Bean(initMethod="initPerson")
    // @Lazy
    public DefaultPersonFactory personFactory(){
        return new DefaultPersonFactory();
    }

}

```

## 运行结果

```
@PostConstruct 正在进行初始化......
afterPropertiesSet 正在进行初始化......
initPerson 正在进行初始化......
获取到的类型为:com.bulingfeng.bean.factory.DefaultPersonFactory@7ce6a65d
```

## 3、延迟加载

延迟加载使用的关键字为@Lazy，也就是说只有当依赖查找（调用bean）时候才会进行初始化。总结就是**按需初始化**。

main方法代码为：

```java
package com.bulingfeng.bean.definition;

import com.bulingfeng.bean.factory.DefaultPersonFactory;
import com.bulingfeng.bean.factory.PersonFactory;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Lazy;

/**
 * @Author:bulingfeng
 * @Date: 2020-08-20
 * 初始化bean
 */
public class InitializationBeanDemo {
    public static void main(String[] args) {
        // 1.创建beanfactory容器
        AnnotationConfigApplicationContext applicationContext=new AnnotationConfigApplicationContext();

        // 2.把类注册到ioc容器中
        applicationContext.register(InitializationBeanDemo.class);
        // 启动spring上下文
        applicationContext.refresh();

        // 通过类型来获取bean
//        PersonFactory personFactory=applicationContext.getBean(PersonFactory.class);
//        System.out.println("获取到的类型为:"+personFactory);
    }


    @Bean(initMethod="initPerson")
    @Lazy
    public DefaultPersonFactory personFactory(){
        return new DefaultPersonFactory();
    }

}

```

输出结果:

```
Process finished with exit code 0
```

因为没有调用**PersonFactory personFactory=applicationContext.getBean(PersonFactory.class)**，所以没有进行初始化。

疑问：

Spring Bean的实例化和初始化区别。