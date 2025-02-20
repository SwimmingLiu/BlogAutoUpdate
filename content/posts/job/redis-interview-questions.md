---
title: "Redis面试题笔记"
date: 2025-02-20T21:21:45+08:00
lastmod: 2025-02-20T21:21:45+08:00
author: ["SwimmingLiu"]

categories:
- 💻 Job

tags:
- Java
- Redis

keywords:
- Java
- Redis

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

## 1. Redis主从复制的原理

【**主从复制的原理**】

1. 同步：从节点向主节点发送`psync`命令进行同步，从节点保存主节点返回的 `runid` 和 ` offset`
2. 全量复制：如果是第一次连接或者连接失败且`repl_backlog_buffer` 缓存区不包含`slave_repl_offset`， 则生成主节点的数据快照(RDB文件)发给从节点
3. 增量复制：全量复制完毕后，主从节点之间会保持长连接。如果连接没有断开或者`slave_repl_offset`仍然在`repl_backlog_buffer`中，则将后续的写操作传递给从节点，让数据保持一致。

**【全量复制细节】**

全量复制的过程是基于TCP长连接的，主要流程如下

1. 从节点发送`psync ? -1`表示需要建立连接进行同步，主节点返回主节点ID `runid` 和 复制进度`offset` (第一次同步用 -1 表示)。从节点接受之后，保存主节点的信息。
2. 主节点执行`bgsave`命令生成数据快照RDB文件，然后将RDB文件发送给从节点。从节点接受文件后，清除现有的所有数据，然后加载RDB文件
3. 如果在制作数据快照RDB文件的过程当中，主节点接收到了新的写操作，主节点会将其记录在`repl buffer` 里面。然后将`repl buffer`当中的写操作发给从节点，让其数据保持一致。

![Redis主从全量复制](https://oss.swimmingliu.cn/ac630d4c-ef8d-11ef-a882-c858c0c1deba)

**【增量复制细节】**

如果主从节点意外断开连接，为了保持数据的一致性，必须重新同步数据。如果使用全量复制来保持一致性的话，开销太大，所以采用增量复制。

增量复制的具体流程如下：

1. 连接恢复后，从节点会发送`psync {runid} {offset}`， 其中主节点ID `runid` 和 复制进度`offset`用于标识是哪一个服务器主机和复制进度。
2. 主节点收到`psync` 命令之后，会用`conitnue`响应告知从节点，采用增量复制同步数据
3. 最后，主节点根据`offset`查找对应的进度，将短线期间未同步的写命令，发送给从节点。同时，主节点将所有的写命令写入`repl_backlog_buffer`， 用于后续判断是采用增量复制还是全量复制。

【注意】从节点 `psync` 携带的 `offset` 为 `slave_repl_offset`。如果 `repl_backlog_buffer`包含`slave_repl_offset` 对应的部分，则采用增量复制，否则采用全量复制。`repl_backlog_buffer`的默认缓冲区大小为`1M`

![Redis主从增量复制](https://oss.swimmingliu.cn/ac9f21a3-ef8d-11ef-9016-c858c0c1deba)

【**为什么要主从复制**】

- **备份数据**：主从复制实现了数据的热备份，是持久化之外的数据冗余方式
- **故障恢复**：当主节点宕机之后，可以采用从节点提供服务。
- **负载均衡**:  主从复制实现了读写分离，只有主节点支持读写操作，从节点只有都操作。在读多写少的场景下，可以提高Redis服务器的并发量。

![Redis主从读写分离](https://oss.swimmingliu.cn/acad0d12-ef8d-11ef-b17f-c858c0c1deba)

## 2. Redis集群的实现原理是什么?

【**Redis集群基本知识**】

- **定义**: Redis集群由多个实例组成，每个实例存储部分数据 (每个实例之间的数据不重复) 。

 【注】集群和主从节点不是一个东西，集群的某一个实例当中可能包含一个主节点 + 多个从节点

- **为什么用**

| 问题             | 解决方案                                                  |
| ---------------- | --------------------------------------------------------- |
| **容量不足**     | 数据分片，将数据分散不存到不同的主节点                    |
| **高并发写入**   | 数据分片，将写入请求分摊到多个主节点                      |
| **主机宕机问题** | 自动切换主从节点，避免影响服务， 不需要手动修改客户端配置 |

- **节点通信协议**：Redis集群采用Gossip协议, 支持分布式信息传播、延迟低、效率高。采用去中心化思想，任意实例(主节点)都可以作为请求入口，节点间相互通信。
- **分片原理**： 采用哈希槽(Hash Slot)机制来分配数据，整个空间可以划分为**16384** (16 * 1024)个槽。 每个Redis负责一定范围的哈希槽,数据的key经过哈希函数计算之后对**16384**取余可定位到对应的节点。

![Redis集群架构图](https://oss.swimmingliu.cn/acc54635-ef8d-11ef-971e-c858c0c1deba)

【**集群节点之间的交互协议**】

-  **为什么用Gossip协议**

1. 分布式信息传播：每个节点定期向其他节点传播状态信息，确保所有节点对集群的状态有一致视图 (采用`ping` 发送 和 `pong` 接受，就像检查心跳一样 )
2. 低延迟、高效率：轻量级通信方式，传递信息很快
3. 去中心化：没有中心节点，任意实例(主节点)都可以作为请求入口，节点间相互通信。

- **Gossip协议工作原理**

1. 状态报告和信息更新：特定时间间隔内，向随机的其他节点报告自身情况 （主从关系、槽位分布）。其他节点接收到之后，会相应的更新对应的节点状态信息
2. 节点检测：通过周期性交换状态信息，可以检测到其他节点的存活状态。预定时间内未响应，则标记为故障节点。
3. 容错处理：如果某个节点故障之后，集群中的其他节点可以重新分配槽位，保持系统的可用性

【**哈希槽的相关机制**】

假定集群中有三个节点，Node1 (0 - 5460)、Node2(5461-10922)、Node3(10923-16383)

集群使用哈希槽的流程如下：

- **计算哈希槽** 

1. 使用CRC16哈希算法计算`user:0001`的CRC16的值
2. 将CRC16的值对16384进行取余 (哈希槽 = CRC16 % 16383)
3. 假如CRC16为12345，哈希槽 = 12345 % 16383 = 12345

- **确定目标节点** ：查询到12345为Node3的存储的键，向该节点发送请求
- **当前非对应节点** ：假设当前连接的节点为Node1，Node1将返回`MOVED`错误到客户端，并让客户端根据`MOVED`携带的Node3的信息(`ip`和端口)重新进行连接，最后从新发送`GET user:0001`请求，获得结果。

## 3. Redis的哨兵机制（Sentinel）是什么？

**【哨兵作用】**

- 监控：哨兵不断监控主从节点的运行状态,定时发送ping进行检测
- 故障转移: 当主节点发生故障时, 哨兵会先踢出所有失效的节点, 然后选择一个有效的从节点作为新的主节点, 并通知客户端更新主节点的地址
- 通知: 哨兵可以发送服务各个节点的状态通知，方便观察Redis实例的状态变化。（比如主节点g了，已经更换为新的主节点）

**【哨兵机制的主观下线和客观下线】**

- **主观下线**：哨兵在监控的过程中，每隔1s会发送 `ping` 命令给所有的节点。如果哨兵超过`down-after-milliseconds` 所配置的时间，没有收到 `pong` 的响应，就会认为节点主观下线。

- **客观下线**：某个哨兵发现节点主线下线后，不能确认节点是否真的下线了（可能是网络不稳定），就询问其他的哨兵是否主观下线了。等待其他哨兵的确认，进行投票，如果超过半数+1 (总哨兵数/2 + 1)，就认定为客观下线。

【注】客观下线只对主节点适用，因为从节点也没必要这样子判断，g了就g了呗。

**【哨兵leader如何选举】**

哨兵leader是采用分布式算法raft选出来的。具体流程如下：

1. 候选人：当哨兵判断为主观下线，则可以当选候选人
2. 投票：每个哨兵都可以投票，但是只能投一票。候选者会优先投给自己。
3. 选举：选取投票结果半数以上的候选人作为leader (哨兵一般设置为奇数，防止平票)

**【主节点如何选举】**

哨兵判断主节点客观下线之后，会踢出所有下线的节点，然后从有效的从节点选新的主节点。选取依据如下：

1. 优先级：按照从节点的优先级 `slave-priority`，优先级的值越小越高。 
2. 主从复制offset值：如果优先级相同，则判断主从复制的offset值哪一个大，表明其同步的数据越多，优先级就越高。
3. 从节点ID：如果上述条件均相同，则选取ID较小的从节点作为主节点。