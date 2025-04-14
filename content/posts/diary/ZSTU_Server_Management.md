---
title: "ZSTU Server Management"
date: 2024-01-04T21:36:25+08:00
lastmod: 2024-01-04T21:36:25+08:00
author: ["SwimmingLiu"]

categories:
- 📓 Diary

tags:
- Server
- ZSTU

keywords:
- Server Management
- FRP

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
## FRP配置

### 跳板机 

```bash
# frps.ini 配置
[common]
bind_port = 7000 #frps服务监听的端口
token = 123 # 链接口令
```

```bash
./frps -c frps.ini # 启动frps
```

### 服务器

```bash
# frpc.ini
[common]
server_addr = x.x.x.x # 此处为 跳板机 的公网ip
server_port = 7000 # 跳板机上frps服务监听的端口
token = 123 # 链接口令

[ssh]
type = tcp
local_ip = 127.0.0.1 
local_port = 22 # 需要暴露的内网机器的端口
remote_port = 6000 # 暴露的内网机器的端口在vps上的端口
```

## SSH连接

```bash
ssh -p 6000 swimmingliu@server.ip # 普通ssh 连接
ssh swimmingliu@server.ip 6000	  # xshell ssh连接
```

## 用户管理

### 添加用户

```bash
sudo adduser xxx
```

### 删除用户

```bash 
sudo deluser xxx
```

## Magic Network

```bash
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
```

## Anaconda3 安装和配置

### 安装Anaconda3

```bash
wget --user-agent="Mozilla" https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/Anaconda3-2023.07-1-Linux-x86_64.sh
bash Anaconda3-2023.07-1-Linux-x86_64.sh
# https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive 清华源
```

### 配置之前的envs

```bash
cp -r old_envs_path anaconda/envs/		#迁移之前的envs环境
```

## 完结撒花❀❀❀

