---
title: 2022.10.10 - 2022.10.14
date: 2022-10-10 22:38:59
password: 1062597351
tags:
---

> - 工作相关：切面测试、环境部署、日志过滤（BLOB）、慢加载分析
>- 博客更新：博客页面样式、[MySQL](../../../08/06/mysql/)

### 切面测试

关于切面的测试，至少应当测试两种逻辑，一个是切面织入的逻辑，一个切面本身的逻辑。

- 在测试切面织入逻辑时，关注的是切面是否在正确的时机被成功执行了，以防切面失效。`@MockBean`并不适合于这类测试，这是由于`@MockBean`在生成新的代理时将直接忽略掉相关切面的注解，导致切面直接失效，因此使用`@MockBean`来模拟Aspect则会发生错误，而应该使用`@SpyBean`；

- 在测试切面本身的逻辑时，关注的是切面中到底发生了什么，是否正确执行了想要给拦截方法增加的逻辑，以防切面错误。此时，像测试以往业务代码一样使用`@MockBean`测试即可。

  

相关代码如下，其中

- `@SpringBootTest`注解是为了加载应用上下文，获取测试需要的Bean；

- `@DirtiesContext`注解是为了在每次运行`TraceLogAspectTest`测试之后删除并重新加载应用程序上下文缓存，避免该测试对其他单测造成影响（这个影响是在环境部署时发现的，本地运行时并未产生影响）。

  

```java
@Slf4j
@Aspect
@Component
public class TraceLogAspect {

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

    private void logging (Long startTime, ProceedingJoinPoint pjp, TraceLog traceLog, Throwable throwable) {
        // ...
    }
}
```

```java
@DirtiesContext
@RunWith(SpringRunner.class)
@SpringBootTest
public class TraceLogAspectTest {

    @SpyBean
    private TraceLogAspect traceLogAspect;

    @SpyBean
    private MyUser myUser;

    @Mock
    private ProceedingJoinPoint pjp;

    @Mock
    private TraceLog traceLog;

    @Test
    public void testWeavingLogic() throws Throwable {
        myUser.fun();
        Thread.sleep(1000);
        Mockito.verify(traceLogAspect).doAroundLogging(Mockito.any(), Mockito.any());
    }

    @Test
    public void testAspectLogic() throws Throwable {
        Object obj = new Object();
        Mockito.when(pjp.proceed()).thenReturn(obj);
        Object result = traceLogAspect.doAroundLogging(pjp, traceLog);

        Mockito.verify(pjp).proceed();
        Assert.assertSame(obj, result);
    }
}
```

### 环境部署

为了将之前写的切面代码部署到云上来测试，在Jenkins上新建了一个环境。

在新建环境的过程中需要写一个用于建环境的脚本，并将该脚本添加到新环境的配置中。在写这个脚本的时候基本就是照着现有的例子写，等有空系统学下 Linux 之后可以再回过头来看下为什么要这样写。

部署上去之后发现切面测试对其他单测造成了影响，给切面测试标注`@DirtiesContext`注解后解决了，可见本地运行情况总是和云上有区别，在 [2022.8.29 - 2022.9.2](../../../09/01/week-1/) 中也记录过类似的问题（云平台给类自动插入了一些不在测试考虑范围内的类属性）。

### 日志过滤

通过以下配置可以令 Hibernate 将 JdbcTemplate Queries 在日志中打印出来，其中，`DEBUG`和`TRACE`的级别是低于默认级别`INFO`的。

（第2行和第3、4行选择一种即可，都用于打印 SQL 参数，区别在于打印多个参数时是否换行）

```properties
logging.level.org.hibernate.SQL=DEBUG
#logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
logging.level.org.hibernate.engine.spi=TRACE
logging.level.org.hibernate.engine.query.spi=TRACE
logging.level.org.springframework.jdbc.core.StatementCreatorUtils=TRACE
```

现在的问题是：在打印 query 时，把一些很大的二进制的参数也打印出来了，这个数据量太大太消耗内存空间，于是希望把这类参数过滤掉。这个问题在 Hibernate 5.2.3 中通过以下代码修复掉了（[参考说明](https://stackoverflow.com/questions/54120081/hibernate-trace-values-of-statement-parameters-except-blobs)）

```java
@Override
public String extractLoggableRepresentation(Blob value) {
    return value == null ? "null" : "BLOB{...}";
}
```

也就是说，`BLOB`类型的数据在 Hibernate 5.2.3 不会被完整打印出来，只会显示`BLOB{...}`，因此，只需把需要过滤的大参数类型（可能是`Byte[]`）改为`BLOB`类型，并确保 Hibernate 版本在 5.2.3 及以上，即可实现大参数过滤。

### 慢加载分析

有一个 HTTP 请求执行时间很长，这个请求中涉及到很多次查询。

现在 mentor 给出的意见是希望把多次查询以及每次查询后的一些逻辑处理（包含过滤）均合并为一次查询来直接得到结果，减少需要数据库服务端需要返回的数据量，并节省多次查询后返回给客户端的网络通信时间，不过我不太确定这样是否可行，疑惑的点在于

- MySQL 优化中有提到 MySQL 连接非常轻量，在返回一个小的查询结果方面很高效，鼓励将大查询切分为多个小查询，我不确定 oracle 是否同理，如果是的话，而现在这样把多个查询合并为一个查询是否可能降低性能，或者说这只是把优化工作从服务端转移给了 DBA？
- 目前的代码是历史遗留代码，风格有点槽糕，逻辑可读性很差，将这些逻辑改为 SQL 算是一个很大的改动，如果不是明确要求要重构代码的话，还是希望尽可能不修改现有代码。

**来自回顾时的疑惑回复**

- **先不论多个小查询和一个大查询之间孰优孰劣（我怀疑差不多），这里真正实现优化的点在于通过将多个查询合并为一个查询来减少返回不必要的数据，当不必要的数据很多时，优化效果就会很显著**
- **宁愿冒着背锅闯祸的风险，也不要错过任何可以学习成长的机会，畏手畏脚不会犯错，但也只能原地踏步**

### 博客样式更新

目前的博客主题是直接用的别人写的主题，有一些地方我试着改了下，勉强改到现在的样子，但觉得还是很有改进空间的，问题是我不太会前端，这种不会又要硬改的效率实在感人。

等我以后有空系统学前端之后自己重新写一个吧，不过按照目前的 TODO 优先级排序来看，估计是好几年以后才有空。