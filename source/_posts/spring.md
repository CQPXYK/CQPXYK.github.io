---
title: Spring Framework
date: 2022-05-10 18:32:28
tags:
---

# 1、总概

Spring是一个取代了EJB的轻量级框架，它提供了一系列底层和基础设施，是以**简化Java EE应用程序的开发**为目标而创建的。

Spring Framework主要包括几个模块：

- 核心容器：Spring最核心的部分，支持IoC，所有Spring模块都构建于核心容器之上；
- 面向切面编程：是开发切面的基础；
- Web 模块：提供了SpringMVC框架给Web应用；
- 数据访问与集成：支持JDBC和ORM的数据访问模块；
- Test 模块：提供了⼀系列的mock对象实现，使得开发者能够很方便的进行测试。

注：IoC和AOP不是Spring提出的，但Spring很好的实现了这两个思想

通过这些模块，Spring可以实现：

- 基于POJO的轻量级和最小入侵性编程；
- 基于依赖注入和面向接口实现松耦合；
- 基于切面和惯例进行声明式编程；
- 基于切面和模板减少样板式代码。

# 2、IoC 

## 2.1 问题与解决

Java组件在协助工作时，可能会反复实例化一些很复杂的组件，为了避免重复工作，考虑共享一些组件，但组件的共享会导致组件之间的依赖关系逐渐复杂，组件的创建、共享以及销毁也变得不好管理，即难以根据依赖关系创建、组装、获取和销毁组件，也很难进行单元测试。

传统应用程序中，组件的控制权在应用程序本身，在IoC模式下，组件控制权发生了反转（从应用程序转移到IoC容器），应用程序只需要**直接使用**（通过`setXXX`方法或者带`XXX`参数的构造方法来注入`XXX`实例，而不再需要`new XXX`）已经创建好配置好的组件。

IoC和DI是从两个不同的角度来描述了同一件事情，IoC是从对象的角度，对象的控制权转移（反转）给了容器；DI是从容器的角度，容器将对象所依赖的其他对象注入进去

Spring的核心就是提供了一个**可扩展的无侵入**的IoC容器， IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value），Map 中存放的是各种对象。无侵入指应用程序的组件无需实现Spring的特定接口，对Spring的IoC容器无感知。容器是一种为特定组件的运行提供必要支持的一个软件环境（组件运行环境和底层服务）。

在Spring的IoC容器中，所有的组件又称为JavaBean，配置一个组件就是在配置一个JavaBean。IoC容器可以管理所有轻量级的JavaBean的生命周期，从而将JavaBean的创建配置与使用相分离。IoC容器在创建和装配Bean时需要知道Bean间的依赖关系，依赖关系可以通过XML文件或者注解这两种方式来进行配置，这种创建应用对象之间的依赖协作关系的行为通常被称为**装配（wiring）**。

## 2.2 XML配置

在后期维护时，XML配置看起来非常清晰，但系统越大，配置内容就越大。

### 2.2.1 使用步骤

1）通过Maven创建工程并引入`spring-context`依赖；

2）通过属性注入或者依赖注入编写组件；

3）根据组件间的依赖关系编写`application.xml`配置文件，定义需要实例化对象的类的全限定类名以及类之间依赖关系描述；

4）创建IoC容器实例，并加载配置文件，使得Spring的IoC容器创建并装配好配置文件中指定的所有Bean，接着就可以从Spring的IoC容器中获取装配好的Bean；

```java
ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
Product product = (Product) context.getBean(Product.class);
```

Spring容器有两种，一种是`BeanFactory`接口（Bean工厂），另一种是`ApplicationContext`接口（应用上下文）。通常情况下都会使用应用上下文，很少使用Bean工厂。常用的`ApplicationContext`接口的实现类有以下三种：

- `ClassPathXmlApplicationContext`：从类路径下的XML配置文件中加载上下文定义，把和上下文定义文件当做资源；

- `FileSystemXmlapplicationContext`：读取文件系统下的XML配置文件并加载上下文定义；

- `AnnotationConfigApplicationContext`：在纯注解模式下自动装配并加载上下文定义；

- `XmlWebApplicationContext`：读取Web应用下的XML配置文件并装载上下文定义。

  

### 2.2.2 静态装配

- 每个Bean都有一个`id`属性，`id`属性定义了Bean的唯一标识；

- `class`属性定义Bean的类的全限定类名（这会通过反射调用无参构造器）或者工厂类的全限定类名（和`factory-method`属性一起使用）来创建对象；

- `scope`属性指定了Bean的作用域，默认是`singleton`；

  | 作用域         | 定义                                                     |
  | :------------- | -------------------------------------------------------- |
  | singleton      | 一个Bean定义只有一个对象实例（默认）                     |
  | prototype      | 每次调用都创建一个实例，允许Bean的定义可以被实例化任意次 |
  | request        | 用于一次HTTP请求，该Bean仅在当前 HTTP request 内有效     |
  | session        | 用于一个HTTP Session，该Bean仅在当前 HTTP session 内有效 |
  | global-session | 用于一个全局HTTP Session                                 |

- `factory-bean`属性指定创建当前Bean的工厂Bean的唯⼀标识，当指定了此属性之后，`class`属性失效；

- `factory-method`属性指定创建Bean的工厂方法，从而能够代替构造方法创建Bean，如果和`class`属性搭配使用，则方法必须是`static`的；

- `init-method`和`destory-method`属性分别指定Bean的初始化和销毁方法，`init-method`属性要求是无参方法，`destory-method`属性只能对单例Bean起作用；

- `default-init-method`和`default-destory-method`属性为上下文中所有的Bean都分别指定默认了初始化和销毁方法；

- `lazy-init`属性默认为`false`，代表会立即加载Bean，也就是在Bean初始化后提前进行实例化；

  - 可以通过将`lazy-init`属性设为`true`来实现延迟初始化，也就是在第一次向容器
    通过 getBean 索取 Bean 时才进行实例化；

  - 对于`scope`属性设为`pototype`的Bean，`lazy-init`属性设置不起作用，即始终保持延迟初始化；

    

- 通过`<property name = "…" value = "…" />`或者`p:xxx = "..."`来注入数据类型；

- 通过`<property name = "…" ref = "…" />`或者`p:xxx-ref = "..."`来注入另一个Bean；

- 通过`<property name = "…"> <bean class = "..." /> </property>`来注入内部Bean，内部Bean无`id`属性；

- 通过`<property name = "..."> <null/> </property>`来注入空值；

- 通过`<property name = "..." <list> <ref bean = "..." /> </list> </property>`或者`<property name = "..." <set> <ref bean = "..." /> </set> </property>`来注入数组类型或者Collection接口的实现类型；

- 通过`<property name = "..." <map> <entry key = "..." value = "..." /> </map> </property>`来注入Map类型，其中`key`和`value`可以根据需要改为`key-ref`和`value-ref`；

- 通过`<constructor-arg values = "..." />`或者`<constructor-arg ref= "..." />`或者`<constructor-arg> <bean class = "..." /> </constructor-arg>`来注入构造器参数（默认的无参构造器不需要这样注入参数。

### 2.2.3 动态装配

在XML配置文件中除了静态定义依赖关系之外，还可以通过Spring表达式语言（Spring Expression Language, SpEL）在运行期将值动态装配到Bean的属性或者构造器参数中。比如`<property name = "..." value = "#{SpEL表达式}" />`可以将`#{}`界定符中的SpEL表达式所计算得到的值装配到Bean属性中。

- SpEL表达式提供了多种运算符；

  | 作用域     | 定义                                                     |
  | :--------- | -------------------------------------------------------- |
  | 算术运算   | `+`、`-`、`*`、`/`、`%`、`^`                             |
  | 关系运算   | `<`、`>`、`==`、`<=`、`>=`、`lt`、`gt`、`eq`、`le`、`ge` |
  | 逻辑运算   | `and`、`or`、`not`、`！`                                 |
  | 条件运算   | `？：`                                                   |
  | 正则表达式 | `matches`                                                |

- `+`运算符除了进行算法运算外，还可以用于进行字符串连接；

- 在XML配置文件中使用`<=`和`>=`符号时会报错，最好使用文本替代方式

- `?.`运算符可以代替`.`运算符来调用方法，并确保访问右边方法前，左边项的值不会为`null`；

- `T()`运算符可以用于调用类作用域的方法和常量，但该运算符的真正价值在于访问指定类的静态方法和常量；

- `x ? x : y`形式的三元运算符可以简写为`x ?: y`；

- `matches`运算符对`String`类型的文本（作为左边参数）应用正则表达式（作为右边参数）；

- `[]`运算符可以从用来获取`Map`、`Properties`中的成员以及字符串中的某个字符；

- `systemEnvironment`包含了应用程序所在的机器上的所有环境变量，`systemProperties`包含了Java应用程序启动时所设置的所有属性，都可以与`[]`运算符搭配使用；

- `.?[]`是一个查询运算符，用于从集合中创建出一个新的集合，新的集合中只存放原集合中符合中括号内的表达式的成员；

- `.^[]`和`.$[]`也是查询运算符，分别用于从集合中查询出符合中括号内的表达式的第一个匹配项和最后一个匹配项；

- `.![]`是投影运算符，用于从集合中的每个成员中选定特定的属性放入一个新的集合中；

SpEL表达式虽然功能强大，但是终究是一个字符串，不易于测试，也没有IDE的语法检查的支持，只有在使用传统方式很难甚至不可能但使用SpEL却很简单的情况下才使用SpEL进行装配，尽量不要把过多逻辑放入SpEL。

### 2.2.4 自动装配

- 通过`autowire = "byName" />`可以为属性自动装配ID与该属性的名字相同的Bean；

- 通过`autowire = "byType" />`可以为属性自动装配类型与该属性的类型相同的Bean；

- 通过`autowire = "constructor" />`可以为构造器自动装配类型与构造器入参类型相同的Bean，此时不需要再声明注入构造器参数；

- `byType`和`constructor`自动装配都具有一个局限性：如果存在多个与该属性的类型相同的Bean，Spring会抛出异常，因此，要么只允许存在一个Bean与需要自动装配的属性类型相匹配，要么将其他需要被排除的Bean的`autowire-candidate`属性设为`false`；

- 通过`autowire = "autodetect" />`可以首先尝试使用`constructor`自动匹配，如果没有发现与构造器相匹配的Bean时，将尝试使用`byType`自动匹配；

- 默认情况下，XML文件的`default-autowire`属性被设置为`none`，标示XML中所有的Bean都不使用自动装配，除非Bean自己配置了`autowire`属性；

- 通过将`default-autowire`属性设置为`byName`、`byType`、`constructor`、`autodetect`中的任意一个，则可以令每一个Bean的所有属性都使用所设置的自动装配方式；

- 即便设置了默认的自动装配方式，依旧可以在每个Bean中重新设置自动装配方式来覆盖默认设置，即便使用了自动装配，依旧可以混合使用显示的装配来覆盖自动装配；

- 当使用`constructor`自动装配时，必须自动装配构造器的所有入参，不能混合使用`constructor`自动装配和`<constructor-arg`元素。

  

## 2.3 注解配置

使用注解自动装配与在XML中使用`autowire`属性自动装配并没有太大差别，但使用注解方式允许更细粒度的自动装配，可以选择性地标注某一个属性来对其应用自动装配，使用也更加方便。但当需要修改一个Bean时，无法很好的确定到底有多少个其他的Bean依赖于这个被修改的Bean。

可以通过XML来配置第三方jar中的Bean，通过注解来配置自身开发的Bean，此时是XML+注解的配置模式，XML文件依旧存在，因此Spring IoC容器的启动依旧是从加载XML开始的。

### 2.3.1 使用步骤

1）通过Maven创建工程并引入`spring-context`依赖；

2）在XML文件中进行配置；

- 配置`<context:annotation-config>`元素，可以告知Spring使用基于注解的自动装配；
- 配置`<context:component-scan>`元素（除了完成与`<context:annotation-config>`元素一样的工作，还允许Spring自动检测Bean和定义Bean，此时不再需要`<bean>`元素）；

- 会扫描`base-package`属性所指定的包及其所有子包，并查找出能够自动注册为Spring Bean的类；

3）定义配置类

- 通过`@Configuration`和`@ComponentScan`注解编写`AppConfig`配置类；

- 通常将`AppConfig`配置类位于自定义的顶层包，其他Bean按类别放入子包；

  

4）通过`@Component`和`@Autowired`等注解编写组件；

5）在`AppConfig`配置类的`main`函数中创建IoC容器实例，容器创建并装配好所有Bean，接着就可以从Spring的IoC容器中获取装配好的Bean。

```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
Product product = (Product) context.getBean(Product.class);
```

### 2.3.2 自动装配

以下注解均用于注入Bean，实现自动装配。

- `@Autowired`

  - 用于类的数据成员和函数成员，相当于把指定类型（`byType`）的Bean注入到指定字段，且不受限于`private`关键字；

  - `@Autowired`默认在指定类型的Bean不存在时报错，而`@Autowired(required = false)`则不会报错，即找到指定类型时注入，没找到时注入`null`；

  - `required`属性可以用于`@Autowired`所使用的任意地方，但当用于构造器装配时，只有一个构造器可以将`@Autowired`的`required`属性设置为`true`，其他构造器的`@Autowired`的`required`属性只能设置为`false`;

  - 当使用`@Autowired`标注多个构造器时，Spring会从满足装配条件的构造器中选择入参最多的那个构造器。

    

- `@Qualifier("beanName")`

  - 在存在多个相同类型的Bean的情况下，用于指定注入的Bean的别名，与`@Autowired`搭配使用，相当于把`byType`自动装配转换为`byName`；

  - 通过`@Qualifier`可以创建自定义的限定器注解，然后通过自定义的限定器注解来代替`@Qualifier`来标注指定注入的Bean的范围。

    

- `@Inject`

  - Java的`@Inject`几乎可以完全替代Spring的`@Autowired`；

  - 与`@Autowired`不同的是，`@Inject`没有`required`属性，因此`@Inject`标注的依赖关系必须存在，如果不存在则会抛出异常。

    

- `@Named`

  - 相对于`@Autowired`所对应的`@Qualiflier`，`@Inject`所对应的是`@Named`；

  - `javax.inject`包里有自己的`@Qualiflier`注解，但不建议使用该包中的`@Qualiflier`注解，而是鼓励使用该注解来创建自定义的限定器注解；

  - 实际上，`@Named`就是使用`javax.inject`包中的`@Qualiflier`注解所创建的限定器注解。

    

- `@Value`

  - 用于某个属性、方法或者方法参数，并传入一个基本类型或者`String`类型的值来装配属性，`String`类型还可以是一个SpEL表达式，表达式的计算结果可以是任意类型；

  - 用于注入资源，Spring提供了一个`org.springframework.core.io.Resource`（注意不是`javax.annotation.Resource`），它可以像`String`、`int`一样使用`@Value`注入；

  - 当程序需要读取一个资源文件（比如配置文件等）中的内容时，可以在`Resource`类型的数据成员`resource`上标注`@Value("classpath:/logo.txt")`，然后直接调用`resource.getInputStream()`就可以获取到classpath中的`logo.txt`文件的输入流，避免了自己搜索文件的代码。

    
  
- `@Resource`

  - 由`javax.annotation.Resource`包提供；

  - 可以通过指定name 和 type来指定注入的Bean；

  - 指定name时找不到则抛出异常，指定type时找不到或是找到多个，都会抛出异常；

  - 既没有指定name，又没有指定type时，自动按照`byName`自动装配；

  - `@Resource`在 Jdk 11中已经移除，此时使用需要单独引入jar包。

    

### 2.3.3 自动检测

以下四个注解均用于声明Bean，实现自动检测。

- `@Component`

  - 默认定义了一个`id`为小写开头的类名的Bean，`@Component("...")`则显式定义了一个指定`id`的Bean；
  
  - 如果一个 Bean 不知道属于哪个层，可以使用`@Component` 注解标注。
  
    
- `@Controller` : 对应 Spring MVC 控制层，主要用于接受用户请求并调用 Service 层返回数据给前端页面。
- `@Service` : 对应服务层，主要涉及一些复杂的逻辑，需要用到 Dao 层。
- `@Repository`：对应持久层即 Dao 层，主要用于数据库相关操作。

这四个注解的用法完全一样，只是为了更清晰的区分不同层的Bean。

在令Spring自动检测组件的过程中，也可以配置相关的过滤器来过滤组件扫描。

- 在配置`<context:component-scan>`元素时，可以通过配置`<context:include-filter`和`<context:exclude-filter`子元素来设置过滤器，分别调整要扫描和不要扫描的类型；

  ```xml
  <context:component-scan
  			base-package = "...">
      <context:include-filter type = "..." expression = "..." />
      <context:exclude-filter type = "..." expression = "..." />
  </context:component-scan>
  ```

- 其中，`type`属性用于指定要使用的过滤器类型；

  | 过滤器类型   | 描述                                                         |
  | :----------- | ------------------------------------------------------------ |
  | `annotation` | 扫描使用指定注解所标注的类，由`expression`属性指定要扫描的注解 |
  | `assignable` | 扫描派生于`expression`属性所指定类型的类                     |
  | `aspectj`    | 扫描与`expression`属性所指定的AspectJ表达式所匹配的类        |
  | `regex`      | 扫描类名称与`expression`属性所指定的正则表达式所匹配的类     |

 ### 2.3.4 基于Java的配置

在基于XML文件的配置中，Bean的类型和ID都是由字符串来标示的，这就导致它们无法进行编译期检查，写起来也很繁琐，而**在基于Java的配置中，则可以进行编译期检查来确保Bean的类型是合法类型**。Spring Boot偏向使用基于Java的配置。

- `@Configuration`

  - 通过在XML文件中配置`<context:component-scan>`元素，还可以告知Spring在包内自动加载使用`@Configuration`注解所标注的所有类；

  - `@Configuration`注解等同于XML文件中的`<beans>`元素，它所标注的类是一个配置类；

  - 配置类会包含一个或多个Spring Bean的定义，这些Bean的定义是使用`@Bean`注解所标注的方法。

    

- `@Bean`

  - 用于方法，当一个Bean不在`AppConfig`配置类所在的包中时，可以在`AppConfig`配置类中编写标注了`@Bean`的方法来创建并返回该Bean，方法名将作为该Bean的ID；

  - 也可以通过`@Bean("...")`或者`@Bean`+`@Qualifier("...")`指定别名；

  - 通过声明方法引用一个Bean并不等同于调用该方法，区别在于每次调用该方法都会得到一个新的实例，而通过`@Bean`标注时，Spring会拦截该方法的调用，并尝试在应用上下文中查找该Bean，这意味着该Bean是Singleton。

    

- `@ComponentScan`

  - 用于类，告知容器自动搜索当前类所在的包以及子包，可代替`<context:component-scan>`元素；

  - 将所有标注为`@Component`、`@Controller`、`@Service`、`@Repository`的Bean自动创建出来，并根据`@Autowired`进行装配。

    
  
- `@Import`

  - 不需要把所有的配置都放到一个配置类中；

  - `@Configuration`标注的配置类可以通过`@Import`来导入其他的配置类，把多个分开的容器配置合并在一个配置中。

    
  
- `@ImportResource`

  - 用于导入Spring的配置文件，让配置文件里面的内容生效；

  - 如果必须要使用基于XML的配置，依旧定义一个`@Configuration`配置类，然后通过`@ImportResource`来加载XML配置文件。

    

### 2.3.5 补充介绍

- `@Scope`

  - 用于类，单独使用`@Component`注解时会默认将组件标注为Singleton，即每次通过`getBean`方法获取到Bean总是同一个实例；

  - 增加`@Scope`注解时，则可以将组件标注为不同的作用域的Bean，例如`@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)`将组件标注为Prototype（原型）。

    

- `@PostConstruct`

  - 用于Bean在注入必要依赖后的`init`方法，从而进行初始化；

  - Spring只根据`@PostConstruct`查找无参数方法，对方法名不作要求。

    

- `@PreDestory`

  - 用于Bean在容器关闭时的`shutdown`方法，从而进行相关的清理操作；

  - Spring只根据`@PreDestory`查找无参数方法，对方法名不作要求。

    

- `@Primary`

  - 在存在多个相同类型的Bean的情况下，用于指定优先注入的Bean，则注入时不需要指出该优先注入的Bean的别名；

  - 相同类型的Bean只有一个可以指定为`@Primary`，其他必须用`@Qualiflier("beanNname")`指定别名。

    

- `@PropertySource("app.properties")`

  - 用于`AppConfig`配置类，告知容器自动读取名为`app.properties`的配置文件，接着可以使用`@Value("${key:defaultValue}")`来注入配置文件中`key`的`value`，这比使用`@Value("classpath:/app.properties")`+输入流的方式来读取配置文件更加简单；

  - 还有一种方式是先通过一个Bean（Spring容器中的默认类名称为`SmtpConfig`）持有所有的配置属性值，然后在其他Bean中使用`@value("#{smtpConfig.property}")`来注入属性值。

    

- `@Profile`

  - 用于配置不同环境下的Bean，程序运行时通过指定运行环境就可以使得容器只创建该环境下的Bean；

  - Spring提供了`native`（开发）、`test`（测试）和`production`（生产）这三种环境。

    

- `@Conditional`

  - 用于Bean声明；
  - 提供了Bean的装载条件判断，从而确定是否创建该Bean。

## 2.4 IoC特性

### 2.4.1 工厂Bean

Spring中Bean有两种，⼀种是普通Bean，⼀种是工厂Bean（`FactoryBean`），`FactoryBean`可以生成某个类型的Bean实例，因此，可以借助于它自定义Bean的创建过程。

Spring提供了工厂模式来创建Bean，过程如下：

1）假如需要以工厂模式创建`Product`，首先需要定义一个工厂Bean（`ProductFactoryBean`），该工厂Bean需要实现`FactotyBean`接口；

2）Spring会优先实例化工厂Bean（`ProductFactoryBean`），然后通过工厂Bean的`getObject`方法创建真正的Bean（`Product`）；

3）因此，如果定义了一个工厂Bean，Spring创建的Bean实际上是这个工厂Bean的`getObject`方法所返回的Bean。

```java
@Component
public class ProductFactoryBean implements FactoryBean<Product> {
    
    String product= "P";

    @Override
    public Product getObject() throws Exception {
        return Product.of(product);
    }

    @Override
    public Class<?> getObjectType() {
        return Product.class;
    }
    
    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

### 2.4.2 Bean工厂

`BeanFactory`接口（Bean工厂）是Spring框架中IoC容器的顶层接口，它只是用来定义一些基础功能和基础规范，是SpringIOC的基础容器。`ApplicationContext`是`BeanFactory`的一个子接口，具备`BeanFactory`提供的全部功能，是IoC容器的高级接口，比`BeanFactory`拥有更多的功能。

IoC容器初始化（获取`BeanFactory`）的关键之处在于`org.springframework.context.support.AbstractApplicationContext#refresh`

1）在`AbstractApplicationContext`类的`refresh`方法中

- 依次调用`obtainFreshBeanFactory`方法 --> `refreshBeanFactory`方法和`getBeanFactory`方法；

- `AbstractRefreshableApplicationContext`类提供了`refreshBeanFactory`方法和`getBeanFactory`方法的具体实现来创建并返回Bean工厂；

  

2）在`AbstractRefreshableApplicationContext`类的`refreshBeanFactory`方法中，完整代码如下

- 以下代码的第8行，调用`createBeanFactory`方法来创建`DefaultListableBeanFactory`类型的`beanFactory`；

- 以下代码的第11行，调用`loadBeanDefinitions`方法来加载`beanFactory`中的`BeanDefinition`（JavaBean在IoC容器内部的数据结构类型）；

- `AbstractXmlApplicationContext`类提供了`loadBeanDefinitions`方法的具体实现；

  

```java
@Override
protected final void refreshBeanFactory() throws BeansException {
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    try {
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        customizeBeanFactory(beanFactory);
        loadBeanDefinitions(beanFactory);
        this.beanFactory = beanFactory;
    }
    catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
    }
}
```

3）在`AbstractXmlApplicationContext`类的`loadBeanDefinitions`方法中，会依次调用多个类的`loadBeanDefinitions`方法，最终会调用到`XmlBeanDefinitionReader`类的`doLoadBeanDefinitions`方法；

4）在`XmlBeanDefinitionReader`类的`doLoadBeanDefinitions`方法中，截取了部分代码如下

- 以下代码的第3行，调用`doLoadDocument`方法来读取XML信息，并保存到`Document`对象中；

- 以下代码的第4行，调用`registerBeanDefinitions`方法来解析`Document`对象，封装`BeanDefinition`对象并进行注册；

  

```java
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource) throws BeanDefinitionStoreException {
    try {
        Document doc = doLoadDocument(inputSource, resource);
        int count = registerBeanDefinitions(doc, resource);
        if (logger.isDebugEnabled()) {
            logger.debug("Loaded " + count + " bean definitions from " + resource);
        }
        return count;
    }
    ...
}
```

5）在`XmlBeanDefinitionReader`类的`registerBeanDefinitions`方法中，完整代码如下

- 以下代码的第4行的方法入参中，依次调用`createReaderContext`方法 --> `getNamespaceHandlerResolver`方法 --> `createDefaultNamespaceHandlerResolver`方法 --> `DefaultNamespaceHandlerResolver`类的构造方法；

- 以下代码的第4行，调用`BeanDefinitionDocumentReader`接口的`registerBeanDefinitions`方法；

- `DefaultBeanDefinitionDocumentReader`类提供了`registerBeanDefinitions`方法的具体实现；

  

```java
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
    BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
    int countBefore = getRegistry().getBeanDefinitionCount();
    documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
    return getRegistry().getBeanDefinitionCount() - countBefore;
}
```

6）在`DefaultBeanDefinitionDocumentReader`类的`registerBeanDefinitions`方法中，会依次调用`doRegisterBeanDefinitions`方法 --> `parseBeanDefinitions`方法 --> `parseDefaultElement`方法 --> `processBeanDefinition`方法；

7）在`DefaultBeanDefinitionDocumentReader`类的`processBeanDefinition`方法中，完整代码如下

- 以下代码第2行，调用`parseBeanDefinitionElement`方法解析得到`BeanDefinitionHolder`对象；

- 以下代码第7行，依次调用`BeanDefinitionReaderUtils`类的`registerBeanDefinition`方法 -->`BeanDefinitionRegistry`接口的`registerBeanDefinition`方法；

-  `DefaultListableBeanFactory`类提供了`registerBeanDefinition`方法的具体实现；

  

```java
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // Register the final decorated instance.
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +	bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```

8）在`DefaultListableBeanFactory`类中

- 定义了如下所示的`beanDefinitionMap`和`beanDefinitionNames`域；

  ```java
  /** Map of bean definition objects, keyed by bean name. */
  private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
  
  /** List of bean definition names, in registration order. */
  private volatile List<String> beanDefinitionNames = new ArrayList<>(256);
  ```

  

- 在`registerBeanDefinition`方法中，会把封装的 XML 中定义的 Bean信息封装得到的`BeanDefinition`类型的`beanDefinition`放入一个`beanDefinitionMap`，将`String`类型的`beanName`放入`beanDefinitionNames`中，从而完成注册。

```java
// Still in startup registration phase
this.beanDefinitionMap.put(beanName, beanDefinition);
this.beanDefinitionNames.add(beanName);
```



### 2.4.3 Bean的生命周期

在 IoC 容器的初始化过程中会对非懒加载的Bean定义完成资源定位，加载读取配置，并解析转化成Spring能够识别的`BeanDefinition`对象，最后将解析得到的`BeanDefinition`存到HashMap 集合中（对于懒加载的Bean来说，其创建是在第一次调用`getBean`方法时完成的）。

<img src="生命周期.jpg" alt="生命周期" style="zoom:100%;" />

上图步骤1-12的关键调用点在`AbstractApplicationContext`类的`refresh`方法中

- 调用`invokeBeanFactoryPostProcessors`方法，会实例化`BeanFactoryPostProcessor`实现类并执行其`postProcessBeanFactory`方法，对应上图步骤1-2；

- 调用`registerBeanPostProcessors`方法，会实例化`BeanPostProcessor`实现类，对应上图步骤3；

- 调用`finishBeanFactoryInitialization`方法，会通过反射执行Bean的构造方法（创建所有非懒加载的单例Bean实例）、注入属性、Bean Class加载以及初始化，对应上图步骤4-12，更多细节可查看2.4.5小节。

  

### 2.4.4 后置处理器

Spring提供了两种后置处理器`BeanFactoryPostProcessor`和`BeanPostProcessor`。从命名中可以看出区别：`BeanFactoryPostProcessor`是针对IoC容器（`BeanFactory`）初始化之后的后置处理器，`BeanPostProcessor`是针对Bean初始化之后的后置处理器。

1）`BeanFactoryPostProcessor`接口只提供了一个方法，该方法参数为`ConfigurableListableBeanFactory`；

```java
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

`ConfigurableListableBeanFactory`接口定义了一些方法，其中有个`getBeanDefinition`方法可以返回定义Bean的`BeanDefinition`对象，从而可以对定义的属性进行手动修改。

```java
public interface ConfigurableListableBeanFactory extends ListableBeanFactory, AutowireCapableBeanFactory, ConfigurableBeanFactory {
    BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
}
```

2）定义一个类实现了`BeanPostProcessor`接口，默认是会对容器中所有的Bean进行处理，也可以通过方法参数来判断要处理的具体的某个Bean。

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
    
    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

### 2.4.5 创建Bean

接2.4.3小节：

1）在`AbstractApplicationContext`类的`finishBeanFactoryInitialization`方法中，会调用`ConfigurableListableBeanFactory`接口的`preInstantiateSingletons`方法，`DefaultListableBeanFactory`类提供了`preInstantiateSingletons`方法的具体实现；

2）在`DefaultListableBeanFactory`类的`preInstantiateSingletons`方法中，截取了部分代码如下

- 以下代码的第12行的`for`循环会触发所有非懒加载的单例Bean的初始化；

- 以下代码的第14行会判断是否是非懒加载的单例Bean，是的话则在IoC容器创建时创建Bean实例、依赖注入以及初始化；

- 以下代码的第32行会依次调用`AbstractBeanFactory`类的`getBean`方法 --> `doGetBean`方法；

- 以下代码的第32行所触发的逻辑和懒加载时第一次调用`context.getBean("beanName")` 所触发的逻辑是一致的；

  

```java
@Override
public void preInstantiateSingletons() throws BeansException {
    if (logger.isTraceEnabled()) {
        logger.trace("Pre-instantiating singletons in " + this);
    }
    
    // Iterate over a copy to allow for init methods which in turn register new bean definitions.
    // While this may not be part of the regular factory bootstrap, it does otherwise work fine.
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    
    // Trigger initialization of all non-lazy singleton beans...
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                if (bean instanceof FactoryBean) {
                    FactoryBean<?> factory = (FactoryBean<?>) bean;
                    boolean isEagerInit;
                    if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                        isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit, getAccessControlContext());
                    }
                    else {
                        isEagerInit = (factory instanceof martFactoryBean && ((SmartFactoryBean<?>) factory).isEagerInit());
                    }
                    if (isEagerInit) {
                        getBean(beanName);
                    }
                }
            }
            else {
                getBean(beanName);
            }
        }
    }
    
    ...
}
```

3）在`AbstractBeanFactory`类的`doGetBean`方法中，截取了部分代码如下

- 以下代码的第14行，通过`isPrototypeCurrentlyInCreation`方法来判断是否要获取一个正在被创建的原型Bean，是的话则直接抛出异常（原型Bean在创建之前会标记这个Bean正在被创建，等创建结束之后会删除标记），**这代表Spring不支持原型 Bean的循环依赖，会直接抛出异常**；

- 以下代码的第31行，会依次调用`AbstractAutowireCapableBeanFactory`类的`createBean`方法 --> `doCreateBean`方法；

  

```java
protected <T> T doGetBean(String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)	throws BeansException {
    String beanName = transformedBeanName(name);
    Object beanInstance;
    
    // Eagerly check singleton cache for manually registered singletons.
    Object sharedInstance = getSingleton(beanName);
    if (sharedInstance != null && args == null) {
        ...
    }
    
    else {
        // Fail if we're already creating this bean instance:
        // We're assumably within a circular reference.
        if (isPrototypeCurrentlyInCreation(beanName)) {
            throw new BeanCurrentlyInCreationException(beanName);
        }
        
        // Check if bean definition exists in this factory.
        ...
        
        try {
            ...
            
            // Guarantee initialization of beans that the current bean depends on.
            ...
            
            // Create bean instance.
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                    ...
                });
                ...
            }
            ...
        }
        ...
    }
    ...
}
```

4）在`AbstractAutowireCapableBeanFactory`类的`doCreateBean`方法中，截取了部分代码如下

- 在以下代码的第9行，调用`createBeanInstance`方法，这里会通过反射调用构造器，创建了Bean对象（此时还未注入属性）；

- 在以下代码的第26行，调用`addSingletonFactory`方法，该方法的第二个入参是一个`ObjectFactory`类型，这里会**将还未注入属性的Bean对象提前以`ObjectFactory`对象的形式暴露到IoC容器中，从而解决单例Bean的属性循环依赖问题（但无法解决构造器循环依赖问题，会直接抛出异常）**；

- 在以下代码的第34行，调用`populateBean`方法，这里会对Bean注入属性；

- 在以下代码的第35行，调用`initializeBean`方法，这里会进行Bean Class加载以及Bean的初始化，更多细节可查看3.5.2小节。

  

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    
    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    Object bean = instanceWrapper.getWrappedInstance();
    Class<?> beanType = instanceWrapper.getWrappedClass();
    if (beanType != NullBean.class) {
        mbd.resolvedTargetType = beanType;
    }
    
    ...
    
    // Eagerly cache singletons to be able to resolve circular references
    // even when triggered by lifecycle interfaces like BeanFactoryAware.
    boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        if (logger.isTraceEnabled()) {
            logger.trace("Eagerly caching bean '" + beanName + "' to allow for resolving potential circular references");
        }
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
    
    ...
    
    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    ...
}
```

# 3、AOP

## 3.1 问题与解决

假设业务的组件中有多个业务方法，每个业务方法除了要执行核心业务逻辑，还需要执行安全检查、事务管理等处理逻辑，这些处理逻辑会重复出现在每个四处分散的业务方法中。

针对该问题可以使用Proxy模式。对每个业务方法实现Proxy，将这些处理逻辑放置在Proxy当中。这种手动编写Proxy的方式不够简单，更简单的方法是将每一种处理看作一个切面，然后以某种自动化的方式将切面逻辑织入到核心逻辑中，从而实现Proxy模式。

依赖注入有助于应用对象之间的解耦，而AOP把系统分解为不同的切面（比如安全检查、事务管理等处理），可以实现横切关注点与它们所影响的对象之间的解耦。

AOP是OOP的延续，OOP是一种垂直纵向的继承体系，AOP是一种横向抽取机制，将横切逻辑代码和业务逻辑代码分开，Java中的AOP机制有：Java代理、Spring框架、AspectJ。

## 3.2 术语介绍

- **连接点（Joinpoint）**

  - 连接点是应用执行过程中**能够**插入切面的所有点，是一种候选点；
  - 切面代码利用这些点插入到应用的正常流程中，从而添加新的行为；
  - 连接点模型有：字段连接点、构造器连接点、方法连接点；
  - Spring只支持方法连接点，如果需要方法拦截之外的连接点拦截，可以转向更为强大的Aspect。

- **通知（Advice）** 

  - 通知定义了切面是**什么**以及**何时**使用；

  - 通知有以下五种类型；

    | 通知            | 描述                                                         |
    | :-------------- | :----------------------------------------------------------- |
    | Before          | 在方法调用之前调用通知                                       |
    | After           | 在方法完成之后调用通知，无论方法执行是否成功                 |
    | After-returning | 在方法成功执行之后调用通知                                   |
    | After-throwing  | 在方法抛出异常后调用通知                                     |
    | Around          | 通知包裹被通知的方法，在被通知方法的调用前和后执行自定义行为 |

  - Spring所创建的通知都是由标准的Java类编写的。

- **切点（Pointcut）**

  - 切点定义了切面在**何处**使用，即哪些连接点会得到通知；
  - Spring所定义的切点通常由XML文件或者注解来进行配置。

- **切面（Aspect）**

  - 切面是通知和切点的结合，切点和通知是切面的最基本元素；
  - 通知和切点共同定义了关于切面的全部内容（什么、何时、何处）。

- **引入（Introduction）**

  - 引入允许在不修改现有类的情况下向现有类添加新方法或属性，令其具有新的行为和状态。

- **织入（Weaving）**

  - 织入是将切面应用到目标对象来创建新的代理对象的过程，切面在指定的连接点被织入到目标对象中；

  - 在目标对象的生命周期里有以下三个时期可以进行织入；

    | 时期     | 描述                                           |
    | :------- | :--------------------------------------------- |
    | 编译期   | 切面在目标类编译时被织入，比如AspectJ          |
    | 类加载期 | 切面在目标类加载到JVM时被织入                  |
    | 运行期   | 切面在应用运行的某个时刻被织入，比如Spring AOP |

  - Spring的AOP是基于JVM的动态代理实现的，在运行期将切面织入到Spring管理的Bean中， AspectJ AOP是基于字节码操作实现的，在编译期织入切面；

  - Spring AOP更加简单，AspectJ AOP更加强大，如果切面比较少，那么两者性能差异不大，如果切面比较多，AspectJ AOP比 Spring AOP 快很多。

## 3.3 XML配置

### 3.3.1 使用步骤

1）通过Maven创建工程并引入`spring-aspects`依赖；

2）编写定义了切面逻辑的简单Java类；

3）在XML文件中将该类注册为Spring应用上下文中的一个Bean；

4）在XML文件中将该类配置为一个切面；

```xml
<aop:config>
    <aop:aspect id = "..." ref = "...">
        <aop:before pointcut = "..." method = "..." arg-names = "..." />
        <aop:after pointcut = "..." method = "..." arg-names = "..." />
    </aop:aspect>                
</aop:config>
```

### 3.3.2 配置元素

- 在`<aop:config>`元素内可以声明一个或多个通知器、切面或者切点；

- 在`<aop:aspect>`元素内可以声明一个简单的切面，`ref`属性引用了一个Bean，该Bean提供了在切面上通知所调用的方法；

- `<aop:before>`等元素分别定义不同的通知类型，`pointcut`属性定义了通知所应用的切点，`method`属性定义了切面上所调用的方法名，`arg-names`属性定义了所调用方法的入参；

- 如果有多个通知的`pointcut`属性值是一致的，那么可以通过以下配置方式来避免重复定义切点，其中，`<aop-pointcut`元素中的`expression`属性定义了通知所应用的切点；

  ```xml
  <aop:config>
      <aop:aspect ref = "...">
          <aop:pointcut id = "xxx" expression = "..." />
          <aop:before pointcut-ref = "xxx" method = "..." arg-names = "..." />
          <aop:after pointcut-ref = "xxx" method = "..." arg-names = "..." />
      </aop:aspect>                
  </aop:config>
  ```

- `<aop:before>`等元素的`pointcut`属性以及`<aop-pointcut`元素的`expression`属性的值是一个AspectJ切点表达式。

### 3.3.3 切点表达式

切点表达式指的是遵循特定语法结构的字符串，其作用是对符合语法格式的连接点进行通知。

在Spring AOP中，需要使用AspectJ的切点表达式来定义通知所引用的切点，比如表达式`execution(*A.B(..))`代表：

- 使用`execution()`指示器来唯一选择`A`类（全限定类名）的`B`方法；

- 方法表达式以`*`开始，标识不关心方法的返回值类型；

- 方法参数列表为`(..)`，标识不关心方法的入参类型。

  

需要注意的是：

- Spring仅支持AspectJ切点指示器的一个子集，`execution()`指示器就属于该子集；

- 当在Spring中尝试使用AspectJ子集之外的其他指示器时，会抛出异常；

- `bean()`指示器使用Bean的ID作为参数来限制切点只匹配特定的Bean，该指示器是由Spring 2.5新引入的。

  

Spring支持的AspectJ切点指示器如下

- `arg()`：限制连接点匹配参数为指定类型的执行方法

- `@args()`：限制连接点匹配参数由指定注解标注的执行方法

- `execution()`：用于匹配是连接点的执行方法

- `this()`：限制连接点匹配AOP代理的Bean引用为指定类型的类

- `target()`：限制连接点匹配目标对象为指定类型的类

- `@target()`：限制连接点匹配特定的执行对象，这些对象对应的类要具备指定类型的注解

- `within()`：限制连接点匹配指定的类型

- `@within()`：限制连接点匹配指定注解所标注的类型（当使用Spring AOP时，方法定义在由指定注解所标注的类里）

- `@annotation`：限制匹配带有指定注解连接点

  

### 3.3.4 引入新功能

切面除了能够为现有的方法增加额外的功能，还能为现有的Spring Bean增加新的方法。

```xml
<aop:config>
    <aop:aspect>
        <aop:declare-parents 
           	types-matching = "" implements-interface = "" default-impl = "" />
    </aop:aspect>
</aop:config>
```

- `<aop:declare-parents>`声明了此切面所通知的Bean在它的对象层次结构会中拥有新的父类型;

- 在这些Bean中，由`types-matching`属性指定的类型会实现由`implements-interface`属性指定的接口；

- 具体的实现由`default-impl`属性来直接标识或者由`delegate-ref`属性来间接委托。

  

## 3.4 注解配置

在上一节的XML配置中，定义了切面逻辑的简单Java类中并没有任何地方让它成为一个切面，因此，不得不使用XML声明通知和切点。但是通过`@Aspect`注解，可以在不需要额外XML配置的情况下将简单Java类转换为一个切面。

### 3.4.1 使用步骤

1）通过Maven创建工程并引入`spring-aspects`依赖；

2）使用`@Aspect`标注编写定义了切面逻辑的简单Java类（下文简称为切面Bean）；

3）使用`@Pointcut`定义一个可以在`@AspectJ`切面内可重用的切点；

- `@Pointcut`注解的值是一个AspectJ切点表达式；

- `@Pointcut`注解所标注的是一个空的标识方法，供`@Pointcut`注解依附；

- 标识方法的方法签名相当于切点的名字，供通知拦截器引用；

  

4）使用`@Before`等通知拦截器注解在切面Bean中编写拦截器代码（即切面逻辑）；

5）给`@Configuration`标注的配置类加上`@EnableAspectJAutoProxy`注解，使得IoC容器自动查找`@Aspect`所标注的Bean，并根据每个方法的通知拦截器将AOP织入到特定的Bean中。

### 3.4.2 通知拦截器

- `@Before`：先执行拦截器代码，再执行目标代码，如果拦截器抛出异常，目标代码不执行；

- `@After`：先执行目标代码，再执行拦截器代码，无论目标代码是否抛出异常，拦截器代码都会执行；

- `@AfterReturning`：和`@After`的区别在于，只有目标代码正常返回时才会执行拦截器代码；

- `@AfterThrowing`：和`@After`的区别在于，只有目标代码抛出异常时才会执行拦截器代码；

- `@Around`：能在目标代码执行前后均执行拦截器代码，即便目标代码抛出异常，依旧继续执行拦截器代码，能够实现以上全部拦截器的功能。

  

### 3.4.3 精准指定切点

除了使用AspectJ切点表达式，还可以考虑通过注解来标注需要拦截的方法，从而精准指定需要在何处织入切面。

- 使用`@Transactional`注解去标注需要拦截的业务方法，即可精准指定需要在何处织入事务处理切面，实现声明式事务

  - 事务往往在service层进行控制；

  - `@Transactional`注解只有在使用基于接口的代理时才会生效；

  - 如果异常被catch了，则事务就不回滚了，如果想让事务回滚，则必须在catch块中进行相关处理之后再往外throw异常；

    

- 除了`@Transactional`注解，还可以自定义注解来标注需要拦截的方法，此时切面Bean中的切点表达式只需写为AspectJ指示器（自定义注解）这样的形式，即可在业务代码中通过自定义注解来精准指定需要在何处织入切面。

  


### 3.4.4 引入新功能

在上文介绍了如何使用基于XML的AOP来为现有的Spring Bean增加新的方法，这一功能依旧可以使用基于注解的AOP来完成。

- `@DeclareParents`等价于`aop:declare-parents>`；

- `@DeclareParents`的`value`属性等同于`aop:declare-parents>`的`types-matching`属性，标识应该被引入指定接口的Bean的类型；

- `@DeclareParents`的`defaultImpl`属性等同于`aop:declare-parents>`的`default-impl`属性，标识所引入接口的实现；

- 被`@DeclareParents`所标注的`static`属性指定了将被引入的接口；

- `@DeclareParents`没有与`aop:declare-parents>`的`delegate-ref`属性等价的属性。

  

## 3.5 AOP特性

### 3.5.1 CGLIB

Spring的AOP是基于JVM的动态代理实现的，在代理对象中织入切面逻辑。如果要代理的对象实现了某个接口，那么 Spring AOP 会使用 **JDK Proxy**去创建代理对象，而对于没有实现接口的对象，Spring AOP 会使用 **CGLIB**生成一个被代理对象的子类来作为代理。需要注意的地方在于：

- Spring通过**CGLIB创建的代理类不会初始化继承自原始类的任何成员变量**，包括`final`类型的成员变量，也就是说，CGLIB所创建的代理所继承的成员变量的值始终为`null`，因此加AOP很可能会导致NPE；

- 以上NPE问题的解决方式是业务类通过`getXXX`方法而不是直接获取`XXX`变量的值，那么CGLIB创建的代理类会覆写`getXXX`方法，从而返回原始业务类的`XXX`变量的值；

- Spring通过**CGLIB创建的代理类无法覆写继承自原始类的final方法**，即无法代理`final`方法。

  

因此，为了正确使用AOP：

- 在访问被注入的Bean的成员变量时，总是调用`getXXX`方法而非直接访问字段；

- 编写可能会被代理的Bean时，避免编写`public final`方法。

  

### 3.5.2 创建代理Bean

接2.4.5小节：

1）在`AbstractAutowireCapableBeanFactory`类的`initializeBean`方法中，会依次调用`applyBeanPostProcessorsAfterInitialization`方法 --> `BeanPostProcessor`接口的`postProcessAfterInitialization`方法，`AbstractAutoProxyCreator`类提供了`postProcessAfterInitialization`方法的具体实现；

2）在`AbstractAutoProxyCreator`类的`postProcessAfterInitialization`方法中，完整代码如下

- 以下代码的第5行，检查当前Bean对象是否已经暴露过了，只有没有暴露过的对象，才生成其代理对象；

- 以下代码的第6行，会依次调用`wrapIfNecessary`方法 --> `createProxy`方法，从而创建代理对象；

  

```java
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}
```

3）在`AbstractAutoProxyCreator`类的`createProxy`方法中，截取了部分代码如下

- 以下代码的第7行，调用`ProxyFactory`类的构造方法生成代理工厂，后续会将创建代理的工作交给`proxyFactory`；

- 以下代码的第17行，调用`ProxyFactory`类的`getProxy`方法来创建代理；

  

```java
protected Object createProxy(Class<?> beanClass, @Nullable String beanName, @Nullable Object[] specificInterceptors, TargetSource targetSource) {
    
    if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
        AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory) this.beanFactory, beanName, beanClass);
    }
    
    ProxyFactory proxyFactory = new ProxyFactory();
    proxyFactory.copyFrom(this);
    
    ...
    
    // Use original ClassLoader if bean class not locally loaded in overriding class loader
    ClassLoader classLoader = getProxyClassLoader();
    if (classLoader instanceof SmartClassLoader && classLoader != beanClass.getClassLoader()) {
        classLoader = ((SmartClassLoader) classLoader).getOriginalClassLoader();
    }
    return proxyFactory.getProxy(classLoader);
}
```

4）在`ProxyFactory`类的`getProxy`方法中，完整代码如下

- 首先会依次调用`ProxyCreatorSupport`类的`createAopProxy`方法 --> `getAopProxyFactory`方法 --> `DefaultAopProxyFactory`类的`createAopProxy`方法，得到`AopProxy`类对象；

- 接着调用`AopProxy`类对象的`getProxy`方法得到代理类对象；

  

```java
public Object getProxy(@Nullable ClassLoader classLoader) {
    return createAopProxy().getProxy(classLoader);
}
```



5）在 `DefaultAopProxyFactory`类的`createAopProxy`方法中，会判断创建代理对象时是使用JDK 代理，还是使用CGLIB，完整代码如下

- 设置`proxyTargetClass=true`会强制使用CGLIB；

- 没有设置参数并且对象类实现了接口则默认使用JDK 代理；

- 没有设置参数并且对象类没有实现接口则也必须使用CGLIB；

- 以`ObjenesisCglibAopProxy`类为例，它继承了父类`CglibAopProxy`的`getProxy`方法；

- 在`CglibAopProxy`类的`getProxy`方法中，最终会通过调用`createProxyClassAndInstance`方法来创建代理类，并生成一个代理类的实例。

  

```java
@Override
public AopProxy createAopProxy(AdvisedSupport config) throws AopConfigException {
    if (!NativeDetector.inNativeImage() && (config.isOptimize() || config.isProxyTargetClass() || hasNoUserSuppliedProxyInterfaces(config))) {
        Class<?> targetClass = config.getTargetClass();
        if (targetClass == null) {
            throw new AopConfigException("TargetSource cannot determine target class: " + "Either an interface or a target is required for proxy creation.");
        }
        if (targetClass.isInterface() || Proxy.isProxyClass(targetClass) || ClassUtils.isLambdaClass(targetClass)) {
            return new JdkDynamicAopProxy(config);
        }
        return new ObjenesisCglibAopProxy(config);
    }
    else {
        return new JdkDynamicAopProxy(config);
    }
}
```



# 4、MVC

## 4.1 问题与解决

由于HTTP协议的无状态性，基于Web的应用程序在状态管理、工作流等问题上面临着重大挑战，Spring MVC就是为了解决这些问题而设计的。

Spring MVC基于模型-视图-控制器（Model-View-Controller, MVC）模式实现，核心思想在于将业务逻辑、数据、显示分离，有助于构建灵活、松耦合的Web应用程序。

整个MVC相当于三层架构中的UI表现层，通过MVC对UI层又进一步进行了分层。

- Model可以是一个JavaBean，也可以是一个包含多个JavaBean的Map；
- View只负责把Model给“渲染”出来；
- Controller用于处理View中的响应，调用业务处理方法并将得到的Model返回给View，Controller中尽量不放业务逻辑代码。

三者职责明确，且开发更简单，因为开发Controller时无需关注View，开发View时无需关心如何创建Model。

## 4.2 请求的生命周期

每当用户在Web浏览器中点击链接或者提交表单的时候，一个Request就开始工作了：

1）Request首先会到达`DispatcherServlet`（前端控制器Servlet，继承自`HttpServlet`）；

- `DispatcherServlet`会查询一个或者多个`Handler mapping`（处理器映射）；

- `Handler mapping`根据Request所携带的URL信息来进行决策；

- `DispatcherServlet`根据查询结果来确定应该将Request委托给哪一个`Controller`（控制器）；

  

2）Request到达`Controller`后会卸下其负载（用户提交的信息）并等待`Controller`完成逻辑处理；

- 设计良好的`Controller`本身几乎不处理工作，而是将业务逻辑委托给一个或多个服务对象；

- 在逻辑处理完成后，会产生一些信息，这些信息被称为**Model（模型）**；

- Model需要被发送给一个**View（视图）**，即以用户友好的方式进行格式化，一般是HTML；

- `Controller`将Model数据打包，标示出用于渲染输出的View名称；

- `Controller`不会与特定的View相耦合；

  

3）Request、Model和View名称被`Controller`发送回`DispatcherServlet`；

- 传递给`DispatcherServlet`的View名称仅仅是一个逻辑名，并不直接表示某个特定的View；

- `DispatcherServlet`使用ViewResolve（视图解析器）来为View的逻辑名称添加前后缀，然后查找一个特定的View实现模板；

  

4）Request的最后一站是View的实现模板，在这里Request交付Model数据，Request的任务就完成了；

- View模板通常是JSP（JavaServer Page），但其他的View技术也可以使用；
- View模板通过Model数据会渲染输出一个View，通过这个View将Response传递给客户端；

至此，就完成了从Request离开浏览器到获取Response返回浏览器的过程。

## 4.3 搭建步骤

1）在web.xml文件中配置`DispatcherServlet`；

```xml
<servlet>
    <servlet-name>spitter</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>spitter</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

- 由于`servlet-name`元素被配置为`spitter`，`DispatcherServlet`在加载时会从一个名为`spitter-servlet.xml`的文件来加载应用上下文；

- `servlet-mapping`元素的`url-pattern`属性将`DispatcherServlet`映射到`/`，声明了它会作为默认的servlet并且会处理所有的请求，包括对静态资源的请求；

  

2）编写`spitter-servlet.xml`文件并配置注解驱动；

```xml
<mvc:resources mapping = "/resources/**" location = "/resources/" />
<mvc:annotation-driven/>
<context:component-scan base-package = "..." />
```

- `<mvc:resources>`元素建立了一个服务于静态资源的处理器，此时，所有的静态资源都必须放置在应用程序的/resources目录下；

- `<mvc:annotation-driven/>`标签与`DispatcherServlet`提供的处理器映射（`DefaultAnnotationHandlerMapping`）相结合就能实现基于注解的控制器类；

- Spring也提供了其他类型的处理器映射，只需在Spring中配置该处理器映射Bean即可使用；

- `<context:component-scan>`元素将查找所有的控制器类并将其注册为Bean；

  

3）编写首页控制器以及面向Spitter和Spittle的控制器，应当为应用程序所提供的每一种资源（而不是为每个用例）编写一个单独的控制器；

4）在`spitter-servlet.xml`文件中配置视图解析器

```xml
<bean class = "org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name = "viewClass" 
              value = "org.springframework.web.servlet.view.JstlView" />
    <property name = "prefix" value = "/WEB-INF/views/" />
    <property name = "suffix" value = ".jsp" />
</bean>
```

- 视图解析器`InternalResourceViewResolver`将View的逻辑名称与JSP（JavaServer Page）相匹配；

- `InternalResourceViewResolver`将View的逻辑名称添加前后缀得到View路径，并解析为View对象；

- View对象将渲染的任务委托给视图模板，即将Request传递给JSP；

- 如果需要使用其他的视图解析器，同样需要在`spitter-servlet.xml`文件中将该视图解析器注册为一个`<bean>`；

  

5）定义JSP视图并完成Spring应用上下文；

- Web层的配置都放在`spitter-servlet.xml`的文件中，这个文件会被`DispatcherServlet`加载，但还需要加载其他的配置文件；

- 通过使用`ContextLoaderListener`，能够加载其他的配置文件到一个Spring应用上下文中。

  

## 4.4 REST功能

以信息为中心的表属性状态转移（Representational State Transfer, REST）已成为替换传统SOAP Web服务的流行方案，REST就是将资源的状态以最合适的形式从服务器端转移到客户端（或者反之），RESTful交互是同步的。

- 表述性（Representational）：REST资源可以用各种形式来进行表述，比如XML、JSON以及HTML；
- 状态（State）：使用REST时，我们更关注资源的状态而不是对资源采取的行为，所有的资源都可以通过URI 定位，而且这个定位与其他资源无关，也不会因为其他资源的变化而变化；
- 转移（Transfer）：REST涉及转移资源数据，它以某一种表述形式从一个应用转移到另一个应用

Spring对REST的支持是构建在Spring MVC之上的，控制器可以处理所有的HTTP方法，包含四个主要的REST方法：GET、PUT、DELETE以及POST，分别对应对资源的检索、更新、删除和创建。通过适当设置`@RequestMapping`注解的`method`属性，就可以让`DispatcherServlet`把不同的HTTP方法的请求定向到特定的控制器方法上。

RESTful Web 服务与传统的 MVC 开发一个关键区别是返回给客户端的内容的创建方式：**传统的 MVC 模式开发会直接返回给客户端一个视图，但是 RESTful Web 服务一般会将返回的数据以 JSON 的形式返回，实现前后端分离开发。**

可处理的HTTP方法具体介绍如下：

- GET

  - 可只直接使用`@GetMapping`注解；

  - 是最常用的方法，可用于获取资源；

  - 在浏览器中可以回退，请求会被浏览器主动缓存；

  - 只能进行URL编码，只接收ASCII字符；

  - 参数直接暴露在URL上，安全性较低，不能用于传递敏感信息。

    

- POST

  - 可只直接使用`@PostMapping`注解；

  - 如果需要添加对象，可使用POST方法传递一个Model 对象；

  - 访问同一地址时也需要再次提交请求，浏览器不会缓存请求；

  - 除了URL，还支持多种编码方式；

  - 参数可以放在request body 中，也可以通过URL传递。

    

- PUT

  - 可只直接使用`@PutMapping`注解；

  - 如果对象需要更新，可使用PUT方法去发送请求。

    

- PATCH

  - 是一个新引入的方法，是对PUT方法的补充；

  - 用来对己知资源进行局部更新，比如一个对象的部分数据而非整个对象数据。

    

- DELETE

  - 可只直接使用`@DeleteMapping`注解；

  - 在执行前先查询是否有数据；

  - 由于返回的是`void` 类型，要注意判断是否成功，可通过存储过程返回值来判断。

    

- OPTIONS

  - 用于获取当前URL所支持的方法；

  - 返回的响应消患会在HTTP 头中包含“Allow”的信息，其值是所支持的方法， 如GET。

    

- HEAD

  - 代表发送HTTP 头消息，GET其实也带了HTTP 头消息。

    

- TRACE

  - 用于显示服务器收到的请求，从而进行测试或诊断。

## 4.5 相关组件

### 4.5.1 控制器

- `@Controller`

  - 用于类，表明该类是MVC的控制器。

    

- `@RequestBody`

  - 用于请求方法参数；

  - 可以将请求中的(JSON/XML）字符串绑走到相应的Bean 上，也可以将其分别绑定到对应的字符串上。

    

- `@ResponseBody`

  - 用于方法；

  - 将请求方法返回的对象转换为指定格式（JSON/XML）后，再写入`Response`对象的`body`数据区。

    

- `@RestController`

  - 是Spring 4.0之后才有的注解；

  - 是`@Controller`和``@ResponseBody``的结合；

  - 用于标注REST风格的控制器；

  - 告诉Spring将结果字符串直接呈现给调用者。

    

- `@RequestMapping`

  - 提供了“路由”信息，告诉Spring任何带有/路径的HTTP请求都应该映射到home方法；

  - 用于类时，定义了这个控制器所处理的根URL路径；

  - 用于方法时，表明该方法是一个请求处理方法，要处理指定URL下的请求；

  - 请求处理方法的签名可以将任何事物作为入参；

  - `value`属性用于指定请求的地址；

  - `method`属性用于指定请求的类型（GET、HEAD、POST、PUT、PATCH、DELETE、OPTIONS、TRACE）；

  - `consumes`属性用于指定请求的内容类型；

  - `produces`属性用于指定返回的内容类型；

  - `params`属性用于指定请求中必须包含的参数值；

  - `headers`属性用于指定请求中必须包含的header值。

    

- `@RequestParam`

  - 用于请求方法参数，当查询参数与方法参数名不匹配时，可以通过该注解将两者绑定。

    

- `@PathVariable`

  - 用于请求方法参数，将请求中的变量映射到请求处理方法的参数上。


### 4.5.2 Servlet

Servlet 是`javax.servlet`包中定义的一个接口，除了Controller以外，有时也需要使用Servlet来实现拦截和监昕功能。

- `@WebServlet(urlPattern = "")`

  - 用于注册Servlet实现类；

  - 使得`DispatcherServlet`核心控制器知道它的作用，以及处理请求`urlpattern`。

    

- `＠ServletComponentScan`

  - 用于启动类；

  - 以扫描注册指定包内的所有Servlet、Filter以及监听器。

    

- `HttpServlet`

  - Servlet抽象类；

  - 自定义的Servlet需要继承`HttpServlet`，并重写`doGet`方法。

    

### 4.5.3 过滤器

对于多个页面都需要进行的例如参数校验等功能，可以通过过滤器来实现，从而避免重复代码。定义一个过滤器应当：

- 实现`Filter`接口；

- 实现`init`、`doFilter`、`destory`方法；

- 使用`@WebFilter(urlPatterns = "")`注解标注；

- 使用`＠ServletComponentScan`注解标注启动类。

  

### 4.5.4 监听器

监听器用于监听Web应用程序中某些对象或信息的创建、销毁、增加、修改、删除等动作，然后做出相应的响应处理。当对象的状态发生变化时，服务器自动调用监听器的方法。具体应用有统计在线用户人数、在结用户、系统加载时的信息初始化等。


Servlet 中的监听器分为以下类型：

- 监听`ServletContext`、`Request`、`Session`作用域的创建和销毁；

  - `ServletContextListener`；

  - `HttpSessionListener`；

  - `ServletRequestListener`；

    

- 监听`ServletContext`、`Request`、`Session`作用域中属性的变化（增加、修改、删除）；

  - `ServletContextAttributeListener`；

  - `HttpSessionAttributeListener`；

  - `ServletRequestAttributeListener`；

    

- 监听`HttpSession`作用域中对象状态的改变（被绑定、解除绑走、钝化、活化）；

  - `HttpSessionBindingListener`；

  - `HttpSessionActivationListener`；

    

定义一个监听器应当：

- 实现上述任意类型的监听器接口；

- 实现监听器接口中对应的方法；

- 使用`@WebListener`注解标注；

- 使用`＠ServletComponentScan`注解标注启动类。

  

### 4.5.5 异常

Spring提供的异常处理方案是利用切面技术实现的，会将异常处理切面织入到目标控制器中。

- 异常类一般需要包含以下信息：

  - 唯一标示异常的 code；
  - HTTP状态码；
  - 错误路径；
  - 发生错误的时间戳；
  - 错误的具体信息。

- `RuntimeException`

  - 一般处理的异常都是`RuntimeException`；

  - 可以通过继承`RuntimeException`或新建`ResponseStatusException`（通过构造器来注入属性值）来定义自己的异常类型。

    

- `@ControllerAdvice`或`@RestControllerAdvice`

  - 可以用于定义异常处理类，默认对所有的`Controller`都有效；

  - `@ControllerAdvice(annotations = {RestController.class})`可以指定特定的注解，使得异常处理类只处理特定注解所标记的类抛出的异常；

  - `@ControllerAdvice("")`可以指定特定的包名，使得异常处理类只处理特定包内抛出的异常；

  - `@ControllerAdvice(assignableTypes = {ExceptionController.class})`可以指定特定的`Controller`，使得异常处理类只处理特定`Controller`抛出的异常。

    

- `@ExceptionHandler`

  - 可以用于定义异常处理方法，可以拦截控制器中发生的异常；

  - `@ExceptionHandler(value = XXXException.class)`可以指定特定的异常类型，使得异常处理方法只处理特定的异常类型。

    

# 5、ORM

## 5.1 问题与解决

业务实体在内存中表现为对象，在数据库中表现为关系型数据。内存中的对象不会被永久保存，只有关系型数据库（或NoSQL 数据库，或文件）中的对象会被永久保存。

ORM（Object Relation Mapping）指对象关系映射，对数据库中的表和内存中的对象建立了映射关系，是连接数据库的桥梁，用于将程序中的数据对象自动持久化到关系数据库中。

ORM 框架一般以中间件的形式存在，目前比较常用的ORM 是国外非常流行的JPA和国内非常流行的MyBatis。

## 5.2 JDBC

JDBC（Java Data Base Connectivity）由一组用Java编写的类和接口组成

- 是Java用于连接数据库的规范，为大部分关系型数据库提供访问接口，即用于执行数据库SQL语句的 Java API，是比较底层的高效的 API，在精通后适合用于性能调优
- JDBC没有指定某种特定数据库，可以为多种数据库提供统一访问的接口，这也更符合程序设计的模式

JDBC需要每次进行数据库连接，然后处理SQL语句、传值、关闭数据库。这些步骤繁琐易出错，为了减少这种可能的错误，减少开发人员的工作量，`JDBCTemplate`就被设计出来了。`JDBCTemplate`是对JDBC 的封装，更便于程序实现，替开发人员完成所有的JDBC 底层工作。此时，对于数据库的操作不再需要每次都进行连接、打开、关闭了。`JDBCtemplate`操作简单易学，但使用时依旧不够简洁。

## 5.3 MyBatis

[MyBatis](https://mybatis.org/mybatis-3/zh/index.html) 持久层框架支持定制化SQL 、存储过程，以及高级映射，可以使用简单的XML或注解来配置和映射原生信息，将接口和Java的POJOs映射成数据库中的记录，免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作（MyBatis3提供的注解可以取代XML），开发灵活性好。

## 5.4 JPA

Java持久化API（JPA，Java Persistence API）是一系列规范化接口，通过注解或XML描述ORM，并将运行期的实体对象持久化到数据库中，让用户不通过任何配置即可完成数据库的操作。无论是哪种持久化存储方式，数据访问对象（Data Access Objects, DAO）都会提供对对象的增加、删除、修改、查询、排序以及分页等方法。

- `@Entity`

  - 用于声明数据库持久化实体类。

    

- `@Table`

  - 用于声明表名，一般与`@Entity`注解一起使用；

  - 如果表名和实体类名相同，那么`＠Table`可以省略。

    

- `@Id`

  - 用于指定主键属性。

    

- `@Transient`

  - 表示该属性并非一个数据库表的字段的映射， ORM 框架将忽略该属性；

  - 代表不持久的虚拟字段。

    

- `@Column`

  - 指定持久的字段属性，如果字段名与列名相同， 则可以省赂。

    

- `@Basic`

  - 指定实体属性的加载方式，实现关联数据的选择性加载；

  - 有`LAZY`和`EAGER`两种，如`＠Basic(fetch=FetchType.LAZY)`；

  - 懒加载是在属性被引用时才生成查询语句，抽取相关联数据；

  - 实时加载则是执行完主查询后，不管是否被引用，都会马上执府后续的关联数据查询；

  - 使用懒加载来调用关联数据，必须要保证主查询的Session （数据库连接会话）的生
    命周期没有结束，否则是无法抽取到数据的。

    

- `@Repository` ：用于标注和数据库操作有关的接口。

- `@Query`：用于指定接口方法的sql语句。

- `@Param`：用于指定方法参数名称。


# 6、参考资料

- 《Spring In Action，Third Edition》
- https://docs.spring.io/spring-framework/docs/5.3.21/reference/html/
- https://www.liaoxuefeng.com/wiki/1252599548343744/1266263217140032
- https://snailclimb.gitee.io/javaguide/#/?id=springspringboot-%e5%bf%85%e7%9c%8b-1
- https://www.cnblogs.com/zrtqsk/p/3735273.html