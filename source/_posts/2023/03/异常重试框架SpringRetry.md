---
title: 异常重试框架SpringRetry
author: songchen
top: false
cover: false
toc: true
mathjax: false
date: 2023-03-09 17:25:33
img:
coverImg:
password:
summary:
tags:
- 机制
categories: [Programming Language,Java language,框架（Framework）,Spring]
---

### 1. 重试机制的业务背景

外部服务对于调用者来说一般都是不可靠的，尤其是在网络环境比较差的情况下，网络抖动很容易导致请求超时等异常情况，这时候就需要用失败重试策略重新调用 API 接口来获取。

在分布式系统中，为了保证数据分布式事务的强一致性，大家在调用RPC接口或者发送MQ时，针对可能会出现网络抖动请求超时情况采取一下重试操作。 大家用的最多的重试方式就是MQ了，但是如果你的项目中没有引入MQ，那就不方便了。

### 2. 重试策略的介绍和限制

重试策略在服务治理方面也有很广泛的使用，通过定时检测，来查看服务是否存活。

重试是有场景限制的，不是什么场景都适合重试，比如参数校验不合法、写操作等（要考虑写是否幂等）都不适合重试。

-   远程调用超时、网络突然中断可以重试。在微服务治理框架中，通常都有自己的重试与超时配置，比如dubbo可以设置retries=1，timeout=500调用失败只重试1次，超过500ms调用仍未返回则调用失败。

### 3. 重试的场景有哪些？

外部 RPC 调用，或者数据入库等操作，如果一次操作失败，可以进行多次重试，提高调用成功的可能性。

### 4. 常用的重试框架

-   Spring异常重试框架Spring Retry：Spring Retry支持集成到Spring或者Spring Boot项目中，而它支持AOP的切面注入写法，所以在引入时必须引入aspectjweaver.jar包。
-   sisyphus 综合了 spring-retry 和 gauva-retrying 的优势，使用起来也非常灵活。
-   [https://github.com/houbb/sisyphus](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fhoubb%2Fsisyphus)
-   guava-retrying 模块提供了一种通用方法， 可以使用Guava谓词匹配增强的特定停止、重试和异常处理功能来重试任意Java代码。

### 5. spring-retry的重试机制

Spring Retry 为 Spring 应用程序提供了声明性重试支持。 它用于Spring批处理、Spring集成、Apache Hadoop(等等)的Spring。

This is a small extension to Google’s Guava library to allow for the creation of configurable retrying strategies for an arbitrary function call, such as something that talks to a remote service with flaky uptime.

### 6. maven 依赖

 <dependency>
  <groupId>org.springframework.retry</groupId>
  <artifactId>spring-retry</artifactId>
 </dependency>

### 7. 启用重试功能

启动类上面添加@EnableRetry注解，启用重试功能，或者在使用retry的service上面添加也可以，或者Configuration配置类上面。建议所有的Enable配置加在启动类上，可以清晰地统一管理使用的功能。

image

### 8. 添加@Retryable和@Recover注解

##### @Retryable注解，被注解的方法发生异常时会重试

-   value：指定发生的异常进行重试
-   include：和value一样，默认空，当exclude也为空时，所有异常都重试
-   exclude：指定异常不重试，默认空 ，当include也为空时，所有异常都重试
-   maxAttemps：重试次数，默认3
-   backoff：重试补偿机制，默认没有

```java
import com.mail.elegant.service.TestRetryService;
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Service;
import java.time.LocalTime;
 
@Service
public class TestRetryServiceImpl implements TestRetryService {
 
	@Override
	@Retryable(value = Exception.class,maxAttempts = 3,backoff = @Backoff(delay = 2000,multiplier = 1.5))
	public int test(int code) throws Exception{
		System.out.println("test被调用,时间："+LocalTime.now());
		  if (code==0){
			  throw new Exception("情况不对头！");
		  }
		System.out.println("test被调用,情况对头了！");
	
		return 200;
	}
}
```

##### @Backoff注解

-   delay:指定延迟后重试
-   multiplier:指定延迟的倍数，比如delay=5000l,multiplier=2时，第一次重试为5秒后，第二 次为10秒，第三次为20秒

##### @Recover注解

当重试到达指定次数时，被注解的方法将被回调，可以在该方法中进行日志处理。需要注意的是发生的异常和入参类型一致时才会回调。

```java
@Recover
public int recover(Exception e, int code){
   System.out.println("回调方法执行！！！！");
   //记日志到数据库 或者调用其余的方法
    return 400;
}
```


可以看到传参里面写的是 Exception e，这个是作为回调的接头暗号（重试次数用完了，还是失败，我们抛出这个Exception e通知触发这个回调方法）。对于@Recover注解的方法，需要特别注意的是：

•方法的返回值必须与@Retryable方法一致

•方法的第一个参数，必须是Throwable类型的，建议是与@Retryable配置的异常一致，其他的参数，需要哪个参数，写进去就可以了（@Recover方法中有的）

•该回调方法与重试方法写在同一个实现类里面

### **9. 注意事项**

-   由于是基于AOP实现，所以不支持类里自调用方法
-   如果重试失败需要给@Recover注解的方法做后续处理，那这个重试的方法不能有返回值，只能是void
-   方法内不能使用try catch，只能往外抛异常
-   @Recover注解来开启重试失败后调用的方法(注意,需跟重处理方法在同一个类中)，此注解注释的方法参数一定要是@Retryable抛出的异常，否则无法识别，可以在该方法中进行日志处理。