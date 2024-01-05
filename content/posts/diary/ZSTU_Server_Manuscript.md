---
title: "ZSTU服务器使用教程 (Yang Li Lab)"
date: 2024-01-05T13:47:03+08:00
lastmod: 2024-01-05T13:47:03+08:00
author: ["SwimmingLiu"]

categories:
- 📓 Diary

tags:
- Server
- ZSTU

keywords:
- Server Management

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

## 安装 Xshell 和 Xftp

```bash
https://www.netsarang.com/en/xshell-download/ # Xshell下载连接
https://blog.csdn.net/m0_67400972/article/details/125346023 # 安装教程
```

## 添加Xshell连接

![image-20240105113129928](E:\李杨课题组\论文笔记\image-20240105113129928.png)

其中 `server.ip` 为服务器公网ip地址，端口为 `6969` 

## 安装Anaconda3

每个用户均被分配 `AnacondaAnaconda3-2023.07-1-Linux-x86_64.sh ` 于主目录

```bash
bash AnacondaAnaconda3-2023.07-1-Linux-x86_64.sh # 安装anaconda3
```

![image-20240105113942737](E:\李杨课题组\论文笔记\image-20240105113942737.png)

输入 `yes` 后， 再按回车键 即可

![image-20240105114146047](E:\李杨课题组\论文笔记\image-20240105114146047.png)

## 初始化Anaconda3

```bash
conda init bash	# 初始化conda
```

然后重新使用Xshell 连接即可

## 科学上网

下载外网文件、克隆Github项目等操作，必须使用科学上网

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

![image-20240105115600487](E:\李杨课题组\论文笔记\image-20240105115600487.png)

## 远程桌面连接

远程连接直接找师兄问 向日葵 和 Teamviewer 密码，连接即可

## 注意事项 

1. 建议非必要不使用远程连接（由于在同一时间段内，远程连接只能单用户使用）
2. 使用远程连接，请先阅读服务器壁纸上的注意事项
3. 如需上传文件，尽量使用移动硬盘，到918实验室拷贝至服务器上 

