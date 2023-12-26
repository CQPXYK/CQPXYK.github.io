---
title: Spring Boot
date: 2022-05-22 17:25:46
tags:
---

 # 1、总概

Spring是以**简化Java EE应用程序的开发**为目标而创建的。

- Spring 2.5 引入了基于注解的组件扫描，这消除了大量针对应用程序自身组件的显式 XML 配置；
- Spring 3.0 引入了基于 Java 的配置，这是一种比XML更加安全的配置方式；
- 但在使用Spring的过程中，依旧无法避免基于XML的配置方式。

Spring Boot是以**简化Spring的配置**为目标而创建的，是约定优先于配置理念下的产物，实现自动配置，开箱即用。

# 2、使用介绍

## 2.1 启动类

Spring Boot要求`main()`方法所在的启动类：

- 必须放到根package下；

- 命名不做要求，但一般是以Application为后缀命名；

- 使用`@SpringBootApplication`注解标注；

- 可以通过`@SpringBootApplication(exclude = {...})`或者`@EnableAutoConfiguration(exclude = {...})`来指定禁用的自动配置。

  

`@SpringBootApplication`等同于以下三个注解的集合：

- `@SpringBootConfiguration`：支持在上下文中注册额外的Bean或导入额外的配置类，是Spring的标准的`@Configuration`的替代，有助于在集成测试中进行配置检测；

- `@EnableAutoConfiguration`：启用 SpringBoot 的自动装配机制（自动扫描+条件装配），详细介绍见3.1节；

- `@ComponentScan`：扫描被声明为JavaBean的类。

  

## 2.2 默认约定

1）Spring Boot默认约定采用Maven的目录结构。

2）Spring Boot默认内置了嵌入式的Web容器，支持四种嵌入式Web容器：Tomcat、Jetty、Undertow、Reactor。

3）Spring Boot默认约定的配置文件是`application.yml`：

- Spring Boot几乎所有的配置项都可以在这个文件中配置，如果不配置，则使用默认项；

- 文件位于`src/main/resources`目录下；

- 文件名必须是`application`而不是其他名称;

- 文件后缀采用`.yml`（[YAML](https://yaml.org/)）或者`.properties`，后者的优先级更高；

- 层级格式的`.yml`文件比`key=value`格式的`.properties`文件更易读；

- 使用`.yml`时：

  - 属性名的值和冒号中间必须是空格；

  - 大小写敏感；

  - 使用空格缩进表示层级关系；

  - 缩进时不能使用tab，只能空格；

  - 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可；

  - `#`表示注释，从这个字符一直到行尾，都会被解析器忽略。

    

  

## 2.3 优先级配置

Spring Boot提供了一种优先级配置的机制，优先级高的配置会覆盖优先级低的配置。

- 优先级最高的是命令行参数配置；

  - 命令行参数的优先级之所以被设置为最高，是因为可以方便我们在测试或生产环境中快速地修改配置参数值，而不需要重新打包和部署应用；

  - 以`--`开头的命令行参数会被转化成应用中可以使用的配置参数，如`--name = Alex` 会设置配置参数`name` 的值为`Alex`；

    

- 不同位置的配置文件的优先级从高到低依次为：

  - 项目目录的`/config`子目录；

  - 项目目录；

  - `src/main/resources/config`子目录；

  - `src/main/resources`子目录（默认配置）；

    

- 配置文件的优先级高于应用Java配置类：

  - 任何`@Component`或`@Configuration`都能注解`@Profile`，从而限制加载它的环境；

  - 通常的应用部署会包含开发、测试、预发和生产等若干个环境。

    ```java
    @Configuration
    @Profile("production")
    public class ProductionConfiguration {...}
    ```

  

## 2.4 读取配置信息

Spring Boot可以通过以下方式来读取配置信息：

- `@Value("${property}")`：适用于只需获取某单一配置项的值的场景；

- `@ConfigurationProperties(prefix = "xxx")`：使用该注解来定义一个`XXXProperties `类或者`getXXXProperties`方法，读取以key以`xxx`为前缀的信息，适用于需要批量获取多个配置项的场景；

  - 配合`@Validated`注解可以对参数实现校验功能。

  - `XXXProperties `类配合`@Component`注解或者`getXXXProperties`方法配合`@Bean `注解就能像使用普通 JavaBean 一样，将`XXXProperties `类注入到其他类中使用；

  - 如果不使用`@Component`或则`@Bean `注解将其声明为Bean，那么需要在启动类或者被加载的配置类上标注`@EnableConfigurationProperties(XXXProperties.class)`注解来注册`XXXProperties `。

    

- `@PropertySource("classpath:xxx.properties")`：使用该注解来定义一个类，读取指定的`.properties` 文件中的信息（`.yml`文件不行），同样可以将该类声明为一个JavaBean。

  

## 2.5 过滤配置类

可以通过`@Conditional`及其派生注解来判断是否过滤候选的配置类，被过滤的配置类不会被SpringBoot 的自动装配机制加载，详细介绍见3.1节

|      `@Conditional`派生注解       |                     判断条件                     |
| :-------------------------------: | :----------------------------------------------: |
|       `@ConditionalOnJava`        |            系统的java版本是否符合要求            |
|       `@ConditionalOnBean`        |                容器中存在指定Bean                |
|    `@ConditionalOnMissingBean`    |               容器中不存在指定Bean               |
|    `@ConditionalOnExpression`     |                满足SpEL表达式指定                |
|       `@ConditionalOnClass`       |                 系统中有指定的类                 |
|   `@ConditionalOnMissingClass`    |                系统中没有指定的类                |
|  `@ConditionalOnSingleCandidate`  | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
|     `@ConditionalOnProperty`      |          系统中指定的属性是否有指定的值          |
|     `@ConditionalOnResource`      |           类路径下是否存在指定资源文件           |
|  `@ConditionalOnWebApplication`   |                  当前是web环境                   |
| `@ConditionalOnNotWebApplication` |                 当前不是web环境                  |
|       `@ConditionalOnJndi`        |                  JNDI存在指定项                  |

# 3、源码学习

SpringBoot具有以下子modules：

- spring-boot：核心工程。
- spring-boot-starters：启动服务工程，可减少第三方jar的依赖。
- spring-boot-autoconfigure：实现自动配置的核心工程。
- spring-boot-actuator：外围支撑性功能。
- spring-boot-tools：开发者的常用工具集。
- spring-boot-cli：命令行交互工具。
- spring-boot-devtools：为开发者服务，其中最重要的功能就是热部署。

## 3.1 启动

生成一个Spring Boot项目时，会有一个以Application为后缀命名的启动类，启动类的`main`方法中会调用`SpringApplication`类的静态`run`方法。

```java
@SpringBootApplication
public class HelloWorldApplication {

   public static void main(String[] args) {
      SpringApplication.run(HelloWorldApplication.class, args);
   }

}
```

在spring-boot模块中，`SpringApplication`类的静态`run`方法是一个可以通过默认设置和用户约定来从具体source运行Spring项目的静态帮助器。

1）在`SpringApplication`类的静态`run`方法中，依次调用了`SpringApplication`类的构造方法和对象的`run`方法，完整代码如下

```java
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
   return run(new Class<?>[] { primarySource }, args);
}
```

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
   return new SpringApplication(primarySources).run(args);
}
```

2）在`SpringApplication`类的构造方法中，完整代码如下

- 以下代码的第8行，将不为空的`primarySources`加入到一个`LinkedHashSet`中；

- 以下代码的第9行，调用`deduceFromClasspath`方法得到Web环境类型（`NONE`、`SERVLET`、`REACTIVE`），当在JUnit测试中使用`SpringApplication`时，通常需要将环境类型设为`NONE`来使得测试类启动时只会初始化 Spring 上下文，不再启动 Tomcat 容器，从而达到加速的目的；

- 以下代码的第10、11、12行，调用`getSpringFactoriesInstances`方法，从而设置`BootstrapRegistryInitializer`列表、`ApplicationContextInitializer`列表以及`ApplicationListener`列表；

- 以下代码的第13行，调用`deduceMainApplicationClass`方法获取当前方法调用栈，找到`main`函数所在的类；

  

```java
public SpringApplication(Class<?>... primarySources) {
   this(null, primarySources);
}

public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    this.bootstrapRegistryInitializers = new ArrayList<>( getSpringFactoriesInstances(BootstrapRegistryInitializer.class));
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

3）在`getSpringFactoriesInstances`方法中，完整代码如下

- 以下代码的第8行，调用`SpringFactoriesLoader`类的`loadFactoryNames`方法来获取传入`type`所对应的实例名称，并放入到一个`LinkedHashSet`中去重；

- 以下代码的第9行，调用`createSpringFactoriesInstances`方法进行初始化；

- 以下代码的第10行，调用`AnnotationAwareOrderComparator`类的`sort`方法，根据Order对实例进行排序（@Order注解或者Order接口）；

  

```java
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
    return getSpringFactoriesInstances(type, new Class<?>[] {});
}

private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
   ClassLoader classLoader = getClassLoader();
   // Use names and ensure unique to protect against duplicates
   Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
   List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
   AnnotationAwareOrderComparator.sort(instances);
   return instances;
}
```

4）在`SpringFactoriesLoader`类的`loadFactoryNames`方法中，完整代码如下

- 以下代码第6行，调用`Class`的`getName`方法获取之前传入的`factoryType`所对应的类名；

- 以下代码的第7行，调用`loadSpringFactories`方法来读取所有jar包中的`META-INF/spring.factories`文件中的配置项的value值（key为上一级方法调用入参，value是一些类名）；

  

```java
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
   ClassLoader classLoaderToUse = classLoader;
   if (classLoaderToUse == null) {
      classLoaderToUse = SpringFactoriesLoader.class.getClassLoader();
   }
   String factoryTypeName = factoryType.getName();
   return loadSpringFactories(classLoaderToUse).getOrDefault(factoryTypeName, Collections.emptyList());
}
```

5）在`SpringApplication`类的`createSpringFactoriesInstances`方法中，完整代码如下

- 以下代码的第4行，遍历刚刚加载得到的类名；

- 以下代码的第6行，根据类名获取到类对象；

- 以下代码的第7行，检测获取的类对象类型是否与需要的一致；

- 以下代码的第8行，获取默认的无参构造方法（传入的`parameterTypes`为空）；

- 以下代码的第9行，通过反射调用类的构造方法来实例化对象；

- 以下代码的第10行，将对象放入`List`结果集；

  

```java
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
      ClassLoader classLoader, Object[] args, Set<String> names) {
   List<T> instances = new ArrayList<>(names.size());
   for (String name : names) {
      try {
         Class<?> instanceClass = ClassUtils.forName(name, classLoader);
         Assert.isAssignable(type, instanceClass);
         Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
         T instance = (T) BeanUtils.instantiateClass(constructor, args);
         instances.add(instance);
      }
      catch (Throwable ex) {
         throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
      }
   }
   return instances;
}
```

6）在`SpringApplication`类对象的`run`方法中，完整代码如下

- 以下代码的第5行，调用`configureHeadlessProperty`方法，将JVM系统属性中的`java.awt.headless`属性设置为`true`，表示运行在服务器端；

- 以下代码的第6行，依次调用了`getRunListeners`方法 --> `getSpringFactoriesInstances`方法 --> `SpringFactoriesLoader`类的`loadFactoryNames`方法 --> 读取`META-INF/spring.factories`文件中的配置项的value值，从而设置`SpringApplicationRunListener`列表，这里列表中只有一个value值（`EventPublishingRunListener`）；

- 以下代码的第7行，依次调用了`SpringApplicationRunListeners`类的`starting`方法 --> `EventPublishingRunListener`类的`starting`方法 --> `SimpleApplicationEventMulticaster`类的`multicastEvent`方法 --> `invokeListener`方法 --> `doInvokeListener`方法 --> `ApplicationListener`接口的`onApplicationEvent`方法，广播了`ApplicationStartingEvent`类型的事件并启动了对此感兴趣的监听器；

- 以下代码的第9行，依次调用了`DefaultApplicationArguments`的构造方法 --> `Source`的构造方法 --> `SimpleCommandLinePropertySource`的构造方法和`parse`方法 --> `CommandLinePropertySource`的构造方法 --> `EnumerablePropertySource`的构造方法 --> `PropertySource`的构造方法，封装了 `main` 方法的参数，从而更方便的解析和获取参数中的值；

- 以下代码的第10行，调用了`prepareEnvironment`方法，从中逐步

  - 通过`getOrCreateEnvironment`方法获取或创建了`ConfigurableEnvironment`；

  - 通过`configureEnvironment`方法依次配置了`PropertySources`和`Profile`属性；

  - 通过`ConfigurationPropertySources`类的`attach`方法将`application.properties`或者`application.yml`作为一个`PropertySource`加到`PropertySources`中；

  - 通过`SpringApplicationRunListeners`类的`environmentPrepared`方法广播事件，通知所有的观察者环境已经准备好了，可以进行后续操作；

    

- 以下代码的第12行，调用了`printBanner`方法，打印`banner`，即每次启动输出的SpringBoot的图标；

- 以下代码的第13行，调用了`createApplicationContext`方法，根据当前环境类型（在`SpringApplication`类的构造方法中确定）来创建对应类型的上下文；

- 以下代码的第15行，依次调用了`prepareContext`方法 --> `ApplicationContextInitializer`列表的`initialize`方法和`SpringApplicationRunListeners`类的`contextPrepared`方法和`contextLoaded`方法；

- 以下代码的第16行，调用了`refreshContext`方法，从中逐步

  - 通过`SpringApplicationShutdownHook`类的`registerApplicationContext`方法来注册一个名为"SpringApplicationShutdownHook"的新线程，如果 JVM 意外终止了，这个线程会把当前的上下文关闭；

  - 通过`refresh`方法来调用`AbstractApplicationContext`类的`refresh`方法，**实现IoC容器的初始化及Bean的生命周期中的部分过程，具体可参见 [Spring Framework](../../10/spring/) 的2.4节**，如果是web应用还会创建嵌入式的Tomcat；

    

- 以下代码的第17行，调用了一个空的`afterRefresh`的方法，用于之后可能要增加的操作；

- 以下代码的第20行，打印日志，显示启动当前应用所花的时间；

- 以下代码的第22行，调用`SpringApplicationRunListeners`类的`started`方法来广播`ApplicationStartedEvent`类型的事件；

- 以下代码的第23行，依次调用了`callRunners`方法 --> 所有`ApplicationRunner`和`CommandLineRunner`对象的`run`方法（同步调用，并非开启新线程）；

- 以下代码的26、27、34、35行，调用了`handleRunFailure`方法来处理异常并抛出`IllegalStateException`异常，在`handleRunFailure`方法中会继续调用`SpringApplicationRunListeners`类的`failed`方法；

- 以下代码的31行，调用`SpringApplicationRunListeners`类的`ready`方法来广播`ApplicationReadyEvent `类型的事件，至此整个`run`方法执行完毕，程序启动完成。

  

```java
public ConfigurableApplicationContext run(String... args) {
   long startTime = System.nanoTime();
   DefaultBootstrapContext bootstrapContext = createBootstrapContext();
   ConfigurableApplicationContext context = null;
   configureHeadlessProperty();
   SpringApplicationRunListeners listeners = getRunListeners(args);
   listeners.starting(bootstrapContext, this.mainApplicationClass);
   try {
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
      ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
      configureIgnoreBeanInfo(environment);
      Banner printedBanner = printBanner(environment);
      context = createApplicationContext();
      context.setApplicationStartup(this.applicationStartup);
      prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
      refreshContext(context);
      afterRefresh(context, applicationArguments);
      Duration timeTakenToStartup = Duration.ofNanos(System.nanoTime() - startTime);
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), timeTakenToStartup);
      }
      listeners.started(context, timeTakenToStartup);
      callRunners(context, applicationArguments);
   }
   catch (Throwable ex) {
      handleRunFailure(context, ex, listeners);
      throw new IllegalStateException(ex);
   }
   try {
      Duration timeTakenToReady = Duration.ofNanos(System.nanoTime() - startTime);
      listeners.ready(context, timeTakenToReady);
   }
   catch (Throwable ex) {
      handleRunFailure(context, ex, null);
      throw new IllegalStateException(ex);
   }
   return context;
}
```

## 3.2 事件监听器

在3.1节中学习到了Spring Boot启动过程中会发布一系列事件并启动事件监听器，本节对此进行详细介绍。

1）在`SpringApplication`类的构造方法中

- 会调用`setListeners`方法和`getSpringFactoriesInstances`方法，来读取到`META-INF/spring.factories`文件中key为`ApplicationListener`的全限定名的配置项的value值，从而获取并注册监听器；

  ```java
  setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
  ```

- 在spring-boot模块中，`META-INF/spring.factories`文件中存在以下配置项

  ```java
  # Application Listeners
  org.springframework.context.ApplicationListener=\
  org.springframework.boot.ClearCachesApplicationListener,\
  org.springframework.boot.builder.ParentContextCloserApplicationListener,\
  org.springframework.boot.context.FileEncodingApplicationListener,\
  org.springframework.boot.context.config.AnsiOutputApplicationListener,\
  org.springframework.boot.context.config.DelegatingApplicationListener,\
  org.springframework.boot.context.logging.LoggingApplicationListener,\
  org.springframework.boot.env.EnvironmentPostProcessorApplicationListener
  ```

2）在`SpringApplication`类对象的`run`方法中，会调用`getRunListeners`方法，完整代码如下

- 以下代码的第3、4行，调用了`SpringApplicationRunListeners`类的构造方法和`getSpringFactoriesInstances`方法，最终会读取到`META-INF/spring.factories`文件中key为`SpringApplicationRunListener`的全限定名的配置项的value值，从而设置`SpringApplicationRunListener`列表；

- 由于spring-boot模块中的`META-INF/spring.factories`文件中存在以下配置项，因此`SpringApplicationRunListener`列表中只有`EventPublishingRunListener`这一个值；

  ```java
  # Run Listeners
  org.springframework.boot.SpringApplicationRunListener=\
  org.springframework.boot.context.event.EventPublishingRunListener
  ```

```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
   Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
   return new SpringApplicationRunListeners(logger,
         getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args),
         this.applicationStartup);
}
```

3）在`SpringApplication`类对象的`run`方法中，会调用`SpringApplicationRunListeners`类的`starting`方法，进而调用列表中每一个`SpringApplicationRunListener`的`starting`方法，即`EventPublishingRunListener`类的`starting`方法，完整代码如下

- 以下代码的第3行，`this.initialMulticaster`是一个`SimpleApplicationEventMulticaster`类型的对象；

- 以下代码的第4行，调用了`SimpleApplicationEventMulticaster`类的`multicastEvent`方法，方法入参是`ApplicationStartingEvent`类型的事件；

  

```java
@Override
public void starting(ConfigurableBootstrapContext bootstrapContext) {
   this.initialMulticaster
         .multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
}
```

4）在`SimpleApplicationEventMulticaster`类的`multicastEvent`方法中，完整代码如下

- 依次调用了`resolveDefaultEventType`方法 --> `ResolvableType`类的`forInstance`方法来获取事件的类型；

- 继续调用入参中包含事件类型的`multicastEvent`方法；

  

```java
@Override
public void multicastEvent(ApplicationEvent event) {
   multicastEvent(event, resolveDefaultEventType(event));
}

private ResolvableType resolveDefaultEventType(ApplicationEvent event) {
    return ResolvableType.forInstance(event);
}
```

5）在`ResolvableType`类的`forInstance`方法中，完整代码如下

- 以下代码的第3、4行，判断事件`instance`是否实现了`ResolvableTypeProvider`接口（该接口表示事件可以被解析），是的话则强转成`ResolvableTypeProvider`类型，然后获取并返回事件类型；

- 以下代码9行，将传入的`instance`包装成`ResolvableType`，并返回一个默认类型；

  

```java
public static ResolvableType forInstance(Object instance) {
   Assert.notNull(instance, "Instance must not be null");
   if (instance instanceof ResolvableTypeProvider) {
      ResolvableType type = ((ResolvableTypeProvider) instance).getResolvableType();
      if (type != null) {
         return type;
      }
   }
   return ResolvableType.forClass(instance.getClass());
}
```

6）在`SimpleApplicationEventMulticaster`类的`multicastEvent`方法中，完整代码如下

- 以下代码的第5行，调用了`AbstractApplicationEventMulticaster`类的`getApplicationListeners`方法来获取对当前事件感兴趣的监听器；

- 以下代码的第10行，依次调用了`invokeListener`方法 --> `doInvokeListener`方法 --> `ApplicationListener`接口的`onApplicationEvent`方法来广播事件并启动监听器；

  

```java
@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
   ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
   Executor executor = getTaskExecutor();
   for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
      if (executor != null) {
         executor.execute(() -> invokeListener(listener, event));
      }
      else {
         invokeListener(listener, event);
      }
   }
}
```

7）在`AbstractApplicationEventMulticaster`类的`getApplicationListeners`方法中，完整代码如下

- 以下代码的第12行，尝试直接从缓存中获取监听器，如果`getApplicationListeners`方法已经执行过，这里就可以直接拿到缓存，如果是第一次执行`getApplicationListeners`方法，则只能获取到`null`；

- 以下代码的第35行，调用了`retrieveApplicationListeners`方法来获取监听器；

  

```java
protected Collection<ApplicationListener<?>> getApplicationListeners(
      ApplicationEvent event, ResolvableType eventType) {

   Object source = event.getSource();
   Class<?> sourceType = (source != null ? source.getClass() : null);
   ListenerCacheKey cacheKey = new ListenerCacheKey(eventType, sourceType);

   // Potential new retriever to populate
   CachedListenerRetriever newRetriever = null;

   // Quick check for existing entry on ConcurrentHashMap
   CachedListenerRetriever existingRetriever = this.retrieverCache.get(cacheKey);
   if (existingRetriever == null) {
      // Caching a new ListenerRetriever if possible
      if (this.beanClassLoader == null ||
            (ClassUtils.isCacheSafe(event.getClass(), this.beanClassLoader) &&
                  (sourceType == null || ClassUtils.isCacheSafe(sourceType, this.beanClassLoader)))) {
         newRetriever = new CachedListenerRetriever();
         existingRetriever = this.retrieverCache.putIfAbsent(cacheKey, newRetriever);
         if (existingRetriever != null) {
            newRetriever = null;  // no need to populate it in retrieveApplicationListeners
         }
      }
   }

   if (existingRetriever != null) {
      Collection<ApplicationListener<?>> result = existingRetriever.getApplicationListeners();
      if (result != null) {
         return result;
      }
      // If result is null, the existing retriever is not fully populated yet by another thread.
      // Proceed like caching wasn't possible for this current local attempt.
   }

   return retrieveApplicationListeners(eventType, sourceType, newRetriever);
}
```

8）在`retrieveApplicationListeners`方法中，截取了部分代码如下

- 以下代码的第11、12行，获取了所有的监听器实例及其对应的beanName；

- 以下代码的第17行，逐个遍历每个监听器实例；

- 以下代码的第18行，调用了`supportsEvent`方法来判断监听器是否对当前事件感兴趣；

- 以下代码的第32行，根据Order接口或者注解对监听器进行排序；

- 以下代码的第33~42行，对缓存进行一次刷新，把以前的缓存清空，将这次运行的结果进行缓存；

  

```java
private Collection<ApplicationListener<?>> retrieveApplicationListeners(
      ResolvableType eventType, @Nullable Class<?> sourceType, @Nullable CachedListenerRetriever retriever) {

   List<ApplicationListener<?>> allListeners = new ArrayList<>();
   Set<ApplicationListener<?>> filteredListeners = (retriever != null ? new LinkedHashSet<>() : null);
   Set<String> filteredListenerBeans = (retriever != null ? new LinkedHashSet<>() : null);

   Set<ApplicationListener<?>> listeners;
   Set<String> listenerBeans;
   synchronized (this.defaultRetriever) {
      listeners = new LinkedHashSet<>(this.defaultRetriever.applicationListeners);
      listenerBeans = new LinkedHashSet<>(this.defaultRetriever.applicationListenerBeans);
   }

   // Add programmatically registered listeners, including ones coming
   // from ApplicationListenerDetector (singleton beans and inner beans).
   for (ApplicationListener<?> listener : listeners) {
      if (supportsEvent(listener, eventType, sourceType)) {
         if (retriever != null) {
            filteredListeners.add(listener);
         }
         allListeners.add(listener);
      }
   }

   // Add listeners by bean name, potentially overlapping with programmatically
   // registered listeners above - but here potentially with additional metadata.
   if (!listenerBeans.isEmpty()) {
      ...
   }

   AnnotationAwareOrderComparator.sort(allListeners);
   if (retriever != null) {
      if (filteredListenerBeans.isEmpty()) {
         retriever.applicationListeners = new LinkedHashSet<>(allListeners);
         retriever.applicationListenerBeans = filteredListenerBeans;
      }
      else {
         retriever.applicationListeners = filteredListeners;
         retriever.applicationListenerBeans = filteredListenerBeans;
      }
   }
   return allListeners;
}
```

9）在`supportsEvent`方法中，通过`GenericApplicationListener`类的`supportsEventType`和`supportsSourceType`方法，判断了判断监听器是否支持当前事件以及是否对这个事件的发起来类感兴趣，完整代码如下；

```java
protected boolean supportsEvent(
      ApplicationListener<?> listener, ResolvableType eventType, @Nullable Class<?> sourceType) {

   GenericApplicationListener smartListener = (listener instanceof GenericApplicationListener ?
         (GenericApplicationListener) listener : new GenericApplicationListenerAdapter(listener));
   return (smartListener.supportsEventType(eventType) && smartListener.supportsSourceType(sourceType));
}
```

10）至此，完整介绍了`ApplicationStartingEvent`事件的广播及启动对此感兴趣的监听器的过程，这一逻辑的入口在`SpringApplicationRunListeners`类的`starting`方法，该方法是在``SpringApplication``类对象的`run`方法中被调用的；

11）在`SpringApplication`类对象的`run`方法中

- 还会调用`SpringApplicationRunListeners`类的`environmentPrepared`、`contextPrepared`、`contextLoaded`、`started`和`ready`方法，如果抛出异常，还会调用`failed`方法；

- 与上述过程同理，这些方法调用分别会实现`ApplicationEnvironmentPreparedEvent`、`ApplicationContextInitializedEvent`、`ApplicationPreparedEvent`、`ApplicationStartedEvent`、`ApplicationReadyEvent`以及`ApplicationFailedEvent`事件的广播，并启动对此感兴趣的监听器。

  

## 3.3 自动配置

启动类使用`@SpringBootApplication`注解标注，而`@SpringBootApplication`注解包含了`@EnableAutoConfiguration`注解。

在spring-boot-autoconfigure模块中，`@EnableAutoConfiguration`注解会启用 SpringBoot 的自动装配机制（自动扫描+条件装配）。

1）`@EnableAutoConfiguration`引入了`AutoConfigurationImportSelector`类，这是用于选择给容器导入哪些组件的选择器，会将符合条件的配置类都加载到IoC容器中，`AutoConfigurationImportSelector`类提供了`selectImports`方法的具体实现；

```java
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {...}
```

2）在`AutoConfigurationImportSelector`类的`selectImports`方法中，调用了`getAutoConfigurationEntry`方法，完整代码如下；

```java
@Override
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    }
    AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
    return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

3）在`AutoConfigurationImportSelector`类的`getAutoConfigurationEntry`方法中，会返回需要被加载的自动配置类，完整代码如下

- 以下代码的第5行，会调用`getAttributes`方法来获取元注解中的属性；

- 以下代码的第6行，会依次调用`getCandidateConfigurations`方法 --> `SpringFactoriesLoader`类的`loadFactoryNames`方法；

- 以下代码的第11行，会根据候选配置类上的`@ConditionalOnClass`注解来判断是否过滤该候选类，被过滤的候选类不会被加载；

  

```java
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
    if (!isEnabled(annotationMetadata)) {
        return EMPTY_ENTRY;
    }
    AnnotationAttributes attributes = getAttributes(annotationMetadata);
    List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    configurations = removeDuplicates(configurations);
    Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = getConfigurationClassFilter().filter(configurations);
    fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationEntry(configurations, exclusions);
}
```

4）在`SpringFactoriesLoader`类的`loadFactoryNames`方法中，会读取`resources/META-INF/spring.factories`文件中key为`EnableAutoConfiguration`的全限定名的value；

- `EnableAutoConfiguration`的全限定名所对应的value有很多（一百多个），每个value都是一个配置类的全限定名；

- SpringBoot 的自动装配机制并不会加载全部value所对应的配置类，而是会根据配置类的`@ConditionalOnClass`等注解所确定的条件来判断是否加载该配置类；

  

5）因此，如果需要`@EnableAutoConfiguration`注解去加载自定义jar包中的Bean，就应该确保：

- 该jar包的配置类中编写了标注了`@Bean`的方法来创建并返回该Bean；

- 该jar包的`src/main/resources/META-INF`目录下存在`spring.factories`文件，并在`spring.factories`文件配置了key为`EnableAutoConfiguration`的全限定名，value为配置类的全限定名的配置项；

  

6）总结与注意事项

- Spring Boot会根据`@EnableAutoConfiguration`注解来猜测开发者想要如何配置Spring并实现自动配置；

- 自动配置被设计成与“starter”一起工作，但是这两个概念并没有直接联系在一起，开发者可以在启动器之外自由选择jar包；

- 自动配置是非侵入性的，开发者在任何时候都可以开始定义自己的配置来替换自动配置的特定部分；

- 在jar包的`src/main/resources/META-INF/spring.factories`文件中，开发者还可以添加配置项来指定自动配置类的加载条件，`@EnableAutoConfiguration`会根据加载条件来判断是否加载某配置类；

- 这些自动配置类被设置为`public`只是为了方便禁用它，实际上这些自动配置类是仅供内部使用的，不建议直接使用这些类的实际内容（例如嵌套的配置类或Bean方法）。

  

## 3.4 Web配置

添加了Spring -boot-starter-web依赖项后，Spring Boot会自动添加Tomcat和Spring MVC，自动配置会假定正在开发一个web应用程序，并相应地设置Spring。

在3.2节中学习到了`@EnableAutoConfiguration`注解会加载`spring.factories`文件中所配置的key为`EnableAutoConfiguration`的全限定名，value为配置类的全限定名的配置项。

本节以`WebMvcAutoConfiguration`这一自动配置类为例，介绍Spring Boot的Web配置。

1）`WebMvcAutoConfiguration`是针对Web配置的自动配置类，截取了部分代码如下

- 以下代码的1、2、6行，通过`@AutoConfigure`和`@AutoConfigureOrder`注解给出了`WebMvcAutoConfiguration`配置类的加载顺序；

- 以下代码的3~5行，通过`@Conditional`及其派生注解给出了`WebMvcAutoConfiguration`配置类的加载条件；

  

```java
@AutoConfiguration(after = { DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
      ValidationAutoConfiguration.class })
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
public class WebMvcAutoConfiguration {...}
```

2）`WebMvcAutoConfiguration`配置类内部有一些静态内部类

- `EnableWebMvcConfiguration`配置类

  - 相当于`@EnableWebMvc`注解的功能；

  - 通过`@EnableConfigurationProperties(WebProperties.class)`注册了`WebProperties`，

  - 重写了一些`WebMvcConfigurationSupport`父类中的方法实现并声明为Bean方法；

    

- `ResourceChainCustomizerConfiguration`配置类

  - 由`OnEnabledResourceChainCondition`来判断是否加载；

  - 默认是不进行加载的；

    

3）`WebMvcAutoConfiguration`配置类中声明了两个Bean方法，注册条件如下

```java
@Bean
@ConditionalOnMissingBean(HiddenHttpMethodFilter.class)
@ConditionalOnProperty(prefix = "spring.mvc.hiddenmethod.filter", name = "enabled")
public OrderedHiddenHttpMethodFilter hiddenHttpMethodFilter() {
   return new OrderedHiddenHttpMethodFilter();
}

@Bean
@ConditionalOnMissingBean(FormContentFilter.class)
@ConditionalOnProperty(prefix = "spring.mvc.formcontent.filter", name = "enabled", matchIfMissing = true)
public OrderedFormContentFilter formContentFilter() {
   return new OrderedFormContentFilter();
}
```

# 4、参考资料

- https://www.jianshu.com/p/f3337a836faf
- https://snailclimb.gitee.io/springboot-guide/#/
- https://www.liaoxuefeng.com/wiki/1252599548343744/1266265175882464
- https://blog.csdn.net/qq_26000415/category_9271293.html
- https://juejin.cn/post/6910458302547623944
- https://juejin.cn/post/6844904106843193357
- https://juejin.cn/post/6844904106847371271
- https://docs.spring.io/spring-boot/docs/2.7.0/reference/html/
- https://docs.spring.io/spring-boot/docs/2.7.0/api/