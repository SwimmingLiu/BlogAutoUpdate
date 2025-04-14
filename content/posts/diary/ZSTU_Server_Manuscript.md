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

![image-20240105113129928](https://oss.swimmingliu.cn/B6xRW.png)

其中 `server.ip` 为服务器公网ip地址，端口为 `6969` 

## 安装Anaconda3

每个用户均被分配 `AnacondaAnaconda3-2023.07-1-Linux-x86_64.sh ` 于主目录

```bash
bash AnacondaAnaconda3-2023.07-1-Linux-x86_64.sh # 安装anaconda3
```

![image-20240105113942737](https://oss.swimmingliu.cn/B6KTv.png)

输入 `yes` 后， 再按回车键 即可

![image-20240105114146047](https://oss.swimmingliu.cn/B6one.png)

## 初始化Anaconda3

```bash
conda init bash	# 初始化conda
```

然后重新使用Xshell 连接即可

## Magic Network

下载外网文件、克隆Github项目等操作，必须使用Magic Network

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

取消Magic Network

```bash
unset http_proxy
unset https_proxy
```

如果使用上面的命令，不能连接Google. 需要远程桌面连接，打开CFW (默认是打开的)

```bash
nohup bash /home/dell/LYJ/Clash/cfw > cfw.out
```

![image-20240105115600487](https://oss.swimmingliu.cn/B6So3.png)

## 国内镜像下载

pip 清华源下载

```bash
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple packge      # packge为包名
```

conda 配置镜像

```bash
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/ 
# 以上两条是Anaconda官方库的镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
# 以上是Anaconda第三方库 Conda Forge的镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/pytorch/
# 以上是Pytorch的Anaconda第三方镜像
```

## 远程桌面连接

远程连接直接找师兄问 向日葵 和 Teamviewer 密码，连接即可

## 注意事项 

1. 建议非必要不使用远程连接（由于在同一时间段内，远程连接只能单用户使用）
2. 使用远程连接，请先阅读服务器壁纸上的注意事项
3. 如需上传文件，尽量使用移动硬盘，到918实验室拷贝至服务器上 

