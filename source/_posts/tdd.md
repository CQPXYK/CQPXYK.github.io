---
title: Test Driven Development
date: 2022-09-30 15:21:35
tags:
---

# 1、概述

单元测试（unit test）是为了检验程序的正确性。一个单元可能是单个程序、类、对象、方法等，它是应用程序的最小可测试部件。

在编写UT时，应当遵循一些规范：

- 代码简单易懂，不可能再为测试代码编写测试；

- 每个测试都互相独立，不依赖运行顺序；

- 不要在`setup`中初始化只在某些测试中使用的对象；

- 不要写冗长的`setup`；

- 不要在`setup`中创建伪对象；

- 注意测试边界条件。

  

与单元测试不同，集成测试依赖于真实的环境以及组件，因此如果一个测试需要使用真实的系统时间，真实的文件系统或者一个真实的数据库，那么就属于集成测试，集成测试结果更加不稳定。

# 2、Junit

Java平台使用的测试框架是JUnit，JUnit是白盒测试，知道测试如何完成功能和完成什么样的功能。

Junit5提供了以下注解（**由于配置问题，使用Junit4相关注解的UT无法被sonar扫描到，注意区分Junit4和Junit5之间的区别**）

- `@Test`

  - 用于标注测试方法；

  - 每次运行一个`@Test`方法前，Junit首先创建一个`XxxTest`实例；

  - 每个`@Test`方法内部的成员变量都是独立的，不能也无法把成员变量的状态从一个`@Test`方法带到另一个`@Test`方法；

  - Junit5中`@Test`不再有参数，每个参数都被移到了一个函数中。

    

- `@BeforeEach`和`@AfterEach`

  - 在每个测试方法前后自动运行；

  - 用于初始化和清理实例变量，在各个测试方法中互不影响（是不同的实例）；

  - 编写测试前准备、测试后清理的固定代码（称之为Fixture）;

  - 好的UT永远不会用到`@AfterEach`，如果用到了那么很可能是因为使用了实际的数据库等外部资源，所编写的其实是集成测试。

    

- `@BeforeAll`和`@AfterAll`

  - 在所有测试方法前后运行；

  - 用于初始化和清理静态变量，必须被标注为静态方法；

  - 在各个测试方法中均是唯一相同实例，会影响各个`@Test`方法。

    

- `@ExtendWith`

  - 在JUnit中有很多运行器负责调用测试代码；

  - 每个运行器都有特殊功能，应根据需要选择不同的运行器来运行测试代码；

  - Junit5中`@ExtendWith`是可重复的，可以使用多个运行器，这意味着多个扩展可以很容器的组合在一起。

    

- `@Transactional`

  - 测试完成后就会回滚，不会产生垃圾数据。

    

- `@Disabled`

  - 执行测试时将忽略掉此方法；
  - 如果用于修饰类，则忽略整个类。 

# 3、Mockito

如果要测试的对象依赖于另一个无法控制或者还未实现的对象，那么就应该使用 Stub（存根）。Stub 是对系统中存在的一个依赖项（或者协作者）的可控制的替代物，通过使用 Stub，在测试代码时无需直接处理这个依赖项。

如果要验证被测试对象是否按预期的方式调用了该被测试对象，并因此导致单元测试通过或失败，那么就应该使用 Mock。Mock 对象会保存交互的历史记录，这些记录之后用于预期的验证，通常每个测试最多有一个 Mock。

Stub 和 Mock 都是伪对象，一个伪对象究竟是 Stub 还是 Mock ，取决于它在当前测试中的使用方式。如果这个伪对象用来检验一个交互，它就是 Mock 对象，否则就是 Stub 对象。 Stub 对象不会导致测试失败，但 Mock 对象可以。

如果需要创建真实对象，那么就应该使用 Spy。使用 Spy 时将调用真正的方法（除非方法被 stubbing）。Spy 与 Mock 相反，Mock 对象在调用未 stubbing 的方法时返回缺省值，而 Spy 对象在调用未 stubbing 的方法时返回真实调用值。

Mockito 框架可以与JUnit结合使用，可用于创建和配置伪对象。Mockito 提供了以下 API

- `mock(Class<?> tClass)`：返回一个伪对象，可声明未 stubbing 时真实调用的缺省值

- `@Mock`注解配合`MockitoAnnotations.initMocks`或者`@ExtendWith(MockitoExtension,class)`一起使用，用于声明一个 Mock 对象

- `@MockBean`用于需要依赖 Spring 容器的测试，并在替换 Spring 容器中的 Bean，或者增加 Bean

- `@InjectMocks`用于声明并创建需要进行测试和方法调用的真实 Class 实例

- `verify(T mock)`：验证 Mock 对象的行为是否发生及行为发生的次数

- `when(T mock).thenReturn(Object obj)`：控制 Stub 对象的返回

  - 不能用于 Spy 对象

  - stubbing 可以被覆盖，真正的返回取决于最后一次 stubbing 

  - `thenCallRealMethod`显示指定某些情况下调用真实实现

  - 尽量不使用`thenAnswer`

    

- `doxxx(Object obj).when(T mock)`：stubbing void 方法或者 **spy 对象**上的方法，或者用于多次 stubbing 相 同的方法，以在测试过程中更改模拟的行为

- `spy(new xxx())`：返回一个真实的 Spy 对象

- `spy(Class<?> tClass)`可为一个抽象类强行创建出实例进行测试

  

# 4、PowerMock

算是 Mockito 的一个扩展，用于 Mock 静态方法和私有方法。