---
title: "Java场景100题"
date: 2025-07-06T13:27:35+08:00
lastmod: 2025-07-06T13:27:35+08:00
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

# 并发与性能优化

## 1. 大文件上传如何处理？分片上传

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson001](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson001)

### 1.1 为什么需要分片上传?

大文件在上传的过程中，耗费的时间比较长。如果网络不稳定，很可能导致上传失败，需要重新上传。分片上传，就可以解决这个问题。

**分片上传定义**：将大文件拆分为不同的文件块，逐块进行上传

### 1.2 如何实现分片上传？

#### 1.2.1 如何存储分片信息

分片上传需要记录文件的属性 (md5、大小、名称)、分片数量、文件存储的完整路径 (本地路径)，还需要记录每个分块的上传情况 (分块大小、分块顺序、分块任务id)。可以用 **分块上传任务表** 和 **分块文件表** 来记录。

1. **分块任务表 (t_shard_upload)**

   ``` sql
   create table if not exists t_shard_upload(
       id varchar(32) primary key,
       file_name varchar(256) not null comment '文件名称',
       part_num int not null comment '分片数量',
       md5 varchar(128) comment '文件md5值',
       file_full_path varchar(512) comment '文件完整路径'
   ) comment = '分片上传任务表';
   ```

2. **分块文件表 (t_shard_upload_part）**

   ``` sql
   create table if not exists  t_shard_upload_part(
       id varchar(32) primary key,
       shard_upload_id varchar(32) not null comment '分片任务id（t_shard_upload.id）',
       part_order int not null comment '第几个分片，从1开始',
       file_full_path varchar(512) comment '文件完整路径',
       UNIQUE KEY `uq_part_order` (`shard_upload_id`,`part_order`)
   ) comment = '分片文件表，每个分片文件对应一条记录';
   ```

分块文件表和分块上传表是 **1 to M** 的关系，假如当前文件分为**10**块。则会出现**1**个分片上传任务，和**10**个分片文件记录。

#### 1.2.2 如何定义分块上传接口

分块上传可以分为下面 5 个接口

1. **创建分片上传任务接口 ** (`/shardUpload/init`)
   - 入参：文件名称、文件大小、文件md5
   - 出参：任务id、分片数量
   - 实现：计算分片数量，创建分片任务
2. **上传分片文件** (`/shardUpload/uploadPart`)
   - 入参：MultiPartFile 文件
   - 出参 ：分片文件记录id、任务id
   - 实现：存入文件到磁盘自动位置，创建分片文件记录
3. 合并分片、完成上传 (`/shardUpload/complete`)
   - 入参：任务id
   - 出参：布尔值
   - 实现：按照顺序合并所有二进制文件
4. 获取分片任务详细信息(`/shardUpload/detail`)
   - 入参：任务id 、文件md5 (二选一)
   - 出参：任务进度、文件名称、文件md5
   - 实现：读取分片文件表查看上传情况
   - 功能：获取分片任务的状态信息，如分片任务是否上传完毕，哪些分片已上传等信息，网络出现故障，可以借助此接口恢复上传

### 1.3 如果上传过程中，出现故障如何处理？

#### 情况1：浏览器无法读取刚才用户选择的文件了

此时需要用户重新选择文件，重新上传。这个地方也可以给大家提供另外一种思路，第1个接口创建分片任务的时候传入了文件的md5，按说这个值是具有唯一性的，那么就可以通过这个值找到刚才的任务，按照这种思路，就需要后端提供一个新的接口：通过文件的md5值找到刚才失败的那个任务，然后继续上传未上传的分片。

#### 情况2：浏览器可以继续读取刚才用户选择的文件

这种情况，可以先调用第4个接口，通过此接口可以知道那些分片还未上传，然后继续上传这些分片就可以了。

## 2. 多线程任务批处理通用工具类

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson002](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson002)

> 使用线程池批量发送短信，当短信发送完毕之后，记录耗时情况

废话不多说， 直接看代码 
**【主要知识点】**

- CountDownLatch：如果想要等异步线程执行之后，再继续回到原方法中，可以使用CountDownLatch
- Executors：用于创建线程池，重写 `executor.execute` 中的 `Runnable` 方法，可实现异步执行线程

```java
public class TaskDisposeUtils {
      /**
     * 使用线程池批处理文件，当所有任务处理完毕后才会返回
     *
     * @param taskList 任务列表
     * @param consumer 处理任务的方法
     * @param executor 线程池
     * @param <T>
     * @throws InterruptedException
     */
    public static <T> void dispose(List<T> taskList, Consumer<? super T> consumer, Executor executor) throws InterruptedException {
        if (taskList == null || taskList.isEmpty() || consumer == null) return;
        CountDownLatch latch = new CountDownLatch(taskList.size());
        for (T item : taskList) {
            executor.execute(() -> {
                try {
                    consumer.accept(item);
                } finally {
                    latch.countDown();
                }
            });
        }
        latch.await();
    }

    public static void main(String[] args) throws InterruptedException {
        long startTime = System.currentTimeMillis();
        List<String> taskList = List.of("短信-1", "短信-2", "短信-3");
        ExecutorService executor = Executors.newFixedThreadPool(10);
        dispose(taskList, TaskDisposeUtils::doTask, executor);
       System.out.println("任务处理完毕,耗时(ms):" + (System.currentTimeMillis() - startTime));
        executor.shutdown();
    }

    public static void doTask(String task) {
        System.out.println(task + "发送成功");
        try {
            TimeUnit.SECONDS.sleep(1);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

## 3. 接口性能压力测试工具

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson003](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson003)

### 3.1 常用压测工具类

[Jemeter](https://jmeter.apache.org/) 、[LoadRunner](https://blog.csdn.net/light2081/article/details/131175276)、ApiFox - 自动化测试

【注意】LoadRunner 只有window可以用

###  3.2 手写压测工具类

废话不多说，看代码

【主要知识点】

- 为什么 `updateFastestTime` 方法需要使用循环而不是直接 `compareAndSet` ：因为直接使用 `compareAndSet` 会出现线程不安全的情况，比如下面的情况

  ``` java
  // 错误的做法（非线程安全）
  int current = fastestCostTime.get();  // 线程A读取到值100
  if (current > costTime) {             // 线程A判断100 > 30，准备更新
      // 此时线程B也读取到100，也判断100 > 50，也准备更新
      fastestCostTime.set(costTime);    // 线程A设置为30
      // 线程B设置为50，覆盖了线程A的结果 (最小时间应该是30s)
  }
  // 使用while循环保证线程安全 （其实就是CAS）
  while (true){
     int fsCostTime = fastestCostTime.get();  // 线程A读取到100
    // 线程B此时将值改为90
    if (fastestCostTime.compareAndSet(fsCostTime, costTime)) {  // 线程A尝试将100改为50
        // 失败！因为当前值已经是90，不等于预期的100
        break;
    }
    // 继续循环，重新获取最新值90，再次尝试
  }
  ```

- 线程池参数：数依次为 `coolPoolSize` 、`maxPoolSize`、`keepAliveTime`、`TimeUnit`、`waitQueue` 等待队列

- 为什么线程需要预热：预热是为了避免线程创建时带来的额外开销，如果不预热，很可能前 `100` 个核心线程需要多消耗 `10~50 ms` 来创建线程

- AtomicInteger：防止多线程修改同一数据时，出现线程不安全的情况

  ``` java
  // 普通Integer的问题
  int value = counter;     // 1. 读取
  value = value + 1;       // 2. 修改
  counter = value;         // 3. 写入
  // 这三步不是原子的，可能被其他线程中断
  
  // AtomicInteger采用CAS + volatile 保证原子性
  atomicCounter.incrementAndGet();  // 原子操作
  ```

``` java
public class LoadRunnerUtils {
    public static class LoadRunnerResult {
        private int requests;           // 请求总数
        private int concurrency;        // 并发量
        private int successRequests;    // 成功请求数
        private int failRequests;       // 失败请求数
        private int timeTakenForTests;  // 请求总耗时(ms)
        private float requestsPerSecond; // 每秒请求数（吞吐量）
        private float timePerRequest;    // 每个请求平均耗时(ms)
        private float fastestCostTime;   // 最快的请求耗时(ms)
        private float slowestCostTime;   // 最慢的请求耗时(ms)
    }

    /**
     * 执行并发压力测试
     * @param requests    总请求数
     * @param concurrency 并发数量
     * @param command     被测试的代码
     * @return 压测结果
     */
    public static LoadRunnerResult run(int requests, int concurrency, Runnable command) throws InterruptedException {
        log.info("压测开始......");
        // 创建固定大小的线程池，预热所有核心线程
        ThreadPoolExecutor executor = createThreadPool(concurrency);
        // 用于等待所有请求完成的同步器
        CountDownLatch latch = new CountDownLatch(requests);  
        // 统计数据（使用原子类保证线程安全）
        AtomicInteger successCount = new AtomicInteger(0);
        AtomicInteger fastestTime = new AtomicInteger(Integer.MAX_VALUE);
        AtomicInteger slowestTime = new AtomicInteger(Integer.MIN_VALUE);
        long startTime = System.currentTimeMillis();
        // 提交所有压测任务
        for (int i = 0; i < requests; i++) {
            executor.execute(() -> executeRequest(command, successCount, fastestTime, slowestTime, latch));
        }
        // 等待所有请求完成
        latch.await();
        executor.shutdown();
        long endTime = System.currentTimeMillis();
        log.info("压测结束，总耗时(ms):{}", (endTime - startTime));
        // 返回压测结果
    }
    
    /**
     * 创建线程池并预热核心线程
     */
    private static ThreadPoolExecutor createThreadPool(int concurrency) {
        // 参数依次为 coolPoolSize、maxPoolSize、keepAliveTime、TimeUnit、等待队列
        ThreadPoolExecutor executor = new ThreadPoolExecutor(concurrency, concurrency,
                0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
        executor.prestartAllCoreThreads();
        return executor;
    }
    
    /**
     * 执行单个请求并统计结果
     */
    private static void executeRequest(Runnable command, AtomicInteger successCount, 
                                     AtomicInteger fastestTime, AtomicInteger slowestTime, 
                                     CountDownLatch latch) {
        try {
            long requestStart = System.currentTimeMillis();
            command.run(); // 执行被测试的方法
            int costTime = (int)(System.currentTimeMillis() - requestStart);
            // 更新统计数据
            updateFastestTime(fastestTime, costTime);
            updateSlowestTime(slowestTime, costTime);
            successCount.incrementAndGet();
        } catch (Exception e) {
            log.error("请求执行失败: {}", e.getMessage());
        } finally {
            latch.countDown(); // 标记当前请求完成
        }
    }
    
    /**
     * 原子更新最快耗时
     */
    private static void updateFastestTime(AtomicInteger fastestTime, int costTime) {
        while (true) {
            int current = fastestTime.get();
            if (current <= costTime || fastestTime.compareAndSet(current, costTime)) {
                break;
            }
        }
    }
}
```



