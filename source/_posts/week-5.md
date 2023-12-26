---
title: 2022.9.26 - 2022.9.30
date: 2022-09-27 11:34:20
password: 1062597351
tags:
---

> - 培训课程：测试驱动开发
>- 工作相关：增加请求信息日志切面
> - 博客更新：[MySQL](../../../08/06/mysql/)、[Spring Framework](../../../05/10/spring/)
> 

TDD的培训我感觉还可以，等有空在 [Test Driven Development](../../../09/30/tdd/) 中整理下内容，flag+1，希望有空。

接的需求比较简单，写一个用于打印日志的切面就可以了，但是写了两天才写完。

之前学AOP的时候，主要侧重于思想概念上的理解梳理，对切点表达式和自定义切面注解的实际使用不太熟练，因此这次把切面写完实现基本功能就花了大半天，还有一天半的时间花在代码风格、需求确认、测试以及各种各样的零碎问题上。现记录部分问题如下

### 尽可能的不修改现有代码，只扩展新代码

为了获取到请求URL，先是在`BaseController`的入参模型处理方法增加了`HttpServletRequest`入参来获取URL。后来查到了有个`RequestContextHolder`静态工具类也可以获取URL，于是不再修改现有`BaseController`，而是在切面中通过静态工具类获取URL。

以前实习的时候问师兄为什么知道有这样的工具可以用呢，师兄说用多了就知道了。其实所有有价值的工具必然是为了解决某个问题，而这个问题必然不是我第一个发现，那些早就遇到该问题的人大多早就提供了相应的工具来解决，而我要做的就是先谷歌一下吧。

### 尽可能避免通过异常来处理正常的逻辑检查

下面这段代码是网上查到的，作用是获取请求URL

```java
ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
String url = attributes.getRequest().getRequestURI().toString();
```

但是我并不确定这段代码是否始终能正常工作而不会抛出异常，比如空指针异常等，所以我改成了这样

```java
private String obtainRequestURL() {
    String url;
    try {
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        url = attributes.getRequest().getRequestURI().toString();
    }
    catch (Exception e) {
        url = "";
    }
    return url;
}
```

在没有发生异常的情况下，try-catch对性能的影响微乎其微，但是一旦发生异常，性能上则是灾难性的，应该尽可能避免通过异常来处理正常的逻辑检查，所以我又改成了这样

```java
private String obtainRequestURL() {
    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
    if (attributes != null) {
        HttpServletRequest request = attributes.getRequest();
        if (request != null) {
            StringBuffer url = request.getRequestURL();
            if (!StringUtils.isBlank(url)) {
                return url.toString();
            }
        }
    }
    return "";
}
```

看起来不太优雅，但是目前没想到更好的写法。

### @annotation和@within

在 [Spring Framework](../../../05/10/spring/) 的3.3.3小节有摘录书本上的关于各种Spring支持的AspectJ切点指示器的区别介绍。当时看到这里的时候觉得每个字我都认识，但是不太清楚到底在说啥，不过为了赶进度所以这个坑先放一下，等有空我再去填，现在碰到了就先填一点。

在写切面时，为了避免在切面Bean中写那些冗长的无错误提示的切点表达式字符串，希望能够定义一个注解来精准指定需要在何处织入切面，那么切面Bean中的切点表达式只需写为指示器（自定义注解）这样的形式即可。

此时，网上大部分都是选择使用`@annotation`指示器，但实际上`@annotation`指示器只对标注在方法上的注解生效；当注解标注在类上时，需要使用`@within`指示器；当注解既可能标注在类上，又可能标注在方法上时，需要使用`@within(...) || @annotation(...)`这样的切点表达式形式。

### 始终考虑异常

因为这个切面的功能就只是打日志，所以一开始写切面的时候也仅仅考虑了日志，当时觉得异常应该由异常相关的切面或者Handler来处理即可，所以我写成了这样

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface TraceLog {
    String type() default "undefined";
}
```

```java
@Around("@within(traceLog)")
public Object doAroundLogging(ProceedingJoinPoint pjp, TraceLog traceLog) throws Throwable {
    Long startTime = System.nanoTime();
    Object result = pjp.proceed();
    logging(startTime, pjp, traceLog);
    return result;
}
```

但是假如业务代码中抛出了异常，后续的`logging`就不会执行了，这个切面也就失去了作用，所以我改成了这样

```java
@Around("@within(traceLog)")
public Object doAroundLogging(ProceedingJoinPoint pjp, TraceLog traceLog) {
    Long startTime = System.nanoTime();
    Throwable throwable = null;
    Object result = null;
    try {
        result = pjp.proceed();
    }
    catch (Throwable t) {
        throwable = t;
    }
    logging(startTime, pjp, traceLog, throwable);
    return result;
}
```

### 始终避免切面影响原本的业务逻辑

由于在使用`@Around`通知拦截器时，需要显示地调用`pjp.proceed()`来执行业务代码，此时就很容易无意间修改了原本的业务逻辑。

在上面的切面代码中，即使业务代码抛出了异常，异常会被切面捕捉而不会抛出，还是会返回一个空的`result`对象，也并不会驱动后续的异常处理机制，这就修改了原本的业务逻辑，所以我又改成了这样

```java
@Around("@within(traceLog)")
public Object doAroundLogging(ProceedingJoinPoint pjp, TraceLog traceLog) throws Throwable {
    Long startTime = System.nanoTime();
    Throwable throwable = null;
    try {
        return pjp.proceed();
    }
    catch (Throwable t) {
        throwable = t;
        throw t;
    }
    finally {
        logging(startTime, pjp, traceLog, throwable);
    }
}
```

### 始终写快速的单测

之前说过补单测令我心累，所以要把测试代码和功能代码一起写，接着就会有下一件令我心累的事。

因为这次写的是切面，所以在测试切面逻辑时使用了`@SpringBootTest`来起一个SpringBoot，可是这样跑起来就很慢很慢很慢......而且这样好像也不能算是单测了......

目前还不知道这个问题可以怎么解决，先放一下，以后知道了回来补。

### 写在最后

以上记录了一部分问题，实际遇到的问题更多，所以写一个切面写了两天。

这个阶段的自己还没有训练出熟练的测试思维和产品思维，容易考虑不周到，为了避免写出漏洞百出的代码，只能花时间来不断自我质疑并改进，下周就要上环境来测试了，好运~