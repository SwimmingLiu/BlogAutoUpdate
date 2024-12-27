---
title: "Java面试题-随手记"
date: 2024-12-27T17:27:35+08:00
lastmod: 2024-12-27T17:27:35+08:00
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

## == 和 equals 区别

`==` 基本类型(int, long, float, char, boolean) 值比较， 引用类型(String，List) 进行地址比较

`equals` 默认就是 `==` ，但是部分引用类型(String，List)重写了该方法，进行值比较

## get 和 post 区别

| 特性             | GET                              | POST                                         |
| ---------------- | -------------------------------- | -------------------------------------------- |
| **目的**         | 获取资源，查询数据               | 提交数据，创建或更新资源                     |
| **请求数据方式** | 参数通过 URL 查询字符串传递      | 数据通过请求体传递                           |
| **数据暴露**     | 数据暴露在 URL 中，较不安全      | 数据存储在请求体中，相对安全                 |
| **数据大小限制** | URL 长度有限制（约 2048 个字符） | 没有数据大小限制                             |
| **适用场景**     | 获取数据，查询，展示资源         | 提交表单，上传文件，修改资源，发送敏感数据等 |

## SpringMVC中@ReponseBody、@PathVariable、@RequestParameters在什么情况下使用?

`@ReponseBody` 用于接受请求体数据，一般用于POST请求

`@PathVariable` 用于接受路径参数，一般用户接受 `id` 

`@RequestParameters` 用于接受请求参数，一般用于GET请求

## JVM堆的结构、GC介绍和作用

[JVM堆结构的参考文章]( https://www.51cto.com/article/710705.html) 、 [GC垃圾回收过程](https://www.bilibili.com/video/BV1dt411u7wi)

| 区域                                         | 主要用途                       | 特点                                                 |
| -------------------------------------------- | ------------------------------ | ---------------------------------------------------- |
| **新生代（Young Generation）**               | 存储新创建的对象，快速垃圾回收 | 包含 Eden 区和两个 Survivor 区，采用复制算法进行回收 |
| **老年代（Old Generation）**                 | 存储长期存活的对象             | 回收频率较低，垃圾回收较耗时                         |
| **永久代（Permanent Generation) (jdk 1.7）** | 存储类的元数据、方法字节码等   | 在 jdk 1.8 被 Metaspace 替代                         |
| **元空间（jdk 1.8）**                        | 存储类的元数据                 | 不再属于堆，使用本地内存，大小由系统限制             |

`GC` 是垃圾回收器， 作用是自动内存管理和避免内存泄漏

## Innodb和MYISAM的特点

`InnoDB` 和 `MyISAM` 是 MySQL 数据库管理系统中的两种主要存储引擎。 `InnoDB` 是  MySQL 5.5 版本的默认引擎。

| 特性           | **InnoDB**                                                   | **MyISAM**                                                 |
| -------------- | ------------------------------------------------------------ | ---------------------------------------------------------- |
| **事务支持**   | 支持事务（ACID），提供 **COMMIT**, **ROLLBACK**, **SAVEPOINT** 等语句 | 不支持事务                                                 |
| **存储方式**   | 行级锁（Row-level locking）                                  | 表级锁（Table-level locking）                              |
| **支持外键**   | 支持外键约束（Foreign Key）                                  | 不支持外键约束                                             |
| **崩溃恢复**   | 支持崩溃恢复，自动恢复到一致性状态                           | 不支持崩溃恢复，数据可能会丢失                             |
| **性能特点**   | 适合高并发读写的应用，性能较慢（因为使用行级锁）             | 适合读多写少的应用，性能较快（因为使用表级锁）             |
| **数据完整性** | 数据完整性和一致性强，支持原子性操作                         | 不支持数据完整性，容易出现数据不一致                       |
| **缓存机制**   | 使用内存缓冲池来提高性能                                     | 使用内存缓存提高查询速度                                   |
| **适用场景**   | 高并发、需要事务和数据一致性的应用，如金融、银行等           | 读多写少、数据一致性要求较低的应用，如日志记录、数据仓库等 |

**行级锁（Row Lock）**：InnoDB使用行级锁，避免了对整张表加锁的性能问题，允许多个事务并发访问不同的行。

**表级锁（Table Lock）**：MySQL使用表级锁时，整个表会被锁定，导致其他事务无法访问表中的任何行，性能较低。

**意向锁（Intention Lock）**：行锁和表锁之间的一种锁，InnoDB用来处理行锁和表锁之间的兼容性问题。

## 如何理解数据库中的MVCC原理?

[MVCC原理解析](https://xie.infoq.cn/article/aada42abfebc550d053c01784)

## 脏读、幻读、不可重复读区别

[快速理解脏读、不可重复读、幻读和MVCC](https://cloud.tencent.com/developer/article/1450773)

## MySQL事务

### 事务特性 (AIDC)

**原子性（Atomicity）**：事务中的操作要么全部成功，要么全部失败。MySQL会保证即使发生崩溃或中断，未提交的事务会被回滚。

**隔离性（Isolation）**：事务的执行不应该被其他事务干扰。隔离级别控制了事务之间的“可见性”。

**持久性（Durability）**：一旦事务提交，变更将永久保存，即使系统崩溃，数据也不会丢失。

**一致性（Consistency）**：事务必须让数据库从一个一致的状态转换到另一个一致的状态，保证数据完整性。

### 事务的隔离级别

**读未提交**：事务可以读取到其他事务未提交的数据，可能导致脏读。

**读已提交**：事务只能读取已提交的事务数据，避免了脏读，但仍然可能出现不可重复读。

**可重复读**：事务在其生命周期内读取的数据始终一致，避免了脏读和不可重复读。可能会出现幻读。

**串行化**：最高的隔离级别，事务强制串行执行，避免脏读、不可重复读和幻读。但性能差，容易产生资源瓶颈。

## 如何优化数据库

### SQL优化

1.**避免使用 select ***

2.**索引优化**

- **索引使用**：创建适当的索引，尤其是常用于 `WHERE`、`ORDER BY`、`JOIN`、`GROUP BY` 中的字段。

- **覆盖索引**：尽量让查询可以通过索引直接返回结果，避免回表。

- **避免过多的索引**：索引虽能加速查询，但也会增加插入、更新的成本。

- **联合索引**：对于多个字段常一起查询的情况，可以考虑创建联合索引。

3.**避免全表扫描**

- 使用 `EXPLAIN` 来分析SQL执行计划，检查是否有全表扫描，并优化查询。

- 示例：`EXPLAIN SELECT * FROM orders WHERE user_id = ?` 可以检查是否使用了索引。

### 数据库架构优化

1.**分库分表**：

- 当数据量非常大时，可以通过分库分表来提高查询和插入的效率。
- 例如：按时间范围、ID范围或业务维度进行水平分表，按业务模块进行垂直分库。

2.**读写分离** (Redis)：

- 使用主从复制，主库处理写操作，从库处理读操作，减轻主库压力，提高系统的并发处理能力。
- 示例：主库执行 `INSERT`、`UPDATE`、`DELETE`，从库执行 `SELECT`。 (现在用Redis用来第一步判断，数据库后操作)

3.**数据库冗余与备份**：

- 增加数据库的冗余副本，定期进行备份，确保数据的高可用性。

### 缓存优化

1.**使用缓存来减轻数据库压力**：对频繁查询的数据进行缓存（如使用 Redis），比如热门商品信息。

2.**合理的缓存策略**：

**缓存穿透** : Redis 和 数据库都不存在，用空对象缓存到Redis

**缓存雪崩** : 

- **大量key同时过期**：设置随机的TLL

- **Redis服务宕机** : 设置Redis集群 + 哨兵模式 / 缓存业务降级限流 / 业务添加多级缓存

**缓存击穿**：Redis的热点Key过期，互斥锁 / **逻辑过期**

3.**避免缓存不一致**：使用双写策略（更新数据库同时更新缓存），或通过异步更新缓存来避免缓存与数据库数据不一致。

### 数据库连接池

使用连接池（如 HikariCP、Druid）可以有效地减少数据库连接的开销，避免频繁创建和销毁连接。

### 数据库并发控制

1.**事务与锁机制优化**

- 通过合理的事务管理和锁策略，避免死锁和性能瓶颈。
- 控制事务的粒度，避免长时间持有锁，减少锁竞争。

2.**乐观锁与悲观锁**：

- 在适当的场景下使用乐观锁（如使用版本号），避免对数据加锁。
- 在高并发下，使用悲观锁（如数据库行级锁、悲观锁）确保数据一致性。

## Redis一主两从+哨兵模式

Redis 采用**一主两从+哨兵**的集群方案，主要是为了在高并发和高可用的场景下保证系统的稳定性和可靠性。**主节点(master)**处理**写**请求，**从节点(slave1 和 slave2)**处理**读**请求，**减少主节点的压力**；哨兵机制监控**主节点和从节点**，能够自动进行故障转移，确保 Redis 集群在节点故障时的高可用性。

**普通的主从模式，当主数据库崩溃时，需要手动切换从数据库成为主数据库**，这就需要人工干预，费事费力，还会造成一段时间内服务不可用，即存在高可用问题。我们又使用了哨兵。**哨兵**是一个**独立的进程**，作为进程，它会独立运行。其原理是**哨兵通过发送ping命令，等待Redis服务器响应，从而监控运行的多个Redis实例。**哨兵可以实现**自动故障修复**，当**哨兵监测到master宕机**，会自动将**slave**切换成**master**，然后**通过发布订阅模式通知其他的从服务器**，修改配置文件，**让它们切换master**。同时那台有问题的**旧主节点**也会变为新主节点的从节点，也就是说当**旧的主即使恢复时**，并不会**恢复原来的主身份**，而是作为**新主的一个从**。

## OAuth2.0 / 单点登录 (SSO)

[OAuth2.0 介绍](https://blog.51cto.com/u_15344989/4964824)、[单点登录SSO介绍](https://developer.aliyun.com/article/636281)

![Spring Security OAuth2 四种认证模式（含流程图）_spring](https://oss.swimmingliu.cn/b2d1d0db-c435-11ef-abfb-c858c0c1deba)

![cas_flow_diagram](https://oss.swimmingliu.cn/b2fd3d99-c435-11ef-80ac-c858c0c1deba)

## 如何防注入攻击？

1.**使用预编译语句（Mybatis）**： 将 SQL 语句与参数分离，防止恶意输入被解释为代码

2.**最小权限原则**: 为指定数据库用户分配最小必要的权限，限制其只能执行特定操作，减少潜在的安全风险。

3.**避免动态拼接SQL** ( XML里面 `#` 和 `$` 区别): 使用`#`，而不用 `$` 防止直接拼接

## 如何设计表的映射关系

> 有一个订单表，一个产品表，一个产品订单表

**订单表（Order）**：存储订单的基本信息，如订单ID、用户ID、订单日期、订单状态等。

**产品表（Product）**：存储产品的详细信息，如产品ID、产品名称、产品价格、库存数量等。

**订单项表（Order_Item）**：用于表示订单与产品之间的多对多关系，记录每个订单中包含的产品及其数量、价格等信息。

## 哪些表需要加索引？对哪些字段加索引?

### 需要索引的表

对于记录数较多的表，建立索引可以显著提升查询性能。

### 需要加索引的字段

1.**主键字段**：主键字段默认建立唯一索引，确保记录的唯一性和快速定位。

2.**经常作为查询条件的字段**：在 `WHERE` 子句中频繁出现的字段，特别是在大表中，应建立索引，以提高查询效率。

3.**用于表连接的字段**：在与其他表进行连接操作时，连接字段应建立索引，以加速连接操作。

4.**用于排序（`ORDER BY`）和分组（`GROUP BY`）的字段**：这些字段建立索引后，可提高排序和分组操作的性能。

5.**选择性高的字段**：选择性高意味着字段值的唯一性较高。在此类字段上建立索引，可以有效过滤数据，提高查询效率。

下面是不需要加索引的

> **避免在频繁更新的字段上建立索引**：因为每次更新不仅要修改数据，还需维护索引，可能影响写操作性能。 (商品库存，商品信息)
>
> **避免在低选择性字段上建立索引**：如性别字段，只有 "男" 和 "女" 两种值，建立索引效果不明显。
>
> **控制索引数量**：过多的索引会增加数据库的维护成本，特别是在频繁写操作的场景下，需要在查询性能和写操作开销之间找到平衡点。

## 组合索引 / 复合索引 / 联合索引

> 像组合索引和复合索引你知道吗?比如说我建了一个a,b,c联合索引。我写代码的时候先写的c，b，a可以吗?

**组合索引**（也称复合索引或联合索引）是指在多个列上创建的单个索引，用于提高多条件查询的性能。

当创建了包含列 `a`、`b`、`c` 的组合索引时，查询条件的顺序会影响索引的使用效果。

这遵循 **最左前缀原则**，即索引的使用从最左边的列开始匹配，必须按照索引定义的列顺序进行匹配。

所以不能使用 `c`，`b` ，`a` 查询。

## MySQL的索引在什么条件下会失效?

- **对索引字段的运算或函数操作**

  ```sql
  SELECT * FROM users WHERE YEAR(birthdate) = 1990;
  ```

- **使用了通配符（`LIKE '%abc'`）**

  ```sql
  SELECT * FROM products WHERE name LIKE '%abc';
  ```

- **在 WHERE 子句中使用了不等于（`<>`）操作符**

  ```sql
  SELECT * FROM orders WHERE status <> 1;
  ```

- **进行 NULL 值查询时未优化**

  ```sql
  SELECT * FROM users WHERE phone_number IS NULL;
  ```

- **数据类型不匹配**

  ```sql
  SELECT * FROM users WHERE age = '25';
  ```

- **使用了 OR 操作符，特别是跨字段时**

  ```sql
  SELECT * FROM products WHERE price = 100 OR category = 'electronics';
  ```

- **索引列的数据分布不均匀**: 当索引列的数据分布极其不均匀时，即使索引可以使用，MySQL 也可能选择不使用索引，因为扫描全表比扫描索引更高效。

## 事务放在MVC当中的哪一层？

事务的管理应放在 **Service层**，这是因为Service层负责业务逻辑，可以统一控制跨多个DAO操作的事务。Controller层应该尽量避免直接管理事务，以保持系统的解耦性和职责清晰。而DAO层则专注于数据的持久化操作，不应承担事务管理的责任。

## 什么时候应该添加事务？

>那我们查询list会加事物吗? 使用delete删除时会加事物吗?

**事务的使用场景**：事务通常用于数据修改操作（如插入、更新、删除），确保数据一致性。

**查询操作的事务性**：查询操作一般不需要事务，除非有一致性需求，需通过隔离级别来控制。

**增、删、改操作的事务性**：增、删、改应该放在事务中进行，确保增、删、改的原子性。

## 并发修改请求如何控制?

> 假设我的银行卡里面只有10块钱，现在过来10个请求都要扣10块钱，是你的话你会怎么控制?

最直接的方式是保证每个扣款操作具有**原子性**。可以通过**悲观锁**或**乐观锁**来控制并发，确保多个请求不会同时扣款。

1.**悲观锁和乐观锁**

- **悲观锁**通过数据库层面的锁机制（如 `SELECT FOR UPDATE`）防止并发修改余额。

- **乐观锁**通过版本号或 CAS 原理进行操作，适用于并发冲突较少的情况。

2.**分布式锁** （`Redission`）

在分布式系统中，可以使用**分布式锁**来保证对共享资源的独占访问。

3.**队列和限流** (`RabbitMQ`)

**队列和限流** 控制请求的处理速度，避免突发的高并发请求。

## 分布式事务如何处理？它的作用?

**为什么需要分布式事务**：分布式事务用于解决分布式系统中不同服务之间的数据一致性问题，确保跨服务操作能够保证最终的一致性。

**分布式事务解决什么问题**：主要解决**跨服务的一致性**，保证多个服务中的操作要么全部成功，要么全部回滚，保持数据一致性。

**分布式事务的解决方案**：（后期学习）

- **2PC**：适合强一致性要求的场景。
- **TCC**：用于操作更复杂的分布式场景。
- **Saga模式**：适合长事务的分布式事务，通常用于最终一致性场景。

## 开发过程中Git分支管理

1.开发的时候来了一个新需求，你们的分支是怎么管理的? 

我们采用三种不同的前缀来管理 `feat` 新增、`fix` 修复、`refactor` 重构

2.增加新需求，分支是从哪里新增的?

每个新需求都会从 `main` 分支上创建一个 `feat` 分支进行开发。

3.建完一个分支之后就开始改代码吗?

确保你从主分支（`main`）创建了最新的分支，避免后续合并时的冲突。

在开发前，应该确认需求的具体内容，并与相关团队成员对接，然后配置好对应环境，再进行开发。

4.开发完之后测试的话，这个代码放到哪里去?

我们会commit `feat` 分支，然后打包上传到测试环境，提交给测试

5.假如现在又来了两个新需求，一个需求先上线，一个需求过几天上线。你们的分支是怎么管理的?

当有多个需求时，先满足第一个要上线的 `需求A` 。然后再拉取代码，编写 `需求B` 的代码

6.你们那个测试拉代码是运维拉代码还是测试拉代码？

测试拉取代码

## Springboot和Spring区别及理解

**Spring**：

- **核心特性**：Spring 提供了控制反转（IoC）和面向切面编程（AOP）的支持，主要用于构建企业级应用程序。
- **配置方式**：Spring的配置是非常灵活但复杂的。通常，开发者需要通过 XML 配置、注解配置或 Java 配置类来配置 Spring 容器和各种模块。
- **模块化**：Spring 框架分为多个模块，如 Spring Core、Spring AOP、Spring Data、Spring MVC 等，开发者需要根据需要集成和配置这些模块。

**Spring Boot**：

- **自动配置**：Spring Boot 提供了大量的自动化配置，开发者只需要少量的配置，甚至可以省去 XML 配置，简化了传统 Spring 配置的复杂度。
- **内嵌服务器**：Spring Boot 提供了 Tomcat、Jetty、Undertow 等内嵌服务器，使得应用可以直接打包为可执行 JAR 文件，无需外部容器支持。
- **开箱即用**：Spring Boot 提供了很多开箱即用的功能，例如，默认的应用结构、内置的健康检查、自动化的日志配置等，极大提高了开发效率。
- **Spring Boot Starter**：使用“Starter”可以让开发者方便地引入常见的依赖包，避免手动配置和集成常用组件。

### Spring和Springboot区别

| **特性**     | **Spring**                                                 | **Spring Boot**                                    |
| ------------ | ---------------------------------------------------------- | -------------------------------------------------- |
| **目标**     | 提供企业级应用开发框架。                                   | 简化 Spring 应用的配置和启动过程。                 |
| **配置方式** | 需要大量的 XML 或 Java 配置。                              | 通过自动配置简化配置，几乎不需要手动配置。         |
| **应用启动** | 需要外部应用服务器（如 Tomcat）。                          | 支持内嵌服务器，应用可以打包为可执行 JAR 文件。    |
| **依赖管理** | 需要手动配置依赖和版本。                                   | 提供预设的 **Spring Boot Starter**，自动管理依赖。 |
| **模块集成** | 需要开发者手动集成各个模块（如 Spring MVC, Spring Data）。 | 自动化集成各个模块，并提供开箱即用的功能。         |
| **开发效率** | 配置和集成较为繁琐，开发效率较低。                         | 通过自动配置和开箱即用的功能，大大提高开发效率。   |
| **启动时间** | 启动速度较慢，需要等待容器的初始化。                       | 启动速度快，集成化和内嵌式服务减少了初始化时间。   |
| **环境依赖** | 配置较为灵活，环境依赖需要手动管理。                       | 内嵌服务器和自动配置降低了环境依赖问题。           |

## RabbitMQ和Kafka区别和理解

[Kafka 和 RabbitMQ 对比 - 原理](https://yindongliang.com/posts/compare-kafka-and-rabbitmq/)

| **应用场景**           | **RabbitMQ**                                                 | **Kafka**                                                   |
| ---------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| **消息大小和格式**     | 适合处理中小型消息，支持各种消息格式，包括文本、JSON等       | 适合处理大型消息和流式数据，支持批量消息传递和日志存储      |
| **实时性要求**         | 支持低延迟的消息传递和定时功能，适用于实时消息处理 （微秒级） | 延迟较高，主要优化吞吐量，适合高吞吐量数据流处理 （毫秒级） |
| **吞吐量**             | 吞吐量较低，适合低并发场景                                   | 吞吐量极高，适用于高并发、大数据场景                        |
| **数据一致性和可靠性** | 提供消息确认机制和强大的持久化功能，保证消息不丢失           | 通过日志存储、分区和副本机制保证数据可靠性和高可用性        |
| **分布式系统支持**     | 支持集群模式，但在扩展性上不如 Kafka                         | 原生支持分布式架构，适用于大规模分布式系统                  |
| **插件支持和生态系统** | 丰富的插件支持，易与各种技术和工具集成，功能多样化           | 插件支持较少，但生态系统广泛，主要用于流处理、大数据平台    |
