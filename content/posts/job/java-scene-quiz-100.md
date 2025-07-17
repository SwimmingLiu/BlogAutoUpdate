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
> **【主要知识点】**

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

【主要知识点】

- **为什么 `updateFastestTime` 方法需要使用循环而不是直接 `compareAndSet`** ：因为直接使用 `compareAndSet` 会出现线程不安全的情况，比如下面的情况

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

- **线程池参数**：数依次为 `coolPoolSize` 、`maxPoolSize`、`keepAliveTime`、`TimeUnit`、`waitQueue` 等待队列

- **为什么线程需要预热**：预热是为了避免线程创建时带来的额外开销，如果不预热，很可能前 `100` 个核心线程需要多消耗 `10~50 ms` 来创建线程

- **AtomicInteger**：防止多线程修改同一数据时，出现线程不安全的情况 

  ``` java
  // 普通Integer的问题
  int value = counter;     // 1. 读取
  value = value + 1;       // 2. 修改
  counter = value;         // 3. 写入
  // 这三步不是原子的，可能被其他线程中断
  
  // AtomicInteger采用CAS + volatile 保证原子性
  atomicCounter.incrementAndGet();  // 原子操作
  ```

- `volatile` 关键字在`AtomicInteger`作用：`volatile` 就像是给变量贴了个"实时更新"的标签。通过内存可见性和禁止指令重排序来保证实时更新

  ```Java
  // 没有volatile的情况
  int count = 0;
  
  // 线程A修改了count = 100
  // 线程B可能还是看到count = 0（因为每个线程都有自己的缓存）
  ```

  所以，如果 没有`volatile` 关键字， 没有线程都有自己的"小本本"（CPU缓存，线程私有），可能记录的数据不是最新的。

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

## 4. Semaphore实现接口限流

> 如果后端服务是单体服务，且只部署在一台服务器上，可以采用 `Semaphore` 信号量的方式实现简单的接口限流机制

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson005](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson005)

`Semaphore` : JUC当中对信号量的实现，用于控制同时访问共享资源的线程数量。

``` java
public class TestController {
    /**
     * Juc中的Semaphore可以实现限流功能，可以将 Semaphore 想象成停车场入口的大爷，
     * 大爷手里面拥有一定数量的停车卡（也可以说是令牌），卡的数量是多少呢？就是Semaphore构造方法中指定的，如下就是50个卡，
     * 车主想进去停车，先要从大爷手中拿到一张卡，出来的时候，需要还给大爷，如果拿不到卡，就不能进去停车。
     * semaphore 内部提供了获取令牌，和还令牌的一些方法
     */
    private Semaphore semaphore = new Semaphore(50);
    /**
     * 来个案例，下面是一个下单的方法，这个方法最多只允许 50 个并发，若超过50个并发，则进来的请求，最多等待1秒，如果无法获取到令牌，则快速返回失败，请重试
     * @return
     */
    @GetMapping("/placeOrder")
    public String placeOrder() throws InterruptedException {
        /**
         * semaphore 在上面定义的，里面有50个令牌，也就是同时可以支持50个并发请求
         * 下面的代码，尝试最多等待1秒去获取令牌，获取成功，则进入下单逻辑，获取失败，则返回系统繁忙，请稍后重试
         */
        boolean flag = this.semaphore.tryAcquire(1, 1L, TimeUnit.SECONDS);
        // 获取到令牌，则进入下单逻辑
        if (flag) {
            try {
                //这里休眠2秒，模拟下单的操作
                TimeUnit.SECONDS.sleep(2);
                return "下单成功";
            } finally {
                //这里一定不要漏掉了，令牌用完了，要还回去
                this.semaphore.release();
            }
        } else {
            return "系统繁忙，请稍后重试";
        }
    }
}
```

## 5. 并行查询 - 提交接口响应速度

> 需求分析：当前接口需要实现下面的功能
>
> 接收一个商品id，需要返回商品下面这些信息，这些信息都在不同的表中，通过商品id就可以查到
>
> - 商品基本信息（如商品名称、价格等基本信息)
> - 商品描述信息（可能是富文本，放在单独的表中）
> - 商品收藏量
> - 商品评论量

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson006](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson006)

### 5.1  常规做法

按照商品 `id` 分别查询不同的数据库，在JVM内存中对结果进行组装。

``` java
public class GoodsController {
    /**
     * 根据商品id获取商品信息(基本信息、描述信息、评论量，收藏量)
     *
     * @param goodsId 商品id
     * @return
     * @throws InterruptedException
     */
    @GetMapping("/getGoodsDetail")
    public GoodsDetailResponse getGoodsDetail(@RequestParam("goodsId") String goodsId) {
        long st = System.currentTimeMillis();
        GoodsDetailResponse goodsDetailResponse = new GoodsDetailResponse();
        // 1、获取商品基本信息，耗时100ms
        goodsDetailResponse.setGoodsInfo(this.getGoodsInfo(goodsId));
        //2、获取商品描述信息，耗时100ms
        goodsDetailResponse.setGoodsDescription(this.getGoodsDescription(goodsId));
        //3、获取商品评论量，耗时100ms
        goodsDetailResponse.setCommentCount(this.getGoodsCommentCount(goodsId));
        //4、获取商品收藏量，耗时100ms
        goodsDetailResponse.setFavoriteCount(this.getGoodsFavoriteCount(goodsId));
        // 总耗时为500ms左右
        LOGGER.info("获取商品信息，普通版耗时：{} ms", (System.currentTimeMillis() - st));
    }
    // 使用sleep模拟数据库查询的延迟
    private int getGoodsFavoriteCount(String goodsId) { 
      try {
          log.info("获取商品收藏量");
          TimeUnit.MILLISECONDS.sleep(100);
      } catch (InterruptedException e) {
          Thread.currentThread().interrupt();
          throw new RuntimeException(e);
      }
      return 10000;
  }
```

### 5.2 使用线程池并行查询商品信息

> 分析：上方的4个商品信息查询之间并没有任何依赖，这些没有依赖的查询其实是可以并行查询的。所以可以使用线程池同时去拿这4个结果，然后在JVM内存中进行组装

【主要知识点】

- CompletableFuture (CF)：主要用于异步执行多个相互独立的任务 （适合异步任务编排，可以方便地组合和转换任务结果）

- CountDownLatch (CDL): 主要用于线程协调，等待一组线程完成 （一般就是用来等待一批线程执行完）

- CF 和 CDL 区别

  - **CountDownLatch** 👉 **运动会起跑**
    裁判（主线程）等待所有运动员（子线程）就位（`countDown`），枪响（计数器归零）后统一开跑。
  - **CompletableFuture** 👉 **外卖订单流程**
    下单（异步任务）→ 制作（`thenApply` 加工）→ 配送（`thenAccept` 交付），每个环节自动触发下一步，无需阻塞

  | **对比维度** | **CountDownLatch**                      | **CompletableFuture**                              |
  | :----------- | :-------------------------------------- | :------------------------------------------------- |
  | **核心用途** | 线程等待机制（阻塞线程直到任务完成）    | 异步编程模型（非阻塞链式任务处理）                 |
  | **工作方式** | 通过计数器递减（`countDown()`）触发释放 | 通过回调链（`thenApply()`/`thenAccept()`）传递结果 |
  | **线程阻塞** | 调用线程主动阻塞（`await()`）           | 调用线程不阻塞，通过回调处理结果                   |
  | **结果传递** | ❌ 不支持任务结果传递                    | ✅ 支持任务结果传递和转换                           |
  | **异常处理** | ❌ 需手动捕获异常                        | ✅ 内置异常处理（`exceptionally()`/`handle()`）     |
  | **任务组合** | ❌ 仅支持单次计数                        | ✅ 支持多任务组合（`allOf()`/`anyOf()`）            |
  | **复用性**   | ❌ 计数器归零后不可重用                  | ✅ 可重复组合新任务                                 |
  | **典型场景** | 启动前等待资源初始化、批量任务并行执行  | 异步API调用、流水线式数据处理、微服务编排          |

  下面看一组CF和CDL的代码对比：

  ```java
  public class AsyncDemo {
      // 模拟线程池
      private static final ExecutorService executor = Executors.newFixedThreadPool(3);
      
      /**
       * CompletableFuture方式 - 支持任务编排和结果处理
       */
      public GoodsInfo getGoodsInfoWithCF(String goodsId) {
          GoodsInfo result = new GoodsInfo();
          // 1. 创建多个异步任务
          CompletableFuture<String> basicInfoFuture = CompletableFuture
              .supplyAsync(() -> queryBasicInfo(goodsId), executor);
          CompletableFuture<Integer> priceFuture = CompletableFuture
              .supplyAsync(() -> queryPrice(goodsId), executor);
          // 2. 对结果进行处理转换    
          CompletableFuture<String> processedInfo = basicInfoFuture
              .thenApply(info -> "处理后的商品信息: " + info)
              .exceptionally(ex -> "获取商品信息失败");
          // 3. 等待所有任务完成并组装结果    
          CompletableFuture.allOf(processedInfo, priceFuture).join();
          result.setInfo(processedInfo.join());
          result.setPrice(priceFuture.join());
          return result;
      }
      
      /**
       * CountDownLatch方式 - 仅支持等待多个线程完成
       */
      public GoodsInfo getGoodsInfoWithCDL(String goodsId) throws InterruptedException {
          GoodsInfo result = new GoodsInfo();
          CountDownLatch latch = new CountDownLatch(2);
          // 1. 需要手动创建线程执行任务
          executor.submit(() -> {
              try {
                  result.setInfo(queryBasicInfo(goodsId));
              } finally {
                  latch.countDown();
              }
          });
          executor.submit(() -> {
              try {
                  result.setPrice(queryPrice(goodsId));
              } finally {
                  latch.countDown();
              }
          });
          // 2. 等待所有任务完成
          latch.await();
          return result;
      }
  ```

  如果需要异步并行 + 任务编排，就用 `CompletableFuture` ；如果只是用于等待一批异步任务执行完，就用 `CountDownLaunch`

``` java
public class GoodsController {
		/**
     * 优化后的方法，根据商品id获取商品信息(基本信息、描述信息、评论量，收藏量)
     *
     * @param goodsId 商品id
     * @return
     * @throws InterruptedException
     */
    @GetMapping("/getGoodsDetailNew")
    public GoodsDetailResponse getGoodsDetailNew(@RequestParam("goodsId") String goodsId) {
        long st = System.currentTimeMillis();
        GoodsDetailResponse goodsDetailResponse = new GoodsDetailResponse();
        // 1、获取商品基本信息，耗时100ms
        CompletableFuture<Void> goodsInfoCf = CompletableFuture.runAsync(() -> goodsDetailResponse.setGoodsInfo(this.getGoodsInfo(goodsId)), this.goodsThreadPool);
        //2、获取商品描述信息，耗时100ms
        CompletableFuture<Void> goodsDescriptionCf = CompletableFuture.runAsync(() -> goodsDetailResponse.setGoodsDescription(this.getGoodsDescription(goodsId)), this.goodsThreadPool);
        //3、获取商品评论量，耗时100ms
        CompletableFuture<Void> goodsCommentCountCf = CompletableFuture.runAsync(() -> goodsDetailResponse.setCommentCount(this.getGoodsCommentCount(goodsId)), this.goodsThreadPool);
        //4、获取商品收藏量，耗时100ms
        CompletableFuture<Void> goodsFavoriteCountCf = CompletableFuture.runAsync(() -> goodsDetailResponse.setFavoriteCount(this.getGoodsFavoriteCount(goodsId)), this.goodsThreadPool);
        //等待上面执行结束
        CompletableFuture.allOf(goodsInfoCf, goodsDescriptionCf, goodsCommentCountCf, goodsFavoriteCountCf).join();
        // 总耗时大约在100ms左右
        LOGGER.info("获取商品信息，使用线程池并行查询耗时：{} ms", (System.currentTimeMillis() - st));
        return goodsDetailResponse;
    }
}
```

## 6. 并行查询可能存在的问题？如何解决？

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson006](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson006)

### 6.1 并行查询中存在的问题

并行查询所用到的线程池配置是非常关键且重要的配置，如果设置不当可能出现严重的性能问题。

比如将核心线程数设置为了 `1` ，而队列大小没有限制，那么所有的请求都变成串行了，会导致请求响应非常慢。另外，核心线程池和队列大小的上限也应该匹配。如果核心线程池设置为 `10` ，而队列大小没有设置上限。该线程池同时只可支持 `10` 个任务并行，其他的请求都会变成串行执行了，进入队列排队，从而导致接口响应特别慢。

### 6.2 如何解决并行查询的问题

解决并行查询问题的核心：不要让任务排队或者排队时间不要太长

下面从线程池的执行流程入手，优化并行查询问题

![image-20240405093250650](https://oss.swimmingliu.cn/536e392e-6313-11f0-99d8-caaeffceb345)

**【解决方案】**

- **增大线程数**：可以将**核心线程数**、**最大线程数调大**，但不能调到超过CPU的最大负载，不然可能会降低系统性能分。该参数应该根据业务的指标进行压测得到一个合理的值

- **减少队列数**：

  - 将队列大小设置的比较小，这样排队的时间大概率会比较短，或者排队失败，直接后面的流程 

    ⚠️ 之前理解存在误区：应该是核心线程检查完了，就检查能否放入队列，再检查最大线程数。而不是核心线程检查完，就去检查最大线程数。

  - 减少队列数至 `0`：这样任务就不会进入队列，而直接创建新的线程去执行，或者走拒绝策略

    >  `LinkedBlockingQueue、ArrayBlockingQueue` 容量是不允许为0的，如果需要用到容量为0的队列，则需要使用**同步阻塞队列** `SynchronousQueue` 

- **拒绝策略**：可以使用 `CallerRunsPolicy`，这个策略是直接在调用线程的当前线程中执行。该策略可以保证任务能快速被处理，不会一直处于阻塞态

  | 策略名称                | 行为描述                                       | 适用场景                     | 生活实例类比                   |
  | :---------------------- | :--------------------------------------------- | :--------------------------- | :----------------------------- |
  | **AbortPolicy**         | 直接抛出 `RejectedExecutionException` 异常     | 需要严格保证任务不丢失的场景 | 餐厅满员时直接拒绝新顾客       |
  | **CallerRunsPolicy**    | 让提交任务的线程自己执行该任务                 | 需要降低任务提交速度的场景   | 经理亲自处理无法分配的任务     |
  | **DiscardPolicy**       | 静默丢弃新任务，不抛异常也不执行               | 允许丢弃非关键任务的场景     | 快递爆仓时直接丢弃新包裹       |
  | **DiscardOldestPolicy** | 丢弃队列中最老的任务，然后尝试重新提交当前任务 | 优先处理新任务的场景         | 超市排队时让等待最久的顾客离开 |

- **线程池隔离**：不同的业务最好使用不同的线程池，互不影响，强烈建议**核心业务**一定要使用单独的线程池。

**【优化后的线程池配置】**

```java
@Bean
public ThreadPoolTaskExecutor goodsThreadPool() {
    ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
    threadPoolTaskExecutor.setThreadNamePrefix("ThreadPool-Goods-");
    // 核心线程数为cpu核数 * 4，最大线程数据为cpu核数 * 8
    threadPoolTaskExecutor.setCorePoolSize(Runtime.getRuntime().availableProcessors() * 4);
    threadPoolTaskExecutor.setMaxPoolSize(Runtime.getRuntime().availableProcessors() * 8);
    // 队列容量为0，则任务就不会进入队列
    threadPoolTaskExecutor.setQueueCapacity(0);
    // 拒绝策略使用CallerRunsPolicy，让当前线程去兜底去执行任务
    threadPoolTaskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
    return threadPoolTaskExecutor;
}
```

## 7. 接口性能调优之大 (长) 事务优化 

> 日常编程中，容易遇到事务执行时间较长的情况。如果该事务的对应的接口，请求量暴增，由于数据库连接池的限制，大部分请求可能会失败。

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson009](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson009)

### 7.1 大(长)事务所带来的问题

下面的代码带有 `@Transcational` 注解， 说明该方法会交给 `Spring` 来管理该方法的事务。具体执行逻辑如下：

``` shell
1、Spring去数据库连接池拿到一个数据库连接
2、开启事务
3、执行bigTransaction()中的代码
4、提交事务
5、将数据库连接还给数据库连接池中
```

在这个过程中，数据库连接会一直被占用。因为数据库连接时有限的，并且是非常稀缺的资。如果长时间被占用，并且数据库连接池中的可用连接都被占用了，则其他请求无法连接数据库。它会导致连接超时，执行方法失败。例如，下面方法中 `this.getData()` 方法会占用 `5s` 的时间。如果 `5s` 内有 `100` 条请求并发执行，且数据库连接池最大连接数为 `20`，超时时间为`3s`。 则剩余的 `80` 条请求，都会出现连接失败的现象。

```java
@Transactional
public void bigTransaction() throws InterruptedException {
    // 1、getData()方法模拟一个比较耗时的获取数据的操作，这个方法内部会休眠5秒
    String data = this.getData();
    //2、将上面获取到的数据写入到db中
    Lesson007PO po = new Lesson007PO();
    po.setId(UUID.randomUUID().toString());
    po.setData(data);
    this.lesson007Mapper.insert(po);
}

public String getData() throws InterruptedException {
    //休眠5秒
    TimeUnit.SECONDS.sleep(5);
    return UUID.randomUUID().toString();
}
```

### 7.2 如何解决大 (长) 事务的问题 (拆分为小事务)

**优化方法**：将大(长)事务进行拆分，将无需数据库连接的部分拆出来，将事务最小化。
例如，上方的代码中，`this.getData()` 方法是不需要操作数据库的，只有最后的 `insert` 方法才需要连接数据库。所以，可以将事务的粒度控制在 `insert` 方法中。

**TranscationTemplate**：Spring中的 `TranscationTemplate` 工具类能够采用编程式的方案，灵活控制事务的粒度。

``` java
/**
 * 使用 TransactionTemplate 编程式事务，可以灵活的控制事务的范围
 *
 * @throws InterruptedException
 */
public void smallTransaction() throws InterruptedException {
    // 1、调用getData()方法，讲获取的数据写到db中，假设 getData方法比较耗时，比如耗时 5秒
    String data = this.getData();

    //2、将上面获取到的数据写入到db中
    Lesson007PO po = new Lesson007PO();
    po.setId(UUID.randomUUID().toString());
    po.setData(data);

    // this.transactionTemplate.executeWithoutResult可以传入一个Consumer，这个Consumer表述需要在事务中执行的业务操作
    this.transactionTemplate.executeWithoutResult(action -> {
        this.lesson007Mapper.insert(po);
    });
}
```

【总结】

1. **控制事务粒度** ：使用 `TransactionTemplate` 编程式事务，精准控制事务的粒度，尽量让事务小型化
2. **非数据库相关代码，避免出现在事务中**：尽量避免将没有事务的耗时操作放到事务代码中；避免在事务中执行远程操作，远程操作是不需要用到本地事务的，所以没有必要放在事务中
3. **事务集中化**：尽量让事务的操作集中在一起执行，比如都放到方法最后，使用 `TransactionTemplate` 执行，这样可使事务最小化

## 8. 动态线程池

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson009](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson009)

### 8.1 为什么需要动态线程池

> 平时我们在开发中，创建了不少线程池，这些线程池都处于游离状态，不方便管理和监控。无法知道目前系统中有哪些线程池、以及每个线程池当前的一个状态，负载情况等，所以我们可以开发一个线程池管理器来解决这个问题。

### 8.2 线程池管理器

统管系统中所有线程池，负责所有线程池的创建、监控等操作。

``` java
public class ThreadPoolManager { 
     /** 线程池Map */
     private static Map<String, ThreadPoolTaskExecutor> threadPoolMap = new ConcurrentHashMap<String, ThreadPoolTaskExecutor>(16);
		/**
     * 创建新的线程池，如果线程池已经创建，返回已经创建的线程池
     *
     * @param name                     线程池名称
     * @param corePoolSize             核心线程数
     * @param maxPoolSize              最大线程数
     * @param queueCapacity            队列大小
     * @param keepAliveSeconds         线程池存活时间（秒）
     * @param threadFactory            线程工厂
     * @param rejectedExecutionHandler 拒绝策略
     * @return
     */
    public static ThreadPoolTaskExecutor newThreadPool(String name, int corePoolSize, int maxPoolSize, int queueCapacity, int keepAliveSeconds, ThreadFactory threadFactory, RejectedExecutionHandler rejectedExecutionHandler) {
        return threadPoolMap.computeIfAbsent(name, threadGroupName -> {
            ThreadPoolTaskExecutor threadPoolExecutor = new ThreadPoolTaskExecutor() {
                private boolean initialized = false; // 初始化标记
                @Override
                protected BlockingQueue<Runnable> createQueue(int queueCapacity) {
                    if (queueCapacity > 0) {
                      	// 使用自定义的 ResizeLinkedBlockingQueue 替代默认队列，保证能够在运行时动态调整队列大小
                        return new ResizeLinkedBlockingQueue<>(queueCapacity);
                    } else {
                      	// 当队列容量设置为0时，使用 SynchronousQueue。表示任务必须立即被线程处理，否则就会被拒绝
                        return new SynchronousQueue<>();
                    }
                }
                @Override
                public void setQueueCapacity(int queueCapacity) {
                  	// 确保线程池已初始化且队列类型正确时才调整容量
                    if (this.initialized && this.getThreadPoolExecutor() != null &&
                            this.getThreadPoolExecutor().getQueue() != null &&
                            this.getThreadPoolExecutor().getQueue() instanceof ResizeLinkedBlockingQueue) {
                        ((ResizeLinkedBlockingQueue) this.getThreadPoolExecutor().getQueue()).setCapacity(queueCapacity);
                    }
                    super.setQueueCapacity(queueCapacity);
                }

                @Override
                public void afterPropertiesSet() {
                  	// 使用 initialized 标记确保线程池只被初始化一次
                    if (initialized) {
                        return;
                    }
                    super.afterPropertiesSet();
                    this.initialized = true;
                }
            };
            // 设置参数
            threadPoolExecutor.setCorePoolSize(corePoolSize);
            ...
            if (threadFactory != null) {
                // ThreadFactory相当于线程池的一些规定 (招聘标准)
                threadPoolExecutor.setThreadFactory(threadFactory);
            }
            if (rejectedExecutionHandler != null) {
                // 拒绝执行的策略，上方提到的四种拒绝策略
                threadPoolExecutor.setRejectedExecutionHandler(rejectedExecutionHandler);
            }
            threadPoolExecutor.afterPropertiesSet();
            return threadPoolExecutor;
        });
    }
   
    /** 获取所有线程池信息 */
    public static List<ThreadPoolInfo> threadPoolInfoList() {
        return threadPoolMap
                .entrySet()
                .stream()
                .map(entry -> threadPoolInfo(entry.getKey(), entry.getValue()))
                .collect(Collectors.toList());
    }
 }
```

### 8.3 动态线程池

动态线程池：无需重启的情况下，可以对线程池进行扩缩容，比如改变线程池的核心线程数量、最大线程数量、队列容量等。

#### 8.3.1 常用动态线程池

**dynamic-tp**：https://github.com/dromara/dynamic-tp

#### 8.3.2 手动实现动态线程池

>线程池中会用到Java中的阻塞队列 `java.util.concurrent.BlockingQueue`，目前 jdk中自带几个阻塞队列都不支持动态扩容。
>比如 `java.util.concurrent.LinkedBlockingQueue`，它的 `capacity` 是 `final` 修饰的，不支持修改。

**动态调整大小的前提**：动态调整线程池大小需要队列容量能够支持调整，我们需要创建可扩容的阻塞队列 `ResizeLinkedBlockingQueue`。代码是从`LinkedBlockingQueue` 中拷贝过来的，增加可修改容量 的`setCapacity` 方法。然后创建线程池的时，使用自定义的阻塞队列便可以实现线程池的动态扩容。

``` java
/**
 * 设置容量
 * @param capacity
 */
public void setCapacity(int capacity) {
    if (capacity <= 0) throw new IllegalArgumentException();
    final ReentrantLock putLock = this.putLock;
    putLock.lock();
    try {
        if (count.get() > capacity) {
            throw new IllegalArgumentException();
        }
        this.capacity = capacity;
    } finally {
        putLock.unlock();
    }
}
```

动态更改线程池大小的方法如下所示：

``` java
public class ThreadPoolManager {  
		/**
     * 动态变更线程池（如：扩缩容、扩缩队列大小）
     * @param threadPoolChange 变更线程池信息
     */
    public static void changeThreadPool(ThreadPoolChange threadPoolChange) {
        ThreadPoolTaskExecutor threadPoolTaskExecutor = threadPoolMap.get(threadPoolChange.getName());
        if (threadPoolTaskExecutor == null) {
            throw new IllegalArgumentException();
        }
        if (threadPoolChange.getCorePoolSize() > threadPoolChange.getMaxPoolSize()) {
            throw new IllegalArgumentException();
        }
        synchronized (ThreadPoolManager.class) {
            // 增加线程情况：需要修改的最大线程大于当前核心线程，则设置当前核心线程为修改后的最大线程
            if (threadPoolChange.getMaxPoolSize() > threadPoolTaskExecutor.getCorePoolSize()) {
              	// 先调整最大线程数，再调整核心数 （先扩大总量，再扩大局部）
                threadPoolTaskExecutor.setMaxPoolSize(threadPoolChange.getMaxPoolSize());
                threadPoolTaskExecutor.setCorePoolSize(threadPoolChange.getCorePoolSize());
                threadPoolTaskExecutor.setQueueCapacity(threadPoolChange.getQueueCapacity());
            } else {
              	// 先减少局部，再减少总量
                threadPoolTaskExecutor.setCorePoolSize(threadPoolChange.getCorePoolSize());
                threadPoolTaskExecutor.setMaxPoolSize(threadPoolChange.getMaxPoolSize());
                threadPoolTaskExecutor.setQueueCapacity(threadPoolChange.getQueueCapacity());
            }
        }
    } 
}
```

# 分布式与微服务

## 1. SpringBoot实现动态Job的实战

相关代码：[https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson011](https://github.com/SwimmingLiu/JavaSceneQuiz100/tree/main/lesson011)

### 1.1 Job表 - 表结构

| 字段名      | 类型         | 可空 | 默认 | 备注                                        |
| ----------- | ------------ | ---- | ---- | ------------------------------------------- |
| id          | varchar(50)  | 否   | -    | id，主键                                    |
| name        | varchar(100) | 否   | -    | job名称，可以定义一个有意义的名称           |
| cron        | varchar(50)  | 否   | -    | job的执行周期，cron表达式                   |
| bean_name   | varchar(100) | 否   | -    | job需要执行那个bean，对应spring中bean的名称 |
| bean_method | varchar(100) | 否   | -    | job执行的bean的方法                         |
| status      | smallint     | 否   | 0    | job的状态,0：停止，1：执行中                |

### 1.2 Job执行管理器

``` java
@Component
public class SpringJobRunManager implements CommandLineRunner {
  	// applicationContext 主要用于在任务执行时动态获取和调用指定的 Bean 和方法，实现灵活的任务调度。
    // 可以把它理解为“Spring 管理的对象工厂和服务总线”
    @Autowired
    private ApplicationContext applicationContext;
		// threadPoolTaskScheduler 是 Spring 提供的线程池定时任务调度器，主要用于在应用中以多线程方式执行定时任务（如定时执行、Cron表达式调度等）
    @Autowired
    private ThreadPoolTaskScheduler threadPoolTaskScheduler;
    // job表相关服务
    @Autowired
    private JobService jobService;
    /**
     * 系统重正在运行中的job列表
     */
    private Map<String, SpringJobTask> runningJobMap = new ConcurrentHashMap<>();
    /**
     * springboot应用启动后会回调
     *
     * @param args incoming main method arguments
     * @throws Exception
     */
    @Override
    public void run(String... args) throws Exception {
        //1、启动job
        this.startAllJob();
        //2、监控db中job的变化（job增、删、改），然后同步给job执行器去执行
        this.monitorDbJobChange();
    }
}
```

Springboot应用启动之后会回调 `CommandLineRunner` 中的 `run` 方法，依次启动job，并监控job中的变化。

#### 1.2.1 启动job

启动job的方式比较简单，就是从数据库中找出所有需要启动的job（状态为start），然后循环启动。

启动job具体方式如下：

```java 
private void startAllJob() {
    List<Job> jobList = this.jobService.getStartJobList();
    for (Job job : jobList) {
        this.startJob(job);
    }
}
/**
 * 启动一个定时任务（job）
 *
 * @param job 需要启动的任务对象
 */
private void startJob(Job job) {
    // 1. 创建任务执行体，注入Spring上下文，便于任务内获取Bean对象
    SpringJobTask springJobTask = new SpringJobTask(job, this.applicationContext);
    // 2. 根据job的cron表达式创建调度触发器
    CronTrigger trigger = new CronTrigger(job.getCron());
    // 3. 使用线程池调度器注册任务，返回调度句柄
    ScheduledFuture<?> scheduledFuture = this.threadPoolTaskScheduler.schedule(springJobTask, trigger);
    // 4. 记录调度句柄到任务对象，便于后续取消 -> 
    springJobTask.setScheduledFuture(scheduledFuture);
    // 5. 将任务放入运行中的任务Map，方便管理和查找
    runningJobMap.put(job.getId(), springJobTask);
    // 6. 记录日志，方便追踪
    logger.info("启动 job 成功:{}", JSONUtil.toJsonStr(job));
}
/**
 * 删除（停止）一个定时任务
 * @param job 需要删除的任务对象
 */
private void deleteJob(Job job) {
    if (job == null) {
        return;
    }
    // 1. 从运行中的任务Map获取任务
    SpringJobTask springJobTask = this.runningJobMap.get(job.getId());
    if (springJobTask == null) { return;}
    // 2. 取消任务调度（停止定时执行）
    springJobTask.getScheduledFuture().cancel(false);
    // 3. 从Map中移除该任务
    runningJobMap.remove(job.getId());
    // 4. 记录日志
    logger.info("移除 job 成功:{}", JSONUtil.toJsonStr(job));
}
/**
 * 更新一个定时任务（先删除再启动）
 * @param job 需要更新的任务对象
 */
public void updateJob(Job job) {
    // 1. 先删除旧的任务
    this.deleteJob(job);
    // 2. 再启动新的任务
    this.startJob(job);
    // 3. 记录日志
    logger.info("更新 job 成功:{}", JSONUtil.toJsonStr(job));
}
```

#### 1.2.2 动态监控DB中job的变化

```java
/**
 * 监控db中job的变化，每5秒监控一次，这个频率大家使用的时候可以稍微调大点
 */
private void monitorDbJobChange() {
    this.threadPoolTaskScheduler.scheduleWithFixedDelay(this::jobChangeDispose, Duration.ofSeconds(5));
}
// 任务变化处理
private void jobChangeDispose() {
    //1、从db中拿到所有job，和目前内存中正在运行的所有job对比，可得到本次新增的job、删除的job、更新的job
    JobChange jobChange = this.getJobChange();
    //2、处理新增的job
    for (Job job : jobChange.getAddJobList()) { this.startJob(job);}
    //3、处理删除的job
    for (Job job : jobChange.getDeleteJobList()) { this.deleteJob(job);}
    //4、处理变化的job
    for (Job job : jobChange.getUpdateJobList()) { this.updateJob(job);}
}
private JobChange getJobChange() {
    //新增的job
    List<Job> addJobList = new ArrayList<>();
    //删除的job
    List<Job> deleteJobList = new ArrayList<>();
    //更新的job
    List<Job> updateJobList = new ArrayList<>();
    //从db中拿到所有job，和目前内存中正在运行的所有job对比，可得到本次新增的job、删除的job、更新的job
    List<Job> startJobList = this.jobService.getStartJobList();
    for (Job job : startJobList) {
        SpringJobTask springJobTask = runningJobMap.get(job.getId());
        if (springJobTask == null) {
            addJobList.add(job);
        } else {
            //job的执行规则变了
            if (jobIsChange(job, springJobTask.getJob())) {
                updateJobList.add(job);
            }
        }
    }
    //获取被删除的job，springJobTaskMap中存在的，而startJobList不存在的，则是需要从当前运行列表中停止移除的
    Set<String> startJobIdList = CollUtils.convertSet(startJobList, Job::getId);
    for (Map.Entry<String, SpringJobTask> springJobTaskEntry : runningJobMap.entrySet()) {
        if (!startJobIdList.contains(springJobTaskEntry.getKey())) {
            deleteJobList.add(springJobTaskEntry.getValue().getJob());
        }
    }
    //返回job变更结果
    return new JobChange(addJobList, updateJobList, deleteJobList);
}
/**
 * 检测两个job是否发生了变化，（cron、beanName、beanMethod）中有任意一项变动了，则返回true
 *
 * @param job1
 * @param job2
 * @return
 */
private boolean jobIsChange(Job job1, Job job2) {
    return !(Objects.equals(job1.getCron(), job2.getCron()) &&
            Objects.equals(job1.getBeanName(), job2.getBeanName()) &&
            Objects.equals(job1.getBeanMethod(), job2.getBeanMethod()));
}
```