---
title: "个人简历常问问题"
date: 2025-02-27T10:52:54+08:00
lastmod: 2025-02-27T10:52:54+08:00
author: ["SwimmingLiu"]

categories:
- 💻 Job

tags:
- Java
- 后端面试

keywords:
- Java
- 后端面试

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

个人简历详情查看 -> [个人简历页](https://rxresu.me/dashboard/resumes)

## 1. 异步秒杀机制的异步是如何实现的?

**【正常秒杀的顺序】**

查询优惠券 -> 判断秒杀库存 -> 查询订单 -> 校验是否一人一单 -> 扣减库存 -> 创建订单

**【异步秒杀】**

为了实现用户异步下单，其实就是把是否能够下单的判断逻辑和下单的操作拆分开。

1. 采用Redis来判断是否有足够的库存和校验一人一单
   - 如果满足条件，把用户、订单id、商品id保存到阻塞队列，直接给用户返回秒杀成功。
   - 如果不满足条件，直接返回秒杀失败。
2. 后台线程会去执行queue里边的消息

这样就可以实现异步的秒杀下单了，那么如果实现判断秒杀库存和校验一人一单呢？

![1653561657295](https://oss.swimmingliu.cn/8f260a35-f4b5-11ef-a915-c858c0c1deba)

**【秒杀库存 + 一人一单】**

1. 用户下单之后，判断redis当中的库存key的value是否大于`0`
   - `value > 0` -> 第2步
   - `value <= 0` -> 直接返回库存不足 （返回`1`）
2. 如果库存充足，判断redis当中的秒杀商品key的 `set` 集合是否已包含`userid`
   - 包含`userid`， 说明用户已经下单了，直接返回当前用户已下单 (返回`2`)
   - 不包含 `userid` -> 第3步
3. 如果用户没有下单，将用户的 `userid` 存入 `set` 里面 (返回`0`)

**【注意】** 整个操作是原子性的，这样就确保了不会出现**超卖现象和一人多单现象**

```lua
-- 1.参数列表
-- 1.1.秒杀商品id
local voucherId = ARGV[1]
-- 1.2.用户id
local userId = ARGV[2]
-- 1.3.订单id
local orderId = ARGV[3]
-- 2.数据key
-- 2.1.库存key
local stockKey = 'seckill:stock:' .. voucherId
-- 2.2.秒杀商品订单key
local orderKey = 'seckill:order:' .. voucherId
-- 3.脚本业务
-- 3.1.判断库存是否充足 get stockKey
if(tonumber(redis.call('get', stockKey)) <= 0) then
    -- 3.2.库存不足，返回1
    return 1
end
-- 3.2.判断用户是否下单 SISMEMBER orderKey userId
if(redis.call('sismember', orderKey, userId) == 1) then
    -- 3.3.存在，说明是重复下单，返回2
    return 2
end
return 0
```

**【阻塞队列实现下单】**

1. 初始化一个`SingleThreadExecutor` 线程池，用于完成后续的下单操作
2. 判断上面`lua`脚本执行后的返回值
   - 如果返回值为`0`， 说明用户满足下单的资格 -> 第2步
   - 如果返回值不为`0`， 说明用户下单失败，返回异常信息
3. 将`userid`、优惠商品id、订单id等信息都存入阻塞队列(`ArrayBlockingQueue`)里面，返回订单id
4. 线程池会异步获取阻塞队列里面的订单信息，然后创建订单。

## 2. 如果用MySQL数据库，如何解决超卖现象

超卖现象的产生是因为多线程并发的时候，出现了库存扣为负数的现象。

1. 假如A线程查询时，库存为`1` 。随后，B线程查询库存也为`1`
2. A线程完成下单之后, 库存减为`0`。此时，B线程又完成下单，库存扣减为`-1`。

**【MySQL如何解决超卖问题】**

一般超卖问题都是采用乐观锁进行解决，也就是`CAS` 自旋。其实，就是判断读取前和第二次读取的，是否出现了数据不一致的情况。如果数据不一致，说明被其他人修改过，就放弃当前的操作。如果没有，就正常修改。

**【乐观锁】**

乐观锁会有一个版本号，每次操作数据会对版本号+1。再提交回数据是，会去校验是否比之前的版本是否大于`1`。如果超过`1`， 则说明当前的数据被修改过。 

乐观锁有一个变种是`CAS`，利用`CAS`进行无锁化机制加锁，`var5` 是操作前读取的内存值，`while` 中的`var1+var2` 是预估值，如果预估值 == 内存值，则代表中间没有被人修改过，此时就将新值去替换 内存值

```java
int var5;
do {
    var5 = this.getIntVolatile(var1, var2);
} while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
return var5;
```

**【MySQL如何解决一人一单】**

就是判断当前用于在数据库里面是否有订单, 可以用`count()` 去统计一下该用户秒杀订单的数量。如果大于`0`，则说明他已经买过商品了，就不能重复下单了。