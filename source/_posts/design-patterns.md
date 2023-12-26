---
title: Design Patterns
date: 2022-05-11 18:44:01
tags:
---

# 1、类图

## 1.1 泛化关系（Generalization）

用于描述非抽象类的继承关系，关键字为`extends`。

<img src="泛化.png" alt="泛化" style="zoom:70%;" />

```java
class Vehicle {...}
class Car extends Vehicle {...}
class Motorcycle extends Vehicle {...}
```

## 1.2 实现关系（Realization）

用于描述抽象类/接口的实现关系，关键字为`implements`。

<img src="实现.png" alt="实现" style="zoom:70%;" />

 ```java
interface Vehicle {...}
class Car implements Vehicle {...}
class Motorcycle implements Vehicle {...}
 ```

## 1.3 组合关系（Composition）

用于描述整体和部分之间的强依赖关系，整体不存在时部分也不存在。

<img src="复合.png" alt="复合" style="zoom:70%;" />

## 1.4 聚合关系（Aggregation）

用于描述整体和部分个体之间的非强依赖关系，整体不存在时部分个体依旧存在。

<img src="聚合.png" alt="聚合" style="zoom:70%;" />

## 1.5 关联关系（Association）

用于描述类对象之间的静态强关联关系，该关系与运行过程的状态无关，在运行前就可以确定。关联关系通常以成员变量（静态方法变量）的形式实现。

关联关系默认不强调方向，表示对象之间默认互相知道，如果特别强调方向（如下图），则表示A知道B，但B不知道A。

<img src="关联.png" alt="关联" style="zoom:70%;" />

## 1.6 依赖关系（Dependency）

用于描述类对象之间的临时性动态关联关系，该关系与运行过程中的状态相关，在运行时确定。依赖应当总是单向的，要杜绝双向依赖及循环依赖的产生。依赖关系通常以类方法的传入传出参数（或局部变量）实现。

<img src="依赖.png" alt="依赖" style="zoom:70%;" />

 ```java
class Car {
    public Offgas move(Gasoline G) {...}
}
class Gasoline {...}
class Offgas {...}
 ```

**耦合紧密程度：泛化 = 实现 >组合 > 聚合 > 关联 > 依赖**

# 2、设计模式

设计模式是解决一些固有问题的一系列方案，学习现有的设计模式可以做到经验复用。

设计模式总是试图令变化的事物与不变的事物相互分离。

## 2.1 创建型

创建型的核心思想在于将对象的创建与使用相分离。

### 2.1.1 单例（Singleton）

确保在一个进程中一个类有且只有一个实例，并提供该实例的全局访问点。

1）线程安全；

 ```java
class Singleton {
    private static final Singleton INSTANCE = new Singleton();
    
    public static Singleton getInstance() {
        return INSTANCE;
    }
    private Singleton() {...}
}
 ```

2）延迟加载，使用同步访问方法以防止循环初始化，但也增加了访问成本，线程安全，性能不好；

 ```java
class Singleton {
    private static Singleton INSTANCE = null;
    
    public synchronized static Singleton getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }
    private Singleton() {...}
}
 ```

3）基于`volatile`的双重检查锁定，除了可以对静态字段实现延迟初始化外，还可以对实例化字段实现延迟初始化，线程安全，性能较上一种方式有所提升；

```java
public class Singleton {
    private volatile static Singleton INSTANCE = null;
    
    public static Singleton getInstance() {
        if (INSTANCE == null) {
            synchronized (Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

4）针对静态域的延迟加载，lazy initialization holder class模式，静态访问方法不需要同步，不会因延迟初始化增加访问成本，线程安全，性能好；

 ```java
class Singleton {
    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
    private Singleton() {...}
    
    private static class SingletonHolder {
        public static final Singleton INSTANCE = new Singleton();
    }
}
 ```

5）枚举类型，绝对安全；

 ```java
enum Singleton {
    INSTANCE;
}
 ```

6）一般不是通过`new`操作符，而是由框架（例如Spring）来创建装配从而获取单例，`@Component`注解默认将组件标注为单例。

### 2.1.2 工厂方法（Factory Method）

提供用于创建一个对象的简单工厂类/工厂接口。

1）如下所示，`User`类关联`Product`接口和`SimpleFactory`工厂类，通过`SimpleFactory`简单工厂实现具体`Product`的实例化。`User`类可忽略具体`Product`（`ProductA`和`ProductB`）的细节，从而将`User`类与具体`Product`实现类解耦。

<img src="简单工厂.png" alt="简单工厂" style="zoom:70%;" />

 ```java
interface Product{...}
class ProductA implements Product{...}
class ProductB implements Product{...}

class SimpleFactory {
    public Product createProduct (int type) {
        if (type == 1) {
            retrun new ProductA();
        }
        else if (type == 2) {
            retrun new ProductB();
        }
    }
}

class User {
    public static void main (String[] args) {
        SimpleFactory simpleFactory = new SimpleFactory();
        Product product = simpleFactory.createProduct(1);
    }
}
 ```

2）以上所述简单工厂类可以改为工厂接口，如下所示，`User`类关联`Product`接口和`ProductFactory`接口，通过`ProductFactory`接口的具体类（`ProductFactoryImpl`）实现具体`Product`（`ProductA`）的实例化。`User`类可忽略真正的工厂（`ProductFactoryImpl`）和具体`Product`（`ProductA`）的细节，从而将`User`类与工厂类及具体`Product`实现类解耦。

<img src="工厂方法.png" alt="工厂方法" style="zoom:70%;" />

 ```java
interface Product{...}
class ProductA implements Product{...}

interface ProductFactory {
    private static ProductFactory impl = new ProductFactoryImpl();
    
    public static ProductFactory getFactory() {
        return impl;
    }
    
    public Product factoryMethod();
}

class ProductFactoryImpl implements ProductFactory {
    public Product factoryMethod() {
        return new ProductA();
    }
}

class User {
    public static void main (String[] args) {
        ProductFactory factory = ProductFactory.getFactory();
        Product product = factory.factoryMethod();
    }
}
 ```

3）以上所述工厂方法可化简为静态工厂方法（Static Factory Method），如下所示

<img src="静态工厂.png" alt="静态工厂" style="zoom:70%;" />

 ```java
interface Product{...}
class ProductA implements Product{...}

interface ProductFactory {
    public static Product factoryMethod() {
        if (...) {
            // return interface from cache
        }
        return new ProductA();
    }
}

class User {
    public static void main (String[] args) {
        Product product = ProductFactory.factoryMethod();
    }
}
 ```

这样做的好处在于`factoryMethod`方法可能会`new`一个新的`ProductA`实例，也可能直接返回一个缓存的实例，对于`User`类来说，无需知道`ProductA`创建的细节。

4）总结：

- 实际更常用的是更简单的静态工厂方法；
- 工厂方法定义工厂接口和产品接口，但创建实际工厂和实际产品被推迟到子类实现，使得调用方只和抽象工厂和抽象产品打交道；
- 调用方尽量持有接口或者抽象类，避免持有具体类型的子类，以便工厂方法能随时切换不同的子类返回，而不影响调用方代码。

### 2.1.3 抽象工厂（Abstract Factory）

提供**用于创建对象家族**的工厂接口（对象家族指很多个必须一起创建出来的相关的对象），对象家族中的单一对象由工厂方法创建出来，即实例化操作由该工厂接口的具体实现类完成。

如下所示，`User`类关联`Product`接口、`Tool`接口以及`AbstractFactory`接口，通过`AbstractFactory`接口的具体类（`FactoryImplA`）实现具体`Product`（`ProductA`）和具体`Tool`（`ToolA`）的实例化。`User`类可忽略真正的工厂（`FactoryImplA`）、具体`Product`（`ProductA`）和具体`Tool`（`ToolA`）的细节，从而将`User`类与工厂类、具体`Product`实现类及具体`Tool`实现类解耦。

<img src="抽象工厂.png" alt="抽象工厂" style="zoom:70%;" />

 ```java
interface Product {...}
interface Tool {...}

interface AbstractFactory {
    public static AbstractFactory createFactoryA() {
        return new FactoryImplA();
    }
    public Product createProduct();
    public Tool createTool();
}

class ProductA implements Product {...}
class ToolA implements Tool {...}

class FactoryImplA implements AbstractFactory {
    public Product createProduct() {
        return new ProductA();
    }
    public Tool createTool() {
        return new ToolA();
    }
}

class User {
    public static void main (String[] args) {
        AbstractFactory factoryA = AbstractFactory.createFactoryA();
        Product product = factoryA.createProduct();
        Tool tool = factoryA.createTool();
    }
}
 ```

### 2.1.4 生成器（Builder）

通过多个可选的步骤或者多个可选的零件来灵活创建一个复杂的对象。

Builder模式并不直接生成一个`Product`类的对象，而是让客户端调用生成器得到一个`Builder`对象，然后在`Builder`对象上调用类似于`setter`的方法来设置可选参数，最后调用无参的`builder`方法来生成真正需要的（通常是不可变类的）`Product`类对象。

如下所示，`Builder`类通常作为需要构建的类的静态成员类，`Builder`类的设置方法均返回`this`，以便**将调用链接起来，得到一个流式的API**。

<img src="生成器.png" alt="生成器" style="zoom:70%;" />

 ```java
class Product {
    private final int requiredParameter;
    private final int optionalParameter;
    
    private Product(Builder builder) {
        this.requiredParameter = builder.requiredParameter;
        this.optionalParameter = builder.optionalParameter;
    }
    
    public static class Builder {
        private final int requiredParameter;
        private int optionalParameter1;
        private int optionalParameter2;
        
        public Builder(int requiredParameter) {
            this.requiredParameter = requiredParameter;
        }
        
        public Builder setOptionalParameter1(int optionalParameter1) {
            this.optionalParameter1 = optionalParameter1;
            return this;
        }        
        public Builder setOptionalParameter2(int optionalParameter2) {
            this.optionalParameter2 = optionalParameter2;
            return this;
        }
        
        public Product build() {
            return new Product(this);
        }
    }
}

class User {
    public static void main (String[] args) {
        Product product = new Product.Builder(requiredParameter)
            .setOptionalParameter1(optionalParameter1)
            .setOptionalParameter2(optionalParameter2)
            .build();
    }
}
 ```

### 2.1.5 原型（Prototype）

用原型实例指定创建对象的类型，并且通过拷贝这个原型来创建新的对象。

原型模式应用不是很广泛，因为很多实例会持有一些无法拷贝给另一个对象共享的资源，只有一些简单属性可以拷贝。

```java
public abstract class Prototype {
    public abstract Prototype myClone();
}

public class ConcretePrototype extends Prototype {

    private String filed;

    public ConcretePrototype(String filed) {
        this.filed = filed;
    }

    @Override
    public Prototype myClone() {
        return new ConcretePrototype(filed);
    }
}

public class User {
    public static void main(String[] args) {
        Prototype prototype = new ConcretePrototype("abc");
        Prototype clone = prototype.myClone();
        System.out.println(prototype == clone); // false
    }
}
```

## 2.2 行为型

行为型的核心思想在于通过对象之间的协作来灵活完成一个整体任务。

### 2.2.1 责任链（Chain of Responsibility）

将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止，使得多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。

<img src="责任链1.png" alt="责任链1" style="zoom:70%;" />

```java
public enum RequestType {
    TYPE1, TYPE2
}
public class Request {...}

public enum ResponseType {
    TRUE, FALSE, NEXT
}
public class Response {...}

public interface Handler {
    Response process(Request request);
}
public class HandlerImpl1 implements Handler {...}
public class HandlerImpl2 implements Handler {...}

public class HandlerChain {
    // 持有所有Handler:
    private List<Handler> handlers = new ArrayList<>();

    public void addHandler(Handler handler) {
        this.handlers.add(handler);
    }

    public Response processHandlers(Request request) {
        // 依次调用每个Handler:
        for (Handler handler : handlers) {
            Response response = handler.process(request);
            if (response != null && !ResponseType.NEXT.equals(response.getResponseType())) {
                System.out.println(response.getMessage());
                return response;
            }
        }
        throw new RuntimeException("Could not handle request: " + request);
    }
}

class User {
    public static void main(String[] args) {
        // 构造责任链:
        HandlerChain chain = new HandlerChain();
        chain.addHandler(new HandlerImpl1());
        chain.addHandler(new HandlerImpl2());
        // 构造请求:
        Request request = new Request(RequestType.TYPE1);
        // 处理请求:
        Response response = chain.processHandlers(request);
    }
}
```

总结：

- 责任链模式的好处在于添加新的处理器或者重新排列处理器非常容易，经常用在拦截、预处理请求等；

- `Handler`的添加的顺序很重要，如果顺序不对，处理的结果可能就不是符合要求的；

- 也可以在`Handler`中添加成员`Handler`，从而指定每个`Handler`的下一个`Handler`并传递`Request`。

  <img src="责任链2.png" alt="责任链2" style="zoom:70%;" />

  

### 2.2.2 命令（Command）

将一个请求命令封装为一个对象，使得：1）可用不同的请求对客户进行参数化，2）对请求排队，3）记录请求日志，4）支持可撤销的操作。

<img src="命令.png" alt="命令" style="zoom:70%;" />

```java
public interface Command {
    void execute();
    void undo();
    void redo();
}

public class XXXCommand implements Command {
    private XXX receiver; // 执行方对象:

    public XXXCommand(XXX receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.xxx();
    }
    
    @Override
    public void undo() {...}
    
    @Override
    public void redo() {...}
}

public class Invoker {
    private List<Command> commands = new ArrayList<>();
    
    public void addCommand(Command command) {
        commands.add(command);
    }
    
    public void invok(int index) {
        list.get(index).execute();        
    }
    
    public void undo(int index) {
        list.get(index).undo();        
    }
    
    public void redo(int index) {
        list.get(index).redo();        
    }
}

class User {
    public static void main(String[] args) {
        Invoker invoker = new Invoker();
        invoker.addCommand(new XXXCommand(new XXX()));
        invoker.invoke(0);
    }
}
```

总结：

- 命令模式把命令的创建和执行分离，调用方`Invoker`只发送命令，而不关心执行方`receiver`具体执行命令的详细过程；

- 在执行方对象`receiver`比较简单时，命令模式增加了系统的复杂度；

- 当执行方对象`receiver`复杂到一定程度，并且需要支持Undo、Redo等功能时，命令模式通过封装`Command`对象，并通过`Invoker`将一系列已执行或未执行的`Command`保存起来，从而支持撤销、重做等操作。

  

### 2.2.3 解释器（Interpreter）

根据某种语言来定义一个解释器，解释器通过抽象语法树实现对用户输入的解释执行。

比如用于匹配字符串的正则表达式（`java.util.regex.Pattern`）就是一个解释器。

解释器模式的实现通常非常复杂，且一般只能解决一类特定问题。

### 2.2.4 迭代器（Iterator）

提供一个用于顺序访问聚合对象中的各个元素的方法，并且不暴露该聚合对象的内部表示。

如下所示，实现Iterator模式的关键是返回一个`Iterator`对象，该对象知道集合的内部结构。

<img src="迭代器.png" alt="迭代器" style="zoom:70%;" />

```java
public interface Iterator<E> {
    boolean hasNext();
    E next();
}

public class ConcreteIterator<E> implements Iterator<E> {
    private Object[] elements;
    private int position = 0;
    
    public ConcreteIterator(Object[] elements, int size) {
        this.elements = Arrays.copyOf(elements, size);
    }
    
    @Override
    public boolean hasNext() {
        return position < elements.length;
    }
    
    @Override
    public E next() {
        return (E) elements[position++];
    }
}
```

```java
public interface Iterable<T> {
    Iterator<T> iterator();
}

public interface Aggregate<E> extends Iterable<E> {
    void add(E e);
}

public class ConcreteAggregate<E> implements Aggregate<E> {
    private int size = 0;
    private int maxsize = 1;
    private Object[] elements = new Object[maxsize];

    @Override
    public Iterator<E> iterator() {
        return new ConcreteIterator<E>(elements,size);
    }

    @Override
    public void add(E e) {
        if (size == maxsize) {
            maxsize *= 2;
            elements = Arrays.copyOf(elements,maxsize);
        }
        elements[size++] = e;
    }
}
```

```java
class User {
    public static void main(String[] args) {
        Aggregate aggregate = new ConcreteAggregate();
        for (int i = 0; i < 10; i++) {
            aggregate.add(i);
        }
        Iterator<Integer> iterator = aggregate.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

### 2.2.5 中介者/调停者（Mediator）

用一个中介对象来封装一系列复杂的对象交互，将多方会谈改为多个双方会谈。

如下所示，模块可以通过调用中介者来间接调用相关模块。

<img src="中介者.png" alt="中介者" style="zoom:70%;" />

```java
public interface Mediator {
    void doEvent(String eventType);
}

public interface Module {
    // 模块可以通过调用中介者来间接调用相关模块
    void onEvent(Mediator mediator);
    
    void fun();
}

public class ModuleA implements Module {
    @Override
    public void onEvent(Mediator mediator) {
        mediator.doEvent("type1");
    }
    
    @Override
    public void fun() {
        System.out.println("ModuleA.fun()");
    }
}

public class ModuleB implements Module {...}
public class ModuleC implements Module {...}
public class ModuleD implements Module {...}
```

```java
public class ConcreteMediator implements Mediator {
    private Module a,b,c,d;

    public ConcreteMediator(Module a, Module b, Module c, Module d) {
        this.a = a;
        this.b = b;
        this.c = c;
        this.d = d;
    }

    @Override
    public void doEvent(String eventType) {
        switch (eventType) {
            case "type1":
                interact1();
                break;
            case "type2":
                interact2();
                break;
            default:
                interact3();
        }
    }

    public void interact1() {
        b.fun();
        c.fun();
        d.fun();
    }
    
    public void interact2() {...}
    public void interact3() {...}
}
```

```java
class User {
    public static void main(String[] args) {
        Module a = new ModuleA();
        Module b = new ModuleB();
        Module c = new ModuleC();
        Module d = new ModuleD();
        Mediator mediator = new ConcreteMediator(a,b,c,d);
        // 模块可以通过调用中介者来间接调用相关模块
        a.onEvent(mediator);
    }
}
```

运行结果：

```java
ModuleB.fun()
ModuleC.fun()
ModuleD.fun()
```

总结：

- 中介使得各个组件不需要显式地相互引用，降低了组件间的耦合关系；
- 组件间的交互可以独立地进行改变而不影响其他的组件；
- 中介经常用在有众多交互组件的UI上，MVC模式可以看作是中介模式的扩展。

### 2.2.6 备忘录（Memento）

在不破坏封装性的前提下获得一个对象的内部状态，并在需要时将该对象恢复到之前的状态。

<img src="备忘录.png" alt="备忘录" style="zoom:70%;" />

```java
public interface Memento {
    int getState();
}

public class MementoImpl implements Memento {
    private int state;
    
    public MementoImpl(int state) {
        this.state = state;
    }
    
    @Override
    public int getState() {
        return state;
    }
}
```

```java
public interface Originator {
    void setState(int state);
    int getState();
    Memento createMemento();
    void setMemento(Memento memento);
}

public class OriginatorImp implements Originator {
    private int state;
    
    @Override
    public void setState(int state) {
        this.state = state;
    }
    
    @Override
    public int getState() {
        return state;
    }
    
    @Override
    public Memento createMemento() {
        return new MementoImpl(state);
    }
    
    @Override
    public void setMemento(Memento memento) {
        this.state = memento.getState();
    }
}
```

```java
class User {
    public static void main(String[] args) {
        Originator originator = new OriginatorImp();
        // 原始状态
        originator.setState(33);
        System.out.println(originator.getState());
        // 存储当前状态以备不时之需
        Memento memento = originator.createMemento();
        // 设置新状态
        originator.setState(71);
        System.out.println(originator.getState());
        // 恢复上一个状态
        originator.setMemento(memento);
        System.out.println(originator.getState());
    }
}
```

运行结果：

```java
33
71
33
```

总结：

- Java的序列化、Undo、Redo这些功能，都可以看作是备忘录模式；

- 对于简单的对象模型，用一个`String`就可以表示其状态，对于复杂的对象模型，通常会使用JSON、XML等复杂格式。

  

### 2.2.7 观察者/监听者（Observer/Listener）

定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动被更新，又称发布-订阅（Publish-Subscribe）模式，发布方和订阅方彼此分离，互不影响。

如下所示，观察者`Admin`和`Customer`与被观察者`Store`彼此分离，互不影响，被观察者将通知变成一个`Event`对象，观察者自己从`Event`对象中读取通知类型和通知数据。

<img src="观察者.png" alt="观察者" style="zoom:70%;" />

 ```java
public interface Observable {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers(Event event);
}

public interface Observer {
    void onEvent(Event event);
}

// 商店是被观察者
public class Store implemets Observable {
    private List<Observer> observers = new ArrayList<>();
    
    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(Event event) {
        for (Observer observer : observers) {
            observer.onEvent(event);
        }
    }
}

// 管理员是观察者
public class Admin implemets Observer {
    @Override
    public void onEvent(Event event) {
        System.out.println("Admin 接收到 " + event.toString() + "事件");
    }
}
// 客户是观察者
public class Customer implemets Observer {...}

class User {
    public static void main (String[] args) {
        Store store = new Store();
        store.registerObserver(new Admin());
        store.registerObserver(new Customer());
        // 商店给注册的管理员和客户发送商品消息
        store.notifyObservers("来自 Store 的商品消息");
    }
}
 ```

总结：

- 不推荐借助Java标准库中的`java.util.Observable`类和`Observer`接口来实现观察者模式；

- 不同的`Event`对象可以有着不同的通知Topic，观察者可以自行订阅自己感兴趣的Topic。

  

### 2.2.8 状态（State）

将不同状态下的行为逻辑分拆到不同的状态类中，允许对象在其内部状态改变时改变其行为。

<img src="状态.png" alt="状态" style="zoom:70%;" />

 ```java
public interface State {
    void handle();
}
public class StateA implemets State {...}
public class StateB implemets State {...}
public class StateC implemets State {...}

class Context {
    private State state = new StateA();
    
    public void transfer (String input) {
        if (...) {
            state = new StateA();
            state.handle();
        }
        else if (...) {
            state = new StateB();
            state.handle();
        }
        else {
            state = new StateC();
            state.handle();
        }
    }
}

class User {
    public static void main (String[] args) {
        Scanner sc = new Scanner(System.in);
        Context ctx = new Context();
        while (true) {
            ctx.transfer(sc.nextLine());
        }
    }
}
 ```

总结：

- 状态模式常用于带有状态的对象中，增加新状态非常容易；

- 状态模式的关键在于状态转换，简单的状态转换可以直接由调用方指定，复杂的状态转换可以在内部根据条件触发完成。

### 2.2.9 策略（Strategy）

定义一系列算法，这些算法被封装到一个对象中，策略模式使得这些算法在运行时可以互相替换，并且独立于使用这些算法的客户端。

<img src="策略.png" alt="策略" style="zoom:70%;" />

 ```java
public interface Strategy {
    void handle();
}
public class StrategyA implemets Strategy {...}
public class StrategyB implemets Strategy {...}
public class StrategyC implemets Strategy {...}

class Context {
    private Strategy strategy = new StrategyA();
    
    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }
    
    public void apply() {
        this.strategy.handle();
    }
}

class User {
    public static void main(String[] args) {
        Context ctx = new Context();
        ctx.apply(); //使用默认策略
        
        ctx.setStrategy(new StrategyB);
        ctx.apply(); //使用指定策略
    }
}
 ```

总结：

- 一个完整的策略模式要定义策略以及使用策略的上下文，策略模式的核心思想是在一个计算方法中把容易变化的算法抽出来作为“策略”参数传进去，使得新增策略不必修改原有逻辑；

- 状态模式与策略模式类似，都能够动态改变对象的行为，状态模式通过状态转移来改变`State`对象，而策略模式通过本身决策来改变所使用的算法；

- 状态模式主要是用于解决动态的状态转移问题，策略模式主要是用于动态地替换所使用的算法；

- 策略模式和Lambda表达式配合使用可极大提高程序灵活性，例如`Arrays.sort (T[] a, Comparator<? Super T> c)`方法就可根据lambda表达式所提供的比较策略`c`来实现不同比较策略下的排序。

### 2.2.10 模板方法（Template Method）

定义一个操作中的算法框架，并将一些步骤的实现延迟到子类，使得子类可以在不改变算法结构的同时修改一些步骤的实现。

如下所示，清洗、调料以及烹饪这三步是固定的，但是可以在`RoastChicken`和`FriedChicken`子类中，定义`souse`和`cook`函数的具体实现。

 ```java
public abstract class Chicken {
    public final void make() {
        clean();
        souse();
        cook();
    }
    
    private void clean() {...}
    protected abstract void souse();
    protected abstract void cook();
}

public class RoastChicken extends Chicken {
    @Override
    protected void souse() {...}      
    @Override
    protected void cook() {...}
}
public class FriedChicken extends Chicken {
    @Override
    protected void souse() {...}      
    @Override
    protected void cook() {...}      
}

class User {
    public static void main(String[] args) {
        Chicken chicken1 = new RoastChicken();
        chicken1.make();
        Chicken chicken2 = new FriedChicken();
        chicken2.make ();
    }
}
 ```

总结：

- 模板方法的核心思想是：父类定义骨架，子类实现某些细节；

- 为了防止子类重写父类的骨架方法，可以在父类中对骨架方法使用`final`关键字；

- 对于需要子类实现的抽象方法，一般声明为`protected`，使得这些方法对外部客户端不可见。

  

### 2.2.11 访问者（Visitor）

抽象出作用于一组复杂对象结构的操作，将对象结构与操作分离开，当需要增加新操作时无需改变现有结构。

如下所示，通过`ElementStructure`来组装对象结构，通过`Visitor`的实现类来定义作用于对象的操作，当需要增加新操作时只需增加`Visitor`的新实现类来完成新操作即可。

<img src="访问者.png" alt="访问者" style="zoom:70%;" />

```java
public interface Element {
    void accept(Visitor visitor);
    String getName();
}

public class ElementA implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
    
    @Override
    public String getName() {
        return "ElementA";
    }
}

public class ElementB implements Element {...}

public class ElementStructure {
    private List<Element> elements = new ArrayList<>();
    
    public void handle(Visitor visitor) {
        for (Element element : elements) {
            element.accept(visitor);
        }
    }
    
    void addElement(Element element) {
        elements.add(element);
    }
}
```

```java
public interface Visitor {
    void visit(ElementA element);
    void visit(ElementB element);
}

public class VisitorA implements Visitor {
    @Override
    public void visit(ElementA element) {
        System.out.println("Visit " + element.getName() + " by VisitorA");
    }
    
    @Override
    public void visit(ElementB element) {
        System.out.println("Visit " + element.getName() + " by VisitorA");
    }
}

public class VisitorB implements Visitor {...}
```

```java
class User {
    public static void main(String[] args) {
        // 定义对象结构
        ElementStructure es = new ElementStructure();
        es.addElement(new ElementA());
        es.addElement(new ElementB());
        // 访问对象结构
        es.handle(new VisitorA());
        es.handle(new VisitorB());
    }
}
```

运行结果：

```java
Visit ElementA by VisitorA
Visit ElementB by VisitorA
Visit ElementA by VisitorB
Visit ElementB by VisitorB
```

## 2.3 结构型

结构型的核心思想在于通过继承、实现以及组合机制使得对象之间具备良好、灵活的结构。

### 2.3.1 适配器（Adapter/Wrapper）

把一个抽象类/接口转换成另一个用户需要的抽象类/接口，使得不兼容的接口可以一起正常工作。

如下所示，`User`类通过`DogToCatAdapter`适配器将`Dog`接口转换成用户需要的`Cat`接口，经过转换，用户在调用`Cat`接口的`meow`方法时，实际间接调用的是`Dog`接口`bark`方法。

<img src="适配器.png" alt="适配器" style="zoom:70%;" />

 ```java
interface Dog {
    void bark();
}
interface Cat {
    void meow ;
}

class DogToCatAdapter implements Cat {
    Dog dog;
    
    public DogToCatAdapter(Dog dog) {
        this.dog = dog;
    }
    
    @Override
    public void meow() {
        dog.bark();
    }
}

class User {
    public static void main(String[] args) {
        Cat cat = new DogToCatAdapter(new Dog());
        cat.meow();
    }
}
 ```

### 2.3.2 桥接（Bridge）

将抽象部分与它的实现部分分离，使它们都可以独立地变化。

如下所示，`Abstraction`和`Implementor`互相分离，都可以独立的派生出子类，任何`Abstraction`的子类都可以和任何`Implementor`的实现类互相组合得到用户想要的类。

<img src="桥接.png" alt="桥接" style="zoom:70%;" />

```java
public interface Implementor {
    void getImplementor();
}

public class ConcreteImplementor implements Implementor {
    @Override
    public void getImplementor() {
        return("ConcreteImplementor");
    }
}

public abstract class Abstraction {
    protected Implementor implementor;

    public Abstraction(Implementor implementor) {
        this.implementor = implementor;
    }
    
    public abstract void getAbstraction();
    public abstract void fun();
}

public class ConcreteAbstraction extends Abstraction {
    public ConcreteAbstraction(Implementor implementor) {
        super(implementor);
    }
    
    @Override
    public void getAbstraction() {
        return("ConcreteAbstraction");
    }

    @Override
    public void fun() {
        System.out.println("fun()");
        System.out.println("Abstraction:" + getAbstraction());
        System.out.println("Implementor:" + implementor.getImplementor());
    }
}

class User {
    public static void main(String[] args) {        
        Abstraction abstraction = new ConcreteAbstraction(
            new ConcreteImplementor());
        abstraction.fun();
    }
}
```

总结：

- 桥接模式通过将抽象部分和实现部分分离，使得设计可以按两个维度独立扩展；
- 直接继承可能会带来子类爆炸，不要过度使用继承，而是优先拆分某些部件，使用组合的方式来扩展功能。

### 2.3.3 组合（Composite）

将对象组合成树形结构以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性。

如下所示，首先统一单个节点以及“组合”节点的抽象类（`Node`类），`Composite`类可以看作是树的中间节点，它可以有子节点（子节点可以是中间节点或者叶子节点），而`Leaf`类可以看作是树的叶子节点，它没有子节点，用户对`Composite`对象和`Leaf`对象的操作是具备一致性的。

<img src="组合.png" alt="组合" style="zoom:70%;" />

```java
public abstract class Node {
    protected String name;
    
    public Node(String name) {
        this.name = name;
    }
    
    public abstract void print(int level);
    public abstract Node add(Node node);
    public abstract Node remove(Node node);
}

public class Composite extends Node {
    private List<Node> children = new ArrayList<>();
    
    public Composite(String name) {
        super(name);
    }

    @Override
    public void print(int level) {
        for (int i = 0; i < level; i++) {
            System.out.print("——");
        }
        System.out.println("Composite:" + name);
        for (Node node : children) {
            node.print(level + 1);
        }
    }

    @Override
    public void add(Node node) {
        children.add(node);
        return this;
    }

    @Override
    public Node remove(Node node) {
        children.remove(node);
        return this;
    }
}

public class Leaf extends Node {
    public Leaf(String name) {
        super(name);
    }

    @Override
    public void print(int level) {
        for (int i = 0; i < level; i++) {
            System.out.print("——");
        }
        System.out.println("left:" + name);
    }

    @Override
    public Node add(Node node) {
        throw new UnsupportedOperationException(); 
    }

    @Override
    public Node remove(Node node) {
        throw new UnsupportedOperationException();
    }
}

class User {
    public static void main(String[] args) {
        Node root = new Composite("root");
        root.add(new Leaf("a"));
        root.add(new Composite("A")
                 .add(new Leaf("b")).add(new Leaf("c")));
        root.add(new Composite("B")
                 .add(new Leaf("d")).add(new Leaf("e")));
        root.add(new Leaf("f"));
        root.print(0);
    }
}
```

运行结果：

```java
Composite:root
——left:a
——Composite:A
————left:b
————left:c
——Composite:B
————left:d
————left:e
——left:f
```

### 2.3.4 装饰器（Decorator）

在运行期动态地给对象实例增加各种新的功能。

如下所示，抽象类`ProductDecorator`内复合了`Product`接口，通过扩展`ProductDecorator`可以得到各种具备不同功能的产品，用户可以叠加各种功能得到自己需要的产品。

<img src="装饰器.png" alt="装饰器" style="zoom:70%;" />

 ```java
public interface Product {
    void fun();
}

public class ProductImplA implemets Product {...}

public abstract class ProductDecorator implemets Product {
    protected final Product product;
    
    protected ProductDecorator(Product product) {
        this.product = product ;
    }

    @Override
    public void fun() {
        this.product.fun();
    }
}

public class AppearanceDecorator extends ProductDecorator {
    public AppearanceDecorator(Product product) {
        super(product) ;
    }
    
    public void fun() {
        // 执行一些关于产品外观功能的操作
        product.fun();
        // 执行一些关于产品外观功能的操作
    }
}

public class EnduranceDecorator extends ProductDecorator {
    public EnduranceDecorator(Product product) {
        super(product);
    }
    
    public void fun() {
        // 执行一些关于产品续航功能的操作
        product.fun();
        // 执行一些关于产品续航功能的操作
    }
}

class User {
    public static void main(String[] args) {
        Product product = new ProductImplA();
        Product product1 = new AppearanceDecorator(product);
        Product product2 = new EnduranceDecorator(product);
        Product product3 = new EnduranceDecorator(
            new AppearanceDecorator(product));
            // ...
    }
}
 ```

总结：

- 可以代替相同功能的子类，从而降低设计各种子类的工作量；

- 把核心功能和附加功能分开，两部分都可以独立地扩展，最后由调用方在运行期间动态地自由组合得到自己需要的功能；

- Proxy让调用者认为获取到的是核心类接口，但实际上是代理类，而Decorator让调用者自己创建核心类（`ProductImpl`），然后在核心类上一层一层地叠加各种功能。

  

### 2.3.5 外观（Facade）

定义了一个高层接口，这个接口为子系统中的一组接口提供一个一致的界面，使得子系统更加容易使用。

如下所示，完成一道烤鸡需要清洗、调料以及烹饪这三个步骤，这些步骤都看做是以一个子系统中的一组接口，此时，可以通过`Facade`来一次性完成所有步骤，用户只需要关联`Facade`，而不关心清洗、调料以及烹饪这些步骤。

<img src="外观.png" alt="外观" style="zoom:70%;" />

```java
public class Food {
    private String name;
    
    public Food(String name) {
        this.name = name;
    }
    public String getName() {
        return name;
    }
}

public class CleanSystem {
    public void clean(Food food) {
        System.out.println("clean " + food.getName());
    }
}

public class SouseSystem {
    public void souse(Food food) {
        System.out.println("souse " + food.getName());
    }
}

public class CookSystem {
    public void cook(Food food){
        System.out.println("cook " + food.getName());
    }
}

public class Facade {
    private Food food;
    
    public Facade(Food food) {
        this.food = food;
    }
    
    public void make() {
        new CleanSystem().clean(food);
        new SouseSystem().souse(food);
        new CookSystem().cook(food);
    }
}

class User {
    public static void main(String[] args) {
        Facade facade = new Facade(new Food("RoastChicken"));
        facade.make();
    }
}
```

总结：

- 通过给客户端提供一个统一入口，可以对外屏蔽内部子系统的调用细节，客户端所需要交互的对象应当尽可能少；
- 对于复杂的Web程序，经常会使用一个统一的网关入口来自动转发到不同的Web服务，这种提供统一入口的网关就是Gateway，它本质上也是一个Facade，还可以附加一些用户认证、限流限速的额外服务。

### 2.3.6 享元（Flyweight）

利用共享的方式来支持大量细粒度的对象。

如下所示，`product1`和`product2`是同一个对象，返回结果为`true`。

 ```java
class Product {
    private static final Map<String, Product> cache = new HashMap<>();
    private final String id;
    
    public static Product factoryMethod(String id) {
        Product product = cache.get(id);
        if (product == null) {
            product = new Product(id);
            cache.put(id, product);
        }
        return product;
    }
    
    public Product(String id) {
        this.id = id;
    }
}

class User {
    public static void main(String[] args) {
        Product product1 = Product.factoryMethod();
        Product product2 = Product.factoryMethod();
        System.out.println(product1 == product2); // true
    }
}
 ```

总结：

- 对于不可变类（实例不能被修改的类）来说，反复创建其相同的实例没有必要，直接向调用方返回一个共享的实例即可；

- 享元模式常用于工厂方法的内部优化，在工厂内部，很可能返回缓存的实例而不是`new`一个新的实例；

- 在实际应用中，享元模式主要应用于缓存，即客户端重复请求某些对象时，不必每次查询数据库或者读取文件，而是直接返回缓存中的对象。

### 2.3.7 代理（Proxy）

对调用方提供一个代理，该代理能够控制对一个对象的访问。

如下所示， `ProductProxy`通过实现`Product`接口，从而在调用`Product`的`fun`方法的前后增加一些额外的操作。

<img src="代理.png" alt="装饰" style="zoom:70%;" />

 ```java
public interface Product {
    void fun();
}

public class ProductImpl implemets Product {...}

public class ProductProxy implemets Product {
    private Product product;
    
    public ProductProxy(Product product ) {
        this.product = product ;
    }
    
    @Override
    public void fun() {
        // 执行一些额外操作
        this.product.fun();
        // 执行一些额外操作
    }
}

class User {
    public static void main(String[] args) {
        Product product = new ProductProxy(new ProductImpl());
        product.fun();
    }
}
 ```

根据代理所增加的额外操作的不同，代理可分为：

- 远程代理（Remote Proxy）：本地调用者持有的接口是一个代理，该代理将对接口的方法转换成远程调用并返回结果，该代理**控制了对远程对象的访问**，此时代理附加的额外操作是对请求对象和返回对象进行编码并发送至远程对象；

- 虚拟代理（Virtual Proxy）：调用者先持有一个代理对象，而真正的对象并没有创建，直到客户端真的需要调用该对象时，才创建真正的对象，该代理**控制了对创建开销很大的对象的访问**，此时代理附加的额外操作是创建真正要访问的对象；

- 保护代理（Protection Proxy）：该代理**根据调用者的权限控制了对对象的访问**，此时代理附加的额外操作是检查调用者是否具有访问的权限；

- 智能代理（Smart Reference）：该代理取代了简单的指针，**在存在多个外部调用者时控制了对对象的访问**，此时代理附加的额外操作是对调用者个数进行计数（即记录对象的引用次数），在调用者访问对象时检查是否已经锁定了它，在调用者不进行调用时释放对象。

# 3、参考资料

- [http://www.cyc2018.xyz/其他/编码实践/面向对象思想.html](http://www.cyc2018.xyz/其它/编码实践/面向对象思想.html)

- https://design-patterns.readthedocs.io/zh_CN/latest/read_uml.html

- [http://www.cyc2018.xyz/其他/设计模式/设计模式.html](http://www.cyc2018.xyz/%E5%85%B6%E5%AE%83/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F%20-%20%E7%9B%AE%E5%BD%95.html#%E4%B8%80%E3%80%81%E5%89%8D%E8%A8%80)

- https://www.liaoxuefeng.com/wiki/1252599548343744/1264742167474528