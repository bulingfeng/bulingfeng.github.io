---
title: java注解的继承
tags: java
categories: java
---

## 1、简介

> 关于java注解的继承其实有两种含义，一种是标记在父类上，然后子类继承父类，然后子类能够获取到父类的注解信息。另一种是就是注解上面添加一个注解，这种情况下，给子注解设置值时候，父注解也可以得到子注解的相关值。

## 2、注解通过类的继承进行传递

> 该注解方式是java原始的注解，主要是通过标记@Inherited该注解，从而能够让继承该类的的注解进行传递。

**代码实现**

```java
package com.bulingfeng.origin.annotation;

import java.lang.annotation.*;
import java.util.Arrays;

/**
 * @Author:bulingfeng
 * @Date: 2020/12/20
 */
public class ClassInheritedTest {
    @Target(value = ElementType.TYPE)
    @Retention(value = RetentionPolicy.RUNTIME)
    @Inherited // 声明注解具有继承性
    @interface AInherited {
        String value() default "";
    }

    @Target(value = ElementType.TYPE)
    @Retention(value = RetentionPolicy.RUNTIME)
    @Inherited // 声明注解具有继承性
    @interface BInherited {
        String value() default "";
    }

    @Target(value = ElementType.TYPE)
    @Retention(value = RetentionPolicy.RUNTIME)
            // 未声明注解具有继承性
    @interface CInherited {
        String value() default "";
    }

    @AInherited("父类的AInherited")
    @BInherited("父类的BInherited")
    @CInherited("父类的CInherited")
    class SuperClass {
    }

    @BInherited("子类的BInherited")
    class ChildClass extends SuperClass {
    }

    public static void main(String[] args) {
        Annotation[] annotations = ChildClass.class.getDeclaredAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation);
        }
        // output: [@annotations.InheritedTest1$AInherited(value=父类的AInherited), @annotations.InheritedTest1$BInherited(value=子类的BInherited)]
    }
}
```

## 3、AliasFor的方式进行注解的继承

> 该注解不是java的原始注解，是spring-framework框架专有的注解方式。其实想知道该注解的运行方式，只需要查看@Service注解和@Compont这两个注解的源码即可。

**代码实现**

```
// 基础注解
package com.bulingfeng.origin.annotation;
import org.springframework.core.annotation.AliasFor;
import org.springframework.stereotype.Indexed;
import java.lang.annotation.*;
/**
 * @Author:bulingfeng
 * @Date: 2020/12/20
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
@Indexed
public @interface MyAnnotation {
    @AliasFor(attribute = "name")
    String value() default "";

    @AliasFor(attribute = "value")
    String name() default "";
}
```

```
// 子注解
package com.bulingfeng.origin.annotation;
import org.springframework.core.annotation.AliasFor;
import java.lang.annotation.*;
/**
 * @Author:bulingfeng
 * @Date: 2020/12/20
 */
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.TYPE)
@MyAnnotation
public @interface SubAnnotation {
		// 如何不写value或者attribute则自动设置继承同@MyAnnotation中属性一样的别名字段
    @AliasFor(annotation=MyAnnotation.class)
    String value() default "";
}
```

```
// 测试类
package com.bulingfeng.origin.annotation;
import org.springframework.core.annotation.AnnotatedElementUtils;
/**
 * @Author:bulingfeng
 * @Date: 2020/12/20
 */
@SubAnnotation("bulingfeng")
public class TestAnnotation {
    public static void main(String[] args) {
        MyAnnotation myAnnotation = AnnotatedElementUtils.getMergedAnnotation(TestAnnotation.class, MyAnnotation.class);
        System.out.println(myAnnotation);
    }
}

```

## 参考文章

```
https://www.jianshu.com/p/a848655d478e
```

