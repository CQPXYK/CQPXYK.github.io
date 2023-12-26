---
title: 2023.8.21 - 2023.8.25
date: 2023-08-22 18:06:57
password: 1062597351
tags:
---

> - 工作相关：ExecutorService, CyberArk Configuration
> - 博客更新：[Java Concurrency](../../../../2022/05/08/java-concurrency/)、[Operating Systems](../../../../2023/08/20/operating-systems/)、[Docker](../../../../2022/09/06/docker/)

### ExecutorService

一般在类的成员变量定义线程池，懒加载，并且通过`shutdown`确保调用 JVM 在回收线程池之前先回收池内线程。

```java
@Component
public class ExecutorUtil {
    
    public static final Map<String, ExecutorService> executors = new ConcurrentHashMap<>();
    
    public static ExecutorService getExecutorService(String scenario) {
        return executors.computeIfAbsent(scenario, s -> new ThreadPoolExecutor(
        0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS, new SynchronousQueue<>(), 
            new ThreadFactoryBuilder().setNameFormat(s).build()));
    }
    
    @PreDestory
    protected static void shutdownAll() {
        executors.values().forEach(ExecutorService::shutdown);
    }
    
}
```

### CyberArk Configuration

使用 CyberArk 来构建 DataSource。

<img src="1.png" alt="" style="zoom:80%;" />

<img src="2.png" alt="" style="zoom:80%;" />

<img src="3.png" alt="" style="zoom:80%;" />

<img src="4.png" alt="" style="zoom:80%;" />

<img src="5.png" alt="" style="zoom:80%;" />

<img src="6.png" alt="" style="zoom:80%;" />

<img src="7.png" alt="" style="zoom:80%;" />

