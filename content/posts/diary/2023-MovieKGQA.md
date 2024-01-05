---
title: "MovieKGQA: 基于知识图谱和neo4j图数据库的电影知识问答系统"
date: 2023-12-12T12:18:57+08:00
lastmod: 2023-12-12T12:18:57+08:00
author: ["SwimmingLiu"]

categories:
- 📓 Diary

tags:
- KGQA
- Wechat Mini Programs

keywords:
- Mini Programs
- Movie KGQA

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
## Introduction

基于知识图谱和neo4j图数据库的电影知识问答系统
<div style="display:flex; justify-content: space-around; ">
<img src="https://i.imgs.ovh/2023/12/12/mM4uR.png" alt="image-20231212102658908" style="box-shadow: 0 0 10px rgba(200, 200, 200);" width=30% height:300px/>
<img src="https://i.imgs.ovh/2023/12/12/mM58p.png" alt="image-20231212102738360" style="box-shadow: 0 0 10px rgba(200, 200, 200);" width=30% height:300px/>
<img src="https://i.imgs.ovh/2023/12/12/mMdFT.png" alt="image-20231212103113278" style="" width=30% height:300px/>
</div>


## Workflow

### DataBase

   爬取豆瓣TOP1000电影信息数据

### Frontend

1. 获取用户输入的信息 （语音输入 / 文本输入）
2. 向电影知识问答后端服务器发送请求
3. 获取返回结果  (成功 -> 4 / 失败 -> 5)
4. 如果返回结果包含image信息，则显示图片和文字，否则只显示文字
5. 请求基于gpt的AI模型服务器，并显示返回结果

### Backend

​	[准备工作]  训练 TF-IDF 向量算法和朴素贝叶斯分类器，用于预测用户文本所属的问题类别

1. 接受前端请求，获取用户输入信息
2. 使用分词库解析用户输入的文本词性，提取关键词
3. 根据贝叶斯分类器，分类出用户文本的问题类型
4. 结合关键词与问题类别，在 Neo4j 中查询问题的答案
5. 返回查询结果 （若问题类型为 演员信息 / 电影介绍，则附加图片url）

### WorkFlow Graph

![workflow graph](https://oss.swimmingliu.cn/0IEuW.png)

## Frame

### DataBase

[![Neo4j](https://img.shields.io/badge/neo4j-test?style=for-the-badge&logo=neo4j&logoColor=white&color=blue)](https://neo4j.com/)

### Frontend

[![wechat mini programs](https://img.shields.io/badge/wechat%20mini%20programs-test?style=for-the-badge&logo=wechat&logoColor=white&color=%2320B2AA)](https://developers.weixin.qq.com/)

### Backend

[![Python](https://img.shields.io/badge/python-3776ab?style=for-the-badge&logo=python&logoColor=ffd343)](https://www.python.org/)[![Flask](https://img.shields.io/badge/flask-3e4349?style=for-the-badge&logo=flask&logoColor=ffffff)](https://flask.palletsprojects.com/)[![Scikit-learn](https://img.shields.io/badge/sklearn-test?style=for-the-badge&logo=scikit-learn&logoColor=white&color=orange)](https://scikit-learn.org/stable/index.html)[![Jieba](https://img.shields.io/badge/jieba-3776ab?style=for-the-badge&logo=python&logoColor=ffd343)](https://github.com/fxsjy/jieba)

## Reference

### Frontend

[微信小程序：微信聊天机器人](https://github.com/JzheTang/wechat_robot_app)

### BackEnd

[基于知识图谱的电影知识问答系统](https://github.com/mrcaidev/kgqa)

[电影知识库问答机器人](https://github.com/futurehear/chatbot)
