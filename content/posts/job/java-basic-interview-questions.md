---
title: "Java基础题面试"
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

![Exception和Error区别](https://pic.code-nav.cn/mianshiya/question_picture/1814905001808924674/j0JcqCQh_image_mianshiya.png)

## 4. Java有什么优势?

