---
title: "SpringBoot实现优雅关闭程序"
subtitle: "SpringBoot实现优雅关闭程序"
layout: post
author: "bulingfeng"
header-style: text
tags:
- SpringBoot
- 技术
---
## 介绍

实际的项目发布过程中，免不了要对应用进行停机发布，这个时候如果有一些请求还没有进行处理完，如果你强行进行关闭，会造成用户体验不好，或者一些数据损失。

基于这样的原因，我们需要对应用程序进行优化的关闭(shutdown gracefully)

## 实现方法

现在SpringBoot已经提供了这方面功能，能够使我们快速的和简单地完成这样的一份工作。我们只需要在application.yml文件中配置上如下配置即可

```yaml
server:
  shutdown: graceful
```

springboot默认是30秒后进行强制关闭，如果你想设置比较长的时间之后再进行关闭程序呢？通过以下配置：

```yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 1m
```

## 验证阶段

根据上面的步骤配置好yaml文件以后，我们来编写一个controller类，来模拟实现一个正在进行的业务处理，代码如下：

```java
@RestController
@RequestMapping("/shutDown")
public class ShutDownController {

    @GetMapping("/action-slow")
    public void shutDownGracefully(){
        for (int i = 0; i < 66; i++) {
            try {
                System.out.println("hello"+i);
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
    }
}
```

启动springboot项目以后，然后调用`/shutDown/action-slow`这个接口，我们可以看到控制在不断的打印数据:

```
hello0
hello1
hello2
hello3
```

这个时候我们通过linux命令来找到对应的项目，然后执行kill操作来观察情况

```shell
kill -15 4231
```

我们看到控制台打印了如下的内容：

```tex
2023-11-27 20:41:43.326  INFO 4231 --- [ionShutdownHook] o.s.b.w.e.tomcat.GracefulShutdown        : Commencing graceful shutdown. Waiting for active requests to complete
```

说明我们配置的优雅的shutdown生效了。

**注意事项**

1. 不能使用kill -9 来对java进程进行杀死，因为kill -9 是强制性的；
2. kill -15 并不是唯一的测试方法，idea你点击下stop也能得到同样的效果。