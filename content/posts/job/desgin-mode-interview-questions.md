---
title: "设计模式面试题笔记"
date: 2025-03-07T21:44:36+08:00
lastmod: 2025-03-07T21:44:36+08:00
author: ["SwimmingLiu"]

categories:
- 💻 Job

tags:
- Java
- Design Mode

keywords:
- Java
- Design Mode

description: "" # 文章描述，与搜索优化相关
summary: "" # 文章简单描述，会展示在主页
weight: # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
slug: ""
math: true
draft: false # 是否为草稿
comments: true
showToc: true # 显示目录
TocOpen: false # 自动展开目录
autonumbering: true # 目录自动编号
hidemeta: false # 是否隐藏文章的元信息，如发布日期、作者等
disableShare: true # 底部不显示分享栏
searchHidden: false # 该页面可以被搜索到
showbreadcrumbs: true #顶部显示当前路径
mermaid: true
cover:
    image: ""
    caption: ""
    alt: ""
    relative: false
---


## 1. 单例模式有哪几种实现？如何保证线程安全？

首先，单例模式和工厂模式都是一种设计模式。单例模式当中，一个类只允许创建一个对象(或者说实例)， 那这个类就是单例类。单例类是不可以被继承的，也没有了多态的特性。

**【单例类的实现方式】**

常规单例模式有五种写法，但是编写代码的过程当中，要注意以下几点：

1. 构造器需要私有化
2. 暴露一个公共获取单例对象的接口 （`obj.getInstance()`）
3. 是否支持懒加载 `延迟加载`
4. 是否线程安全

五种写法为：

1. **饿汉式**： 类加载的时候，就一起把 `instance` 静态实例创建好了，所以创建的过程市线程安全的。

   饿汉式的单例模式虽然不支持懒加载，有点浪费资源。但其实不会占用太多资源，并且如果一个实例初始化的过程比较复杂，就应该放在启动的时候来处理，避免运行时卡顿或发生问题， 满足`fail-fast` 失败快速解决的设计原则

   ``` java
   public class EagerSingleton {  
       private static Singleton instance = new Singleton();  
       private Singleton (){}  
       public static Singleton getInstance() {  
       	return instance;  
       }  
   }
   ```

2. **懒汉式**：相较于饿汉式的方式，修改成延迟加载的模式。注意`getInstance()`方法没有上锁的话，在大量线程并发请求的时候，可能创建多个实例。

   ```java
   public class Singleton {  
       private static Singleton instance;  
       private Singleton (){}  
       public synchronized static Singleton getInstance() {  
           if (instance == null) {  
               instance = new Singleton();  
           }  
           return instance;  
       }  
   }
   ```

3. **双重检查锁**：饿汉式锁不支持延迟加载，然后懒汉式锁的粒度比较大，不支持高并发。双重检查锁可以实现既延迟加载，又支持高并发。其实就是在判断了没有实例之后，再进行上锁，创建实例。 但是实例必须用`volatile` 修饰，不然`new` 操作创建对象时，容易出现重排序的问题。

   ```java
   public class DclSingleton {  
       // volatile如果不加可能会出现半初始化的对象
       // 现在用的高版本的 Java 已经在 JDK 内部实现中解决了这个问题（解决的方法很简单，只要把对象 new 操作和初始化操作设计为原子操作，就自然能禁止重排序）,为了兼容性我们加上
       private volatile static Singleton singleton;  
       private Singleton (){}  
       public static Singleton getInstance() {  
           if (singleton == null) {  
               synchronized (Singleton.class) {  
                   if (singleton == null) {  
                       singleton = new Singleton();  
                   }  
               }  
           }  
           return singleton;  
       }  
   }
   ```

4. **静态内部类**：利用Java的内部类，再调用`getInstance()`方法的时候，直接返回内部类的实例。他会再调用方法之后，创建内部类的实例对象。实例的唯一性和创建过程的线程安全性，都有JVM来保证。这种方法既是线程安全的，又能够做延迟加载。

   ```java
   public class InnerSingleton {
       /** 私有化构造器 */
       private Singleton() {
       }
       /** 对外提供公共的访问方法 */
       public static Singleton getInstance() {
           return SingletonHolder.INSTANCE;
       }
       /** 写一个静态内部类，里面实例化外部类 */
       private static class SingletonHolder {
           private static final Singleton INSTANCE = new Singleton();
       }
   }
   ```

5. **枚举**：通过Java枚举类型本身的特性，保证实例创建线程的安全性和实例的唯一性。

   ```java
   // 使用枚举实现单例模式
   public enum Singleton {
       INSTANCE;
       // 单例中的方法示例
       public void doSomething() {
           System.out.println("单例方法执行");
       }
   }
   // 使用方法：
   Singleton.INSTANCE.doSomething();
   ```

   也可以用单例项作为枚举的成员变量，累加器可以像下面这样编写：

   ``` java
   public enum GlobalCounter {
       INSTANCE;
       private AtomicLong atomicLong = new AtomicLong(0);
   
       public long getNumber() { 
           return atomicLong.incrementAndGet();
       }
   }
   ```

**【单例模式的安全问题】**

1. **反射入侵**：如果想要阻止其他人构造实例，仅仅私有化构造器还是不够的，因为我们可以利用反射机制来获取私有构造器进行构造。如果要避免这种情况发生，可以再构造器当中进行判断实例是否已存在，避免多次利用构造器构造实例

   ```java
   public class Singleton {
       private volatile static Singleton singleton;
       private Singleton (){
           if(singleton != null) 
               throw new RuntimeException("实例：【"
                       + this.getClass().getName() + "】已经存在，该实例只允许实例化一次");
       }
   
       public static Singleton getInstance() {
           if (singleton == null) {
               synchronized (Singleton.class) {
                   if (singleton == null) {
                       singleton = new Singleton();
                   }
               }
           }
           return singleton;
       }
   }
   // 利用反射机制来入侵构造实例
   @Test
   public void testReflect() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
       Class<DclSingleton> clazz = DclSingleton.class;
       Constructor<DclSingleton> constructor = clazz.getDeclaredConstructor();
       constructor.setAccessible(true);
   
       boolean flag = DclSingleton.getInstance() == constructor.newInstance();
   
       log.info("flag -> {}",flag);
   }
   ```

2. **序列化与反序列化**：如果单例存到文件流当中，再进行反序列话，也不是同一个实例。但是可以用`readResolve()`的方法，将返回值作为反序列化的结果，而不会克隆一个新的实例，保证jvm当中只有一个实例存在。

   ```java
   @Test
   public void testSerialize() throws IllegalAccessException, NoSuchMethodException, IOException, ClassNotFoundException {
       // 获取单例并序列化
       Singleton singleton = Singleton.getInstance();
       FileOutputStream fout = new FileOutputStream("D://singleton.txt");
       ObjectOutputStream out = new ObjectOutputStream(fout);
       out.writeObject(singleton);
       // 将实例反序列化出来
       FileInputStream fin = new FileInputStream("D://singleton.txt");
       ObjectInputStream in = new ObjectInputStream(fin);
       Object o = in.readObject();
       log.info("他们是同一个实例吗？{}",o == singleton); // 如果直接获取，反序列化后的不是同一个实例
   }
   // 添加readResolve()来解决序列化和反序列化问题
   public class Singleton implements Serializable {
       // 省略其他的内容
       public static Singleton getInstance() { 
       }
       // 需要加这么一个方法
       public Object readResolve(){
           return singleton;
       }
   }
   ```

**【为什么要用单例模式】**

1. **为了全局唯一**：系统中如配置类、全局计数器等类型，应该都都只能保存一份数据，不应该有多份数据。

   - 配置类：系统仅有一个配置文件，加载到内存后映射成唯一的配置实例
   - 全局计数器：用于数据统计、生成全局递增`id` 等功能，必须要是唯一的，否则可能导致统计无效、`ID` 重复等问题。

   ``` java
   // 全局id生成器
   public class GlobalCounter {
       private AtomicLong atomicLong = new AtomicLong(0);
       private static final GlobalCounter instance = new GlobalCounter();
       // 私有化无参构造器
       private GlobalCounter() {}
       public static GlobalCounter getInstance() {
           return instance;
       }
       public long getId() { 
           return atomicLong.incrementAndGet();
       }
   }
   // 查看当前的统计数量
   long courrentNumber = GlobalCounter.getInstance().getId();
   ```

2. **处理资源访问冲突**：假如需要日志输出的功能，可以使用单例i面资源访问冲突

   ```java
   public class Logger {
       private String basePath = "D://log/";
       private static Logger instance = new Logger(); 
       private FileWriter writer;
       private Logger() {
           File file = new File(basePath);
           try {
               writer = new FileWriter(file, true); //true表示追加写入
           } catch (IOException e) {
               throw new RuntimeException(e);
           }
       }
       public static Logger getInstance(){ //确保全局只有一个logger实例对象
           return instance;
       }
       public void log(String message) {
           try {
               writer.write(message);
           } catch (IOException e) {
               throw new RuntimeException(e);
           }
       }
       public void setBasePath(String basePath) {
           this.basePath = basePath;
       }
   }
   ```

**【单例模式存在的问题】**

1. **无法支持面向对象编程`OOP`**： `OOP` 的三大特性是**封装、继承、多态**。单例把构造函数私有化了，不支持继承和多态。所以无法对它进行拓展。
2. **很难横向拓展**：单例类只能有一个对象实例，如果后面需要进行拓展，创建多个实例。必须修改源码，无法友好拓展。

**【不同作用范围的单例模式】**

1. 线程级别单例：单例类对象是进程唯一的，如果想要线程唯一。在不使用`ThreadLocal`的时候，可以采用`ConCurrentHashMap` 的方式，用线程`id`为`key`， 实例为`value`。每个线程的存取都从共享的 `map` 当中进行操作。

   ```java
   public class Connection {
       private static final ConcurrentHashMap<Long, Connection> instances
               = new ConcurrentHashMap<>();
       private Connection() {}
       public static Connection getInstance() {
           Long currentThreadId = Thread.currentThread().getId();
           instances.putIfAbsent(currentThreadId, new Connection());
           return instances.get(currentThreadId);
       }
   }
   ```

2. **容器级别的单例**：将单例的作用范围由进程切换到一个容器，可能会更加方便我们进行单例对象的管理。这也是`Spring` 的核心思想。`Spring` 提供一个单例容器，确保一个实例是容器级别的单例，并且在容器启动时完成初始化。具体优势如下：

   - 所有的`bean` 都以单例的形式存放在容器中，避免大量的对象被创建，造成`JVM` 内存抖动严重，频繁`GC`。
   - 程序启动时，初始化单例`bean`， 满足`fast-fail`，将所有构建过程的异常暴露在启动时，而非运行时。
   - 缓存了所有单例`bean`，启动的过程相当于预热的过程，运行时不必进行对象创建，效率更高。
   - 容器管理`bean`的生命周期，结合依赖注入使得解耦更加彻底、扩展性更好。

## 2. 什么是策略模式？一般用在什么场景？

策略模式是行为设计模式的一种，通过定义一系列的算法类。允许在运行时动态选择算法，从而实现更加灵活的代码结构。该模式用于组织和调用这些算法，让程序结构变得更加灵活，具有更好的维护性和扩展性。

策略模式一般用于当一个功能存在多种算法的时候，需要根据不同的情况使用不同的计算算法(都封装成类的)。这样就可以避免利用大量的`if-else` 或者 `switch-else`

**【为什么要用策略模式】**

- **避免程序存在判断或选择分支语句**：当程序存在大量的 `if-else` 或者 `switch-else` 判断语句，代码可能变得难以维护
- **避免破坏现有功能**：当算法的实现经常变更或需要拓展的时候，直接修改代码可能会破坏现有功能。

**【策略模式的场景】**

- **多种算法可互换**：需要动态选择算法，例如排序算法的选择。有很多种排序算法，可以把不同的排序方式封装成一个独立的算法类 (快速排序、归并排序、直接插入排序等)
- **避免条件语句**：采用策略模式替换掉代码当中的大量`if-else` 或 `switch` 语句
- **与上下文独立**：客户端不需要知道具体的实现细节，只需以来抽象策略。

**【策略模式典型应用场景】**

- **支付系统**：支持多种支付方式，比如微信、支付宝、信用卡
- **数据压缩**：提供不同的压缩算法
- **日志策略**：根据日志级别动态选择记录策略

**【策略模式的组成】**

- **`Strategy` 策略**：用来约束一系列具体的策略算法。`Context` 上下文使用这个接口来调用具体的策略实现定义的算法。如果多个算法具有公共功能的化，把`Strategy` 实现为抽象类，然后把多个算法的功能实现到`Stragy` 里面。 (比如多种排序算法，都放在 `Strategy` 抽象类里面)
- **`ConcreteStrategy` 具体策略**： 具体的策略实现，负责实现`Strategy` 策略的接口 (多种排序算法的具体实现)
- **`Context` 上下文**：上下文是负责和具体的策略类交互，通常上下文会吃有一个真正的策略实现 (就是调用哪个排序方法，比如说 `main` 函数)

![策略模式的组成](https://oss.swimmingliu.cn/f649bf1f-fb59-11ef-8f2a-c858c0c1deba)

## 3. 什么是模板方法模式？一般用在什么场景？

模板方法就是在抽象类里面定义好算法的骨架，具体步骤在子类实现。

**【模板方法特点】**

1. **算法骨架**：在基类中定义一个算法的固定执行步骤，具体实现步骤交给子类实现
2. **复用代码**：子类复用基类中定义的公共逻辑，只需要实现特定的逻辑
3. **遵循开闭规则**：模板方法是扩展开放，修改闭合的

**【典型使用场景】**

- **数据请求处理**: 读取数据、处理数据、输出结果
- **Web请求处理**：解析请求、处理逻辑、返回响应
- **`String Template` 字符串模板**

``` java
String name = "World";
String greeting = STR."Hello, \{name}!";
System.out.println(greeting); // 输出: Hello, World!
```

## 4. 谈谈你了解的最常见的几种设计模式，说说他们的应用场景

**【常见的设计模式】**

- **单例模式**：保证系统中一个类只有一个实例对象，比如全局配置、全局计数器、数据库连接池
- **策略模式**：封装一组算法让他们之间能够相互替代，避免大量的`if-else` 和 `switch-case` 语句，比如用户选择不同的支付策略，或者调用不同的排序算法。
- **模板模式**：提炼核心流程封装成一个方法，比如像支付逻辑(参数校验、调起支付接口、修改支付状态)，除了调起支付接口以外，其他的流程基本一致。所以可以封装成模板方法，然后把调起支付接口的操作，在具体实现方法当中重写该方法。
- **简单工厂模式**：获取不同的对象时可以使用，将对象的创建逻辑抽离复用。
- **外观模式**：为子系统提供一组统一接口，隐藏内部实现细节，方便子系统直接调用，而无需关注实现细节(比如高德和百度的 `SDK`)
- **代理模式**：通过创建代理对象来控制哦对实际对象的访问，例如`Sping AOP`切面编程采用代理模式来动态生成增强目标对象的代理 (通过 `JDK` 动态代理或者`CGLIB`代理) 。`Sping AOP` 默认优先使用 JDK 动态代理。当目标类未实现接口时，才会切换为 CGLIB 动态代理。

**【Spring中的设计模式】**

![Spring中的设计模式](https://oss.swimmingliu.cn/f6a094ff-fb59-11ef-b5d7-c858c0c1deba)

## 5. 你认为好的代码应该是什么样的？

**【通俗易懂的讲】** 

- **清晰易懂，保持简洁和易读性**：代码简洁、直观，函数和变量命名都有意义，可读性非常强，有合理的注释。
- **高内聚低耦合**：高内聚是指代码模块内部功能集中，每个模块都有单一职责。低耦合指的是不同模块之间的依赖关系尽量松散，修改一个模块的时候，其他模块受到的影响比较小。
- **可测试**：每个模块都设计为独立的、可验证的单元，编写单元测试用例的时候，能够确保代码的正确性并且发现潜在问题。
- **易于扩展，遵循开闭原则**：代码具有一定灵活性，能在不破坏现有功能的情况下，方便进行扩展和修改。
- **符合团队规范**：代码风格和团队整体代码规范统一，有助于协助团队协作和代码审查。
- **减少硬编码、魔法值**：尽量避免硬编码和魔法值的现象，方便后续进行修改。

**【好代码具备以下特性】**

| 设计原则     | 代码特性                   | 结果                                   |
| ------------ | -------------------------- | -------------------------------------- |
| 单一职责     | 类或模块职责单一           | 降低类的复杂度，增强可读性和维护性     |
| 开闭原则     | 对扩展开放，对修改关闭     | 提高扩展性，减少对已有代码的修改       |
| 高内聚低耦合 | 模块职责明确，依赖关系松散 | 增强代码的可维护性和扩展性             |
| 接口隔离原则 | 接口小且专用               | 减少无关实现代码，增强接口的灵活性     |
| 依赖倒置     | 依赖于抽象而非具体实现     | 降低模块间的耦合度，提高代码灵活性     |
| 合成复用原则 | 优先使用组合而非继承       | 提高灵活性，避免继承导致的高聚合       |
| 里氏替换原则 | 子类可以无缝替换父类       | 确保继承体系的正确性，增强代码的稳定性 |
| 迪米特法则   | 减少类之间的依赖           | 降低耦合度，增强模块独立性             |

## 6. 工厂模式和抽象工厂模式有什么区别？

**【工厂模式】**

- **对象**：创建一种类型的产品对象，比如让google创建安卓系统， 不同品牌的手机厂商(工厂子类)就可以根据安卓系统当中的功能，重写出其他的`os` 系统，比如鸿蒙系统、澎湃系统等等
- **工厂结构**：有且只有一个抽象的工厂类，定义创建产品的抽象方法，然后具体的工厂子类去实现这个方法来实际创建具体的产品。比如 `android` 系统的源码是大家都可以看到的，其他厂商也可以根据`android` 系统的设计，重新实现部分函数，修改成其他的功能。
- **使用场景**：当创建过程比较复杂，想把对象创建和使用分离时常用，比如创建数据库连接对象等简单的单一产品创建创建适用。

**【抽象工厂模式】**

- **对象**：用于创建一系列相关的产品对象，比如创建手机的时候，需要连带创建配套的充电器、耳机等配套产品
- **工厂结构**：抽象工厂类定义了多个抽象创建方法，分别用于创建一系列相关的产品。更具体的工厂子类要实现这些抽象方法，提供一整套的具体产品的创建。
- **使用场景**：当系统要创建多个相互依赖或者关联的对象的时候，确保这些对象搭配合理。比如游戏开发中创建不同风格(比如科技风格、古风)的角色、武器、场景等一整套相关的元素时，就适合用抽象工厂模式。

```java
// 抽象产品A
public interface ProductA {
    void use();
}

// 具体产品A1 -> 手机
public class ConcreteProductA1 implements ProductA {
    @Override
    public void use() {
        System.out.println("Using ConcreteProductA1");
    }
}

// 具体产品A2 -> 电脑
public class ConcreteProductA2 implements ProductA {
    @Override
    public void use() {
        System.out.println("Using ConcreteProductA2");
    }
}

// 抽象产品B
public interface ProductB {
    void eat();
}

// 具体产品B1 -> 手机充电线
public class ConcreteProductB1 implements ProductB {
    @Override
    public void eat() {
        System.out.println("Eating ConcreteProductB1");
    }
}

// 具体产品B2 -> 电脑充电线
public class ConcreteProductB2 implements ProductB {
    @Override
    public void eat() {
        System.out.println("Eating ConcreteProductB2");
    }
}

// 抽象工厂 -> 把两个相互关联的抽象产品一起创建
public interface AbstractFactory {
    ProductA createProductA();
    ProductB createProductB();
}
// 具体工厂1
public class ConcreteFactory1 implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA1();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB1();
    }
}
// 具体工厂2
public class ConcreteFactory2 implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ConcreteProductA2();
    }

    @Override
    public ProductB createProductB() {
        return new ConcreteProductB2();
    }
}
// 使用抽象工厂创建产品
public class Client {
    public static void main(String[] args) {
        AbstractFactory factory1 = new ConcreteFactory1();
        ProductA productA1 = factory1.createProductA();
        ProductB productB1 = factory1.createProductB();
        productA1.use();
        productB1.eat();

        AbstractFactory factory2 = new ConcreteFactory2();
        ProductA productA2 = factory2.createProductA();
        ProductB productB2 = factory2.createProductB();
        productA2.use();
        productB2.eat();
    }
}
```

