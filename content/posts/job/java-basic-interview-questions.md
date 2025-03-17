---
title: "Java基础题面试笔记"
date: 2025-02-19T15:18:58+08:00
lastmod: 2025-02-19T15:18:58+08:00
author: ["SwimmingLiu"]

categories:
- 💻 Job

tags:
- Java

keywords:
- Java

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

## 1. 序列化和反序列化

1.序列化和反序列化：把对象转换为字节流，用于存储和传输；读取字节流数据，重新创建对象。
2.序列化不包括静态对象：序列化和反序列化的本质是调用对象的`writeObject`和`readObject`方法,来实现将对象写入输出流和读取输入流。但是，静态变量不属于对象，所以调用这两个方法就没法儿让静态变量参与。

## 2. 什么是不可变类？

1.不可变类：初始化之后，就不能修改的类。
2.修饰方法：final 和 private 修饰所有类和变量
3.不可修改：不暴露set方法，只能通过重新创建对象替代修改功能(`String`的replace方法)
4.优缺点：
优点：线程安全，缓存友好
缺点：频繁拼接和修改会浪费资源

## 3. Exception和Error区别?

1.Exception和Error定义区别：Exception是可处理程序异常，Error是系统级不可回复错误
2.try-catch建议：
    1.范围能小则小
    2.Exception最好要写清楚具体是哪一个Exception(IOException)
    3.null值等能用if判断的，不要用try-catch,因为异常比条件语句低效
    4.finally不要直接return和处理返回值

![Exception和Error区别](https://oss.swimmingliu.cn/946b73cc-ef5d-11ef-95ab-c858c0c1deba)

## 4. Java 中的 hashCode 和 equals 方法之间有什么关系？

1、`equals()` 和 `hashCode()` 的关系

- 如果两个对象`euqals()` 为 `true`， 则其 `hashCode()`一定相同
- 如果两个对象`hashCode()` 相同，其`equals()`结果不一定为`true`

2、为什么重写`equals()`之后，一定要重写`hashCode()`

当重写`equals()` 之后，通常是重新定义了两个对象相等的逻辑。如果不重写`hashCode()`方法， 则在散列集合（`HashMap` 和 `HashSet`）中，可能无法正确存储和检索，因为两个相同的对象可能有不同的`hash`值。

>例如，下方Person类重写了`equals()` 方法，但是没有重新`hashCode()`
>
>``` Java
>public class Person {
>private String name;
>private int age;
>
>@Override
>public boolean equals(Object obj) {
>   if (this == obj) return true;
>   if (obj == null || getClass() != obj.getClass()) return false;
>   Person person = (Person) obj;
>   return age == person.age && Objects.equals(name, person.name);
>}
>}
>```
>
>创建相同的对象，并添加到`HashSet`中
>
>```java
>Person p1 = new Person("Alice", 25);
>Person p2 = new Person("Alice", 25);
>Set<Person> set = new HashSet<>();
>set.add(p1);
>set.add(p2);
>```
>
>由于`hashCode`() 没有重写，所有两个相同的对象可能有不同的散列码，导致集合当中有两个相同的元素
>
>如何重写`hashCode()`?
>
>只需要让其hash值，采用`equals()`当中相同的判断条件生成合理的散列值即可
>
>```java
>@Override
>public int hashCode() {
>return Objects.hash(name, age);
>}
>```

## 5. 接口和抽象类有什么区别？

| 特性                   | 接口                                                         | 抽象类                                                       |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **设计角度**           | 自上而下，先定义好我们需要的方法，然后在具体的类当中，实现该接口的方法 | 自下而上，写了很多类之后，发现他们有共性，可以把代码复用。因此封装成一个抽象类，减少代码冗余 |
| **方法实现**           | 所有方法默认是 `public` 和 `abstract` 修饰 (`JDK 8` 之后可以设置`default`方法或者静态方法)，接口类可以是空的 (`Serializable` 序列化接口) | 可以包含`abstract` 抽象方法 (无实现) 和 具体方法(有实现)，至少包含一个抽象方法 |
| **构造函数和成员变量** | 接口不能包含构造函数，所有成员变量默认为 `public static final` 常量 | 抽象类可以包含构造函数，成员变量可以有不同的修饰符，不一定是常量 |
| **多继承**             | 一个类只能实现多个接口                                       | 抽象类只能单继承                                             |

**【注意】** 

1. 接口和抽象类是不可以被实例化的，只能用来实现或者继承。

2. `JDK 9` 之后，接口可以定义私有方法，用于`default` 方法的内部逻辑复用

   ```java
   // 支付接口
   interface PaymentInterface {
       // 支付方法
       void pay(double amount);
       // 通知支付状态
       void notifyPaymentStatus(String status);
       // 私有方法，用于检查支付金额是否合法
       private boolean isAmountValid(double amount) {
           return amount > 0;
       }
       // default 方法，包含了对私有方法的调用
       default void performPayment(double amount) {
           if (isAmountValid(amount)) {
               pay(amount);
           } else {
               System.out.println("Invalid payment amount");
           }
       }
   }
   ```

3. 接口当中的 `default` 方法，实现对象可以直接调用，也可以进行重写。`static` 方法只能通过接口名调用

## 6. JDK 动态代理和 CGLIB 动态代理有什么区别？

- **JDK 动态代理**：基于反射机制和接口，要求所有代理类都必须实现某个接口 (目标对象至少实现一个接口)
- **CGLIB**：基于 `ASM` 字节码生成工具，通过继承的方式生成目标类的子类来实现代理，所以要注意 `final` 方法 （子类可以继承并使用父类的 `final` 方法，但不能重写（Override）该方法）

| 特性                | JDK 动态代理                                | CGLIB                                                 |
| ------------------- | ------------------------------------------- | ----------------------------------------------------- |
| **底层实现**        | 接口 + 反射，通过反射调用目标对象的接口方法 | 生成子类的字节码 + 重写被代理类的方法                 |
| **代理对象**        | 必须实现接口                                | 不需要实现接口                                        |
| **性能**            | 创建代理开销小，方法调用开销大              | 方法调用开销小，创建代理开销大 (需要生成子类的字节码) |
| **限制**            | 不能代理没有接口的类                        | 不能代理 `final` 类 和 `final` 方法                   |
| **使用场景**        | 需要实现接口的类                            | 没有接口的类                                          |
| Spring AOP 默认方式 | AOP 默认采用JDK动态代理                     | 目标类没有接口的时候，采用 `CGLIB`                    |

【注意】在不同的`jdk`版本下，`JDK` 动态代理 和 `CGLIB` 的性能都不一样 (可参考 [CGLIB与JDK动态代理的运行性能比较](https://www.cnblogs.com/haiq/p/4304615.html))

>- JDK 1.6：运行次数少的情况下，`JDK`动态代理和`CGLIB` 基本没差，甚至`JDK`动态代理更快。当次数增加之后，`CGLIB` 会稍微快一些
>- JDK 1.7: 基本都是 `JDK`动态代理比较快，运行次数较少的情况下，`JDK`动态代理比`CGLIB` 快 `30%` 左右。当次数增加之后，`JDK`动态代理比`CGLIB` 快了接近一倍
>- JDK 1.8：和 `JDK 1.7` 表现基本一致

**【JDK 动态代理详解】**

`JDK` 动态代理是通过反射机制，来实现代理接口中的方法的。通过 `java.lang.reflect.Proxy` 类 和 `InvocationHandler` 接口来实现代理， 代理对象只代理接口中的方法。当调用代理对象的方法的时候，代理类会拦截方法调用，然后通过 `InvocationHandler.invoke()` 方法执行额外的逻辑。 (`AOP` 切面编程的原理)

![JDK动态代理结构图](https://oss.swimmingliu.cn/bd651ce8-0332-11f0-951a-c858c0c1debd)

例如，下面代码中，`DataQuery` 是接口， `DatabaseDataQuery` 实现该接口。`JDK` 动态代理要求被代理对象至少实现一个接口，以便代理类通过接口暴露代理行为。
通过 `Proxy.newProxyInstance()` 创建代理对象，需要三个参数：

- **类加载器**：`Thread.currentThread().getContextClassLoader()`， 用于加载代理类
- **接口列表**：`new Class[]{xxx.class}`， 指定代理类应该实现的接口, 里面存放接口的 `class`
- **调用处理器** `InvocationHandler`：代理对象的实际逻辑是用实现这个`InvocationHandler`接口的类来晚来成的，比如`CacheInvocationHandler` 拦截代理方法，调用并添加额外逻辑。底层的实现逻辑其实就是反射机制。

```java
// 接口
public interface DataQuery {
    String query(String queryKey);
    String queryAll(String queryKey);
}
// 目标实现类
public class DatabaseDataQuery implements DataQuery {
    @Override
    public String query(String queryKey) {
        // 他会使用数据源从数据库查询数据很慢
        System.out.println("正在从数据库查询数据");
        return "result";
    }
    @Override
    public String queryAll(String queryKey) {
        // 他会使用数据源从数据库查询数据很慢
        System.out.println("正在从数据库查询数据");
        return "all result";
    }
}
// 调用处理器：用于拦截方法
public class CacheInvocationHandler implements InvocationHandler {
    private HashMap<String,String> cache = new LinkedHashMap<>(256);
    private DataQuery databaseDataQuery;
    public CacheInvocationHandler(DatabaseDataQuery databaseDataQuery) {
        this.databaseDataQuery = databaseDataQuery;
    }
    public CacheInvocationHandler() {
        this.databaseDataQuery = new DatabaseDataQuery();
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 1、判断是哪一个方法
        String result = null;
        if("query".equals(method.getName())){
            // 2、查询缓存，命中直接返回
            result = cache.get(args[0].toString());
            if(result != null){
                System.out.println("数据从缓存重获取。");
                return result;
            }
            // 3、未命中，查数据库（需要代理实例）
            result = (String) method.invoke(databaseDataQuery, args);
            // 4、如果查询到了,进行呢缓存
            cache.put(args[0].toString(),result);
            return result;
        }
        // 当其他的方法被调用，不希望被干预，直接调用原生的方法
        return method.invoke(databaseDataQuery,args);
    }
}
// 测试类：如何使用JDK动态代理
public class Main {
    public static void main(String[] args) {
        // jdk提供的代理实现，主要是使用Proxy类来完成
        // 1、classLoader：被代理类的类加载器，用于加载代理类
        ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
        // 2、代理类需要实现的接口数组
        Class[] classes = new Class[]{DataQuery.class};
        // 3、InvocationHandler
        CacheInvocationHandler cacheInvocationHandler = new CacheInvocationHandler();
        DataQuery dataQuery  = (DataQuery) Proxy.newProxyInstance(contextClassLoader, classes, cacheInvocationHandler);
        // 事实上调用query方法的使用，他是调用了invoke
        String result = dataQuery.query("key1");
        System.out.println(result);
        System.out.println("--------------------");
        result = dataQuery.query("key1");
        System.out.println(result);
        System.out.println("--------------------");
        result = dataQuery.query("key2");
        System.out.println(result);
        System.out.println("++++++++++++++++++++++++++++++++++++");
        // 事实上调用queryAll方法的使用，他是调用了invoke
        result = dataQuery.queryAll("key1");
        System.out.println(result);
        System.out.println("--------------------");
        result = dataQuery.queryAll("key1");
        System.out.println(result);
        System.out.println("--------------------");
        result = dataQuery.queryAll("key2");
        System.out.println(result);
        System.out.println("--------------------");
    }
}
```

**【CGLIB 动态代理详解】**

`CGLIB` (Code Generation Library) 代码生成库 是基于字节码操作的，它可以生成目标类的子类，并且重写目标类的方法来实现代理。通过继承方式拦截所有非 `final` 方法的调用。 `CGLIB` 使用的是 `ASM` 字节码生成框架，生成的是字节码级别的代理类。性能相对较好，但是生成代理类的开销比 `JDK` 动态代理 稍微大一些。

![CGLIB动态代理类结构图](https://oss.swimmingliu.cn/bf8d4fef-0332-11f0-b046-c858c0c1debd)

`Main` 类当中，没有实现任何接口，这就是 `CGLIB` 的优势之一，不需要实现任何接口。 `CGLIB` 通过生成目标类的子类来实现代理。`CGLIB` 通过 `Enhancer` 类来创建代理对象，需要配置父类和拦截器。

- **父类**：`enhancer.setSuperclass(DatabaseDataQuery.class)` ，配置目标类作为父类
- **拦截器**：`enhancer.setCallback(new CacheMethodInterceptor());` ，配置拦截器，拦截所有非 `final` 方法的调用，和`InvocationHandler` 差不多，可以插入额外的逻辑。

```java
// 拦截器：用于拦截方法
public class CacheMethodInterceptor implements MethodInterceptor {
    private HashMap<String,String> cache = new HashMap<>();
    private DatabaseDataQuery databaseDataQuery;
    public CacheMethodInterceptor( ) {
        this.databaseDataQuery = new DatabaseDataQuery();
    };
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        // 1、判断是哪一个方法
        String result = null;
        if("query".equals(method.getName())){
            // 2、查询缓存，命中直接返回
            result = cache.get(args[0].toString());
            if(result != null){
                System.out.println("数据从缓存重获取。");
                return result;
            }
            // 3、未命中，查数据库（需要代理实例）
            result = (String) method.invoke(databaseDataQuery, args);
            // 4、如果查询到了,进行呢缓存
            cache.put(args[0].toString(),result);
            return result;
        }
        return method.invoke(databaseDataQuery,args);
    }
}
// 测试类：如何使用CGLIB进行动态代理
public class Main {
    public static void main(String[] args) {
        //cglib通过Enhancer实现
        Enhancer enhancer = new Enhancer();
        //设置父类
        enhancer.setSuperclass(DatabaseDataQuery.class);
        //设置一个拦截器，用来拦截方法
        enhancer.setCallback(new CacheMethodInterceptor());
        //创建代理类，其实就是目标类的子类
        DatabaseDataQuery databaseDataQuery = (DatabaseDataQuery) enhancer.create();
        databaseDataQuery.query("Key1");
        databaseDataQuery.query("Key1");
        databaseDataQuery.query("Key2");
    }
}
```

## 7. 你使用过 Java 的反射机制吗？如何应用反射？

- **反射机制定义**：Java 的反射机制是指在运行的时候，获取类的结构信息 (比如方法、字段、构建函数) ，然后获取操作对象的一种机制。反射机制可以在运行的时候，动态创建对象、调用方法、访问字段，不需要在编译的时候知道这些信息。反射的核心类包括 `Class`、`Constuctor`、`Method`、`Filed`

- **反射机制作用**：

  - **动态获取类信息**：包括类名、包名、父类等，不需要在编译的时候知道类的信息
  - **动态创建对象**：可以通过 `Class` 类或者 `Constructor` 对象的 `newInstance()` 动态创建对象，不需要在编译的时候知道对象的类型
  - **动态调用对象的方法**：通过 `Method` 类的 `invoke()` 方法实现
  - **访问和修改对象的字段值**：通过 `Filed.set()` 直接修改对象的值，和通过`SetAccessible(true)` 绕过访问限制

- **反射机制的应用场景**： 一般的业务编码用不到反射机制，但是在框架上会用到反射机制。因为写框架的时候，很多场景是很灵活的，不能确定目标对象的类型，只能通过反射动态获取对象信息。比如 `Spring` 使用反射机制来读取和解析配置文件，从而实现 `DI` 依赖注入和 `AOP` 切面编程等功能。

  `DI` 依赖注入的具体实现，假如对指定类加上 `@Service` 注解，下面是具体的实现过程：

  1. `Spring` 在容器启动初始化的时候, 将所有带有`@Service`、`@Component`等注解的类，放入容器当中，注册为`Bean`
  2. `Spring` 容器启动的时候，会将带有`@Autowired` 注解的类进行标记，然后使用反射机制将所有的字段、方法、构造器进行注入

- **反射的使用方法**：

  - **获取 `Class` 对象**

    ```java
    Class<?> clazz = Class.forName("com.swimmingliu.MyClass");
    // 或者
    Class<?> clazz = MyClass.class;
    // 或者
    Class<?> clazz = obj.getClass();
    ```

  - **创建对象** ：一般都采用 `Constructor` 来构造对象

    ```java
    Object obj = clazz.newInstance(); // 已过时
    Constructor<?> constructor = clazz.getConstructor(); 
    Object obj = constructor.newInstance();
    ```

  - **访问字段**：可以让私有对象可见，而且能够设置它的值

    ```java
    Field field = clazz.getField("name"); // 假设 private String name;
    field.setAccessible(true); // 允许访问 private 字段
    Object value = field.get(obj);
    field.set(obj, newValue);
    ```

  - **调用方法**：

    ```java
    Method method = clazz.getMethod("myMethod", String.class);
    Object result = method.invoke(obj, "param");
    ```

    