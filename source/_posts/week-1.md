---
title: 2022.8.29 - 2022.9.2
date: 2022-09-01 20:57:44
password: 1062597351
tags:
---

> - 工作相关：写单测
>- 博客更新：[Clean Code ](../../../05/06/clean-code/)

目前感觉这份工作比较松弛，入职一个多月了都在不紧不慢的做一些收获甚微的培训，这周终于有活干了，所以我心目中的第一周是从这周开始的。

这周的工作就是补单测，尽可能提高测试覆盖率。

在写单测的过程中，总是回忆起 [Clean Code ](../../../05/06/clean-code/)中的内容，所有不便测试的代码其实都是不够整洁的代码，而足够整洁的代码也都能比较方便的进行测试。TDD的思路能够强迫自己去编写整洁的代码，这种后来专门补单测的行为在混乱代码和覆盖率指标的双重夹击下真的心累。

但是心累不重要，所有的心累都会过去的，重要的是疑惑，所有的疑惑都值得记录，这就是写每周记录的原因。

下面这段代码为一个类定义了一个私有的无参构造器，一旦调用就会抛出异常

```java
public class ClassA {
    private ClassA() {
        throw new IllegalStateException();
    }
}
```

那么为了测这段代码，思路是通过反射调用该类的私有构造器，并断言会抛出异常，并且为了降低测试代码重复率，可以写一个专门用于测这类方法的工具类，代码如下

```java
public class ClassATest {

    @Test
    public void testPrivateConstructor() throws Exception {
        ConstructorTester.testPrivateConstructor(ClassA.class);
    }
}
```

```java
public class ConstructorTester {

    public static void testPrivateConstructor(Class<?> tClass) throws Exception {
        Constructor constructor = tClass.getDeclaredConstructor();
        Assert.assertTrue(Modifier.isPrivate(constructor.getModifiers()));
        constructor.setAccessible(true);
        Assert.assertThrows(InvocationTargetException.class, constructor::newInstance);
    }
}
```

依照这个思路，后续又继续写了一些用于测试无参构造器和全参构造器的工具，代码如下

```java
public class ConstructorTester {

    private static Set<String> skipFieldSet = new HashSet<>();

    static {
        skipFieldSet.add("log");
        skipFieldSet.add("serialVersionUID");
        skipFieldSet.add("__$lineHits$__");
        skipFieldSet.add("jacocoDate");
    }

    public static void testPrivateConstructor(Class<?> tClass) throws Exception {
        Constructor constructor = tClass.getDeclaredConstructor();
        Assert.assertTrue(Modifier.isPrivate(constructor.getModifiers()));
        constructor.setAccessible(true);
        Assert.assertThrows(InvocationTargetException.class, constructor::newInstance);
    }

    public static void testNoArgsConstructor(Class<?> tClass) throws Exception {
        Constructor constructor = tClass.getDeclaredConstructor();
        Object obj = constructor.newInstance();
        Assert.assertNotNull(obj);
        MatcherAssert.assertThat(obj, instanceOf(tClass));
    }

    public static void testAllArgsConstructor(Class<?> tClass, Object[] values) throws Exception {
        Field[] fields = getCleanFields(tClass);
        int length = fields.length;
        Class<?>[] tClasses = new Class[length];
        for(int i = 0; i < length; i++) {
            tClasses[i] = fields[i].getType();
            MatcherAssert.assertThat(values[i], instanceOf(tClasses[i]));
        }

        Constructor constructor = tClass.getDeclaredConstructor(tClasses);
        Object obj = constructor.newInstance(values);
        Assert.assertNotNull(obj);

        for (int i = 0; i < length; i++) {
            fields[i].setAccessible(true);
            Assert.assertEquals(values[i], fields[i].get(obj));
        }
    }

    private static Field[] getCleanFields(Class<?> tClass) {
        Field[] unCleanFields = tClass.getDeclaredFields();
        List<Field> cleanFiels = new ArrayList<>();
        for (int i = 0; i < unCleanFields.length && !skipFieldSet.contains(unCleanFields[i].getName()); i++) {
            cleanFiels.add(unCleanFields[i]);
        }
        return cleanFiels.toArray(new Field[cleanFiels.size()]);
    }
}
```

其中，定义`skipFieldSet`是为了过滤一些不在测试考虑范围内的类属性（比如平台自动插入的属性）。

但是`skipFieldSet`的存在总是让我觉得不太舒服，因为我无法确定这段代码将来运行在其他平台上是否又会自动插入一些未知的需要过滤的属性，这也意味着这段代码某种程度上无法实现平台无关性。

除此之外，这些构造器可以由IDEA自动生成，仅仅为了提高测试覆盖率而去写这些变了味道的测试代码有意思吗？



