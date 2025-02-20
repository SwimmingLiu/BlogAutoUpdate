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

