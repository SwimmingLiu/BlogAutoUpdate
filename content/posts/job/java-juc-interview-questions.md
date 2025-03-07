---
title: "(JUC) Java并发面试题笔记"
date: 2025-03-05T21:45:49+08:00
lastmod: 2025-03-05T21:45:49+08:00
author: ["SwimmingLiu"]

categories:
- 💻 Job

tags:
- Java
- JUC

keywords:
- Java
- JUC

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

# (JUC) Java并发面试题笔记

## 1. 什么是 Java 内存模型（JMM）？

`JMM` 是 Java内存模型 ， 它是Java虚拟机 `JVM` 定义的一种规范，用于描述多线程程序中的变量，像实例字段、静态字段和数组元素，他们如何在内存中存储和传递的规则。也就是规定线程啥时候从住内存里面读数据，啥时候把数据写回到住内存。`JMM` 的核心目标是确保多线程环境下的**可见性**、**有序性**和**原子性**, 避免硬件和编译器优化带来的不一致问题。

**【可见性、有序性、原子性定义】**

- **可见性**：确保一个线程对共享变量的修改，能够及时被另外一个线程看到。 `volatile`就是用来保证可见性的，强制线程每次读写的时候，直接从主内存当中获取最新值。
- **有序性**：指线程执行操作的顺序。JMM允许某些指令重排序之后再提高性能，但会保证线程内的操作顺序不会被破坏。通过`happens-before` (线程A发生在线程B之前)的关系来保证跨线程的有序性。
- **原子性**：操作不可分割，线程不会在执行的过程当中被打断。例如, `synchronize` 关键字能确保方法或代码块的原子性

**【JMM作用】**

因为不同的操作系统都有一套独立的内存模型，但是Java为了满足跨平台的特性，它需要定义一套内存模型屏蔽个操作系统之间的差异。我们可以利用JMM当中定义好的从Java源码到CPU指令的执行规范，也就是使用`synchronized` 、`volatile` 等关键字，还有`happens-before`原则，就可以写出并发安全的代码了。
比如说，线程A和线程B同时操作 `变量-1`，假如最开始`变量-1` 是 `0`

- 首先，线程A和线程B都读取了`变量-1`
- 然后，线程B对取到的`变量-1`自增为`1`，并写回主内存
- 此时，线程A对读取到的`变量-1`也自增`1`，并写回主内存。这就会导致线程B的操作失效了，出现并发安全问题。

如果有JMM，我们就可以在线程A要修改数据之前,让它采用CAS乐观锁的方式进行修改。再次去读主内存当中的值，然后修改之后，再判断一下主内存的值是否发生变化。如果没有发生变化，就写回主内存。如果发生变化，就要进行自旋。

**【注意】** 工作内存是每个线程独立的内存空间，其他线程都是看不到的。主内存是Java堆内存的一部分，所有的实例变量、静态变量和数组元素都存储在主内存当中。

![JMM架构图](https://oss.swimmingliu.cn/49ddf552-f9c8-11ef-99c5-c858c0c1deba)

**【内存间交互操作类型 (8种原子操作)】**

1. **lock 上锁**：把一个变量表示为一条线程独占的状态
2. **unlock 解锁**： 把一个变量从独占状态中释放出来，释放后的变量才能被其他线程锁定
3. **read 读取**： 从主内存当中读取一个变量到工作内存中
4. **load 载入**：把`read`操作从主内存中得到的变量值放入工作内存的变量副本当中
5. **use 使用**：把工作内存当中的一个变量值传递给执行引擎
6. **assign 赋值**：把一个从执行引擎接收到的值赋给工作内存中的变量
7. **store 存储**：把工作内存中的一个变量的值传送给主内存中
8. **write 写入**：把store操作从工作内存中得到的变量值放入主内存的变量中

**【volatile 特殊规则】**

- **可见性**：对于 `volatile` 修饰的变量的写操作会立即刷新到内存中，任何线程对这个`volatile` 变量的读操作都能立即看到最新的值。
- **禁止指令重排序**： 在对 `volatile` 变量进行读/写操作的时候，会插入内存屏障，禁止指令重排序。也就是该变量的写操作不能与之前的读/写操作重排序，它的都操作不能与之后的读/写操作重排序。

**【Happens-Before 原则】**

见下一个问题

## 2. 什么是Java的 happens-before 规则?

`happens-before` 原则就是 `JMM` 当中定义操作间顺序的规则，确保操作的有序性和可见性。

1. **程序次序规则**：线程当中所有操作都是按程序代码的顺序发生
2. **监视器锁规则**：解锁操作发生在同一个锁的随后的加锁操作之前，说白了，先解锁，后上锁
3. **`volatile` 变量规则**： `volatile` 变量的写操作发生在对改变量随后的读操作之前，先写后读
4. **线程启动规则**：线程A对线程B的`Thread.start()` 调用发生在这个新线程的每一个操作之前
5. **线程终止规则**：线程A所有的操作都发生在其他线程检测到线程A终止之前
6. **线程中断规则**：对线程的中断操作 (`Thread.interrupt()`) 发生在被中断线程检测到的中断时间之前 (通过`Thread.interrupted()` 或 `Thread.isInterrupted()` 进行检测)
7. **对象终结规则**： 一个对象的构造函数执行结束发生在这个对象的`finalize()` 方法之前
8. **传递性**： 如果A `happens-before` B，B `happens-before` C, 则 A `happens-before` C

## 3.  Java 的 synchronized 是怎么实现的？

**【实现原理】** 

`synchronized` 关键字是以来 `JVM` 的Monitor (监视器锁)和 Object Header (对象头) 实现的。其中，重量级所依赖于 `Monitor` 监视器锁， 轻量级锁和偏向锁都依赖于对象头 (Object Header)

**【不同使用场景的区别】**

- **修饰方法**：会在修饰的方法的访问标志中增加一个 `ACC_SYNCHRONIZED` 标志，每当有一个线程访问该方法时， JVM回去检测该方法的访问标志。如果包含`ACC_SYNCHRONIZED` 标志， 线程必须获得该方法对应的对象监视器锁 (对象锁)，也就是`Monitor` 当中的`owner`执行该线程。 然后再执行该方法，保持同步性。
- **修饰代码块**：在代码块的前后分别插入 `monitorenter` 和 `monitorexit` 字节码指令， 可以理解为加锁和解锁

**【可重入性】** 

`synchronized` 是可以重入的，每次获取一次锁。如果是当前线程的锁，计数器加`1`，锁释放的时候，计数器减`1`。直到计数器减为 `0` 为止，锁才真正释放

**【`synchronized` 锁升级】**

`synchronized` 锁分为三种，偏向锁，轻量级锁，重量级锁。其升级次序为偏向锁(一个线程) -> 轻量级锁 (多个线程) -> 重量级锁 (多个线程竞争激烈)

![Synchronized对象头](https://oss.swimmingliu.cn/4a286ce8-f9c8-11ef-b127-c858c0c1deba)

- **偏向锁**：当有线程第一次获取锁的时候， `JVM` 会采用修改锁对象的对象头，将该线程标记为偏向状态。对象头里面会记录线程`ID` 和 对应的`epoch` 偏向锁版本。后续该线程再获取锁，基本没啥开销。
- **轻量级锁**：当有另外的线程尝试去获取已经被偏向的锁时，锁会升级为轻量级锁。此时，申请上锁的过程中，JVM会在当前线程的栈帧中创建一个锁记录`Lock Record`，让锁记录指向锁对象。然后用`CAS` 替换锁对象的`Mark Word`， 并将`Mark Word` 的值存入锁记录。如果替换成功，锁对象的`Mark Word` 就变成当前线程的所锁记录。使用 `CAS` 操作的目的是减少锁竞争的开销。
- **重量级锁**：当 `CAS` 失败无法获取锁的时候，JVM判定其为多个线程竞争锁激烈。锁会升级成为重量锁，会使用操作系统的互斥量 `Mutex` 来实现线程的阻塞和环形。若果获取锁成功，线程会被放入Monitor的 `owner` 当中

![Monitor结构](https://oss.swimmingliu.cn/4a41e56d-f9c8-11ef-8eab-c858c0c1deba)

**【锁消除和锁粗化】**

- **锁消除**：JVM判断对象是否只在当前线程使用，如果只在当前线程使用，则消除不必要加的锁
- **锁粗化**：多个锁频繁操作出现时，JVM会将这些锁操作合并，见锁获取和释放的开销。

**【参考材料】** [黑马程序员-synchronized锁升级](https://blog.csdn.net/qq_38246328/article/details/124654374)

## 4. AQS了解过吗?

`AQS` (`Abstract Queued Synchronizer`简称，抽象的队列式同步器), 它其实就是把跟锁相关的操作进行抽象和封装的一个工具类。将排队、入队、加锁、中断这些方法提供出来，方便其他相关`JUC` 锁的使用，具体的加锁时机、入队时机都需要实现类自己控制。
`AQS` 通过维护一个共享状态 `state` 和 一个先进先出 `FIFO`  的等待队列，来管理线程对共享资源的访问。共享状态 `state` 用 `volatile` 修饰，表示当前资源的状态，保证了可见性和有序性。在独占锁中 (`ReentrantLock` 或者 `synchronized`), `state` 为 `0` 表示未被占用，为`1` 表示已被占用。假如当前共享资源已经被占用，线程获取资源失败，会被加入到`AQS` 的等待队列里面。这个队列是一个变体的 `CLH` 队列， 采用双向链表结构，其节点包含线程的引用、等待状态、前驱节点和后继节点的指针。`AQS` 的常见实现类有 `ReentrantLock`、`CountDownLatch` 、`Semaphore` 等等
其中，`CLH` 队列是一种基于链表的、公平的自旋锁算法，主要用于多线程环境中实现锁的公平分配。

![AQS结构](https://oss.swimmingliu.cn/f147d41b-fb59-11ef-9d36-c858c0c1deba)

**【AQS 工作流程】** 公平锁和非公平锁的区别就在于，公平锁是直接放到队里面，不会主动去尝试 `CAS` 获取锁

1. 先进先出 `FIFO` 队列用来实现多线程的排队工作，当线程加锁失败时，就把这个线程封装成一个`Node` 节点置于队列尾部

![AQS队列的加锁过程](https://oss.swimmingliu.cn/f2067cce-fb59-11ef-8306-c858c0c1deba)

2. 当持有锁的线程释放锁的时候，`AQS` 会将等待队列中第一个线程唤醒，并让它重新尝试获取锁

![AQS队列的加锁过程-有线程持有锁](https://oss.swimmingliu.cn/f21cfc0f-fb59-11ef-a390-c858c0c1deba)

**【同步状态 - state】**

`AQS` 使用一个`volatile` int类型的成员变量`state`来表示同步状态， 当`state = 0` 的时候表示没有锁，当`state = 1` 的时候表示当前对象锁已经被占有了。提供了三种`API`， 分别用于获取状态、设置状态和采用`CAS` 更新状态

```java
// 同步状态
private volatile int state;
// 获取状态
protected final int getState() {
    return state;
}
// 设置状态
protected final void setState(int newState) {
    state = newState;
}
// CAS更新状态
protected final boolean compareAndSetState(int expect, int update) {
    // See below for intrinsics setup to support this
    return unsafe.compareAndSwapInt(this, stateOffset, expect, update);
}
```

**【FIFO队列 - Node】**

上面`AQS` 工作流程提到的`Node` 节点如下， `Node` 节点里面包含`waitStatus` 用于标记节点状态， `thread`  线程指针指向当前节点所代表的线程，还有前驱节点和后继结点指针。

```java
// Node类用于构建队列
static final class Node {
    // 标记节点状态。常见状态有 CANCELLED（表示线程取消）、SIGNAL（表示后继节点需要运行）、CONDITION（表示节点在条件队列中）等。
    volatile int waitStatus;
    // 前驱节点
    volatile Node prev;
    // 后继节点
    volatile Node next;
    // 节点中的线程，存储线程引用，指向当前节点所代表的线程。
    volatile Thread thread;
}

// 队列头节点，延迟初始化。只在setHead时修改
private transient volatile Node head;
// 队列尾节点，延迟初始化。
private transient volatile Node tail;

// 入队操作
private Node enq(final Node node) {
    for (;;) {
        Node t = tail;
        if (t == null) { // 必须先初始化
            if (compareAndSetHead(new Node()))
                tail = head;
        } else {
            node.prev = t;
            if (compareAndSetTail(t, node)) {
                t.next = node;
                return t;
            }
        }
    }
}
```

**【同步队列和条件队列】**

`AQS` 有两种 `FIFO` 队列，同步队列和条件队列。同步队列用于锁的获取和释放，条件队列用于特定条件下管理线程的等待和唤醒。常用的`ReentrantLock` 就是基于同步队列实现的。

- **同步队列的实现原理**：当一个线程尝试获取锁失败的时候，`AQS` 会将该线程包装成一个 `Node` 节点加入到队列的尾部。这个节点会处于等待状态，直到所资源被其他线程释放。当锁被释放的时候，持有锁的线程会通知其他后继结点(如果存在的话)，后继结点会尝试获取锁，这个过程一直持续到线程成功获取锁或队列为空

  ```java
  private Node addWaiter(Node mode) {
      Node node = new Node(Thread.currentThread(), mode);
      // 尝试快速路径：直接尝试在尾部插入节点
      Node pred = tail;
      if (pred != null) {
          node.prev = pred;
          if (compareAndSetTail(pred, node)) {
              pred.next = node;
              return node;
          }
      }
      // 快速路径失败时，进入完整的入队操作
      enq(node);
      return node;
  }
  private Node enq(final Node node) {
      for (;;) {
          Node t = tail;
          if (t == null) { // 队列为空，初始化
              if (compareAndSetHead(new Node()))
                  tail = head;
          } else {
              node.prev = t;
              if (compareAndSetTail(t, node)) {
                  t.next = node;
                  return t;
              }
          }
      }
  }
  ```

- **条件队列的实现原理**：条件队列主要用于实现条件变量，实现了线程间的协调和通信。允许线程在特定条件不满足的时候挂起，等到其他线程改变了条件并显示的唤醒等待在该条件队列上的线程，典型的条件队列使用场景就是`ReentrantLock` 的 `Condition`

  ```java
  public class ConditionObject implements Condition, java.io.Serializable {
      // 条件队列的首尾节点
      private transient Node firstWaiter;
      private transient Node lastWaiter;
      // ...
  }
  // ConditionObject是AQS的一个内部类，用于实现条件变量。条件变量用于线程间通信，允许一个或多个线程在特定条件成立之前等待，同时释放先关的锁。
  public final void await() throws InterruptedException {
      // 如果当前线程在进入此方法之前已经被中断了，则直接抛出InterruptedException异常。
      if (Thread.interrupted())
          throw new InterruptedException();
      
      // 将当前线程加入到等待队列中。
      Node node = addConditionWaiter();
      
      // 释放当前线程所持有的锁，并返回释放前的状态，以便以后可以重新获取到相同数量的锁。
      int savedState = fullyRelease(node);
      
      // 中断模式，用于记录线程在等待过程中是否被中断。
      int interruptMode = 0;
      
      // 如果当前节点不在同步队列中，则表示线程应该继续等待。
      while (!isOnSyncQueue(node)) {
          // 阻塞当前线程，直到被唤醒或中断。
          LockSupport.park(this);
          
          // 检查线程在等待过程中是否被中断，并更新interruptMode状态。
          if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
              break;
      }
      
      // 当节点成功加入到同步队列后，尝试以中断模式获取锁。
      // 如果在此过程中线程被中断，且不是在signal之后，则设置中断模式为REINTERRUPT。
      if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
          interruptMode = REINTERRUPT;
      
      // 如果节点后面还有等待的节点，从等待队列中清理掉被取消的节点。
      if (node.nextWaiter != null) // clean up if cancelled
          unlinkCancelledWaiters();
      
      // 根据中断模式处理中断。
      if (interruptMode != 0)
          reportInterruptAfterWait(interruptMode);
  }
  ```

  当线程调用了`Condition` 的 `await()` 方法后，它会释放当前持有的锁，并且该线程会被加入到条件队列中等待。直到被另一个线程的`signal()` (唤醒等待队列中头节点对应的线程) 或者 `signalAll()`（唤醒所有等待的线程）方法唤醒或者被中断。

  ```java
  public final void signal() {
      if (!isHeldExclusively())
          throw new IllegalMonitorStateException();
      Node first = firstWaiter;
      if (first != null)
          doSignal(first);
  }
  private void doSignal(Node first) {
      do {
          if ( (firstWaiter = first.nextWaiter) == null)
              lastWaiter = null;
          first.nextWaiter = null;
      } while (!transferForSignal(first) &&
               (first = firstWaiter) != null);
  }
  ```

**【`ReentrantLock`、`CountDownLatch`、`Semaphore` 区别】**

| **对比维度** | **ReentrantLock**                | **CountDownLatch**                 | **Semaphore**                          |
| ------------ | -------------------------------- | ---------------------------------- | -------------------------------------- |
| **用途**     | 互斥访问共享资源，支持可重入锁   | 等待多个线程完成操作后触发后续任务 | 控制同时访问共享资源的线程数量（限流） |
| **同步方式** | 独占模式（Exclusive）            | 共享模式（Shared）                 | 共享模式（Shared）                     |
| **资源管理** | 通过 `state` 记录锁重入次数      | 通过 `state` 表示剩余需等待的计数  | 通过 `state` 表示可用许可证数量        |
| **释放机制** | 需显式调用 `unlock()` 释放锁     | 自动触发（`countDown()` 减至 0）   | 需显式调用 `release()` 归还许可        |
| **可重用性** | 可重复加锁/解锁（可重入）        | 一次性使用（计数归零后不可重置）   | 可重复获取/释放许可证                  |
| **线程协作** | 基于条件变量 `Condition` 控制    | 无条件协作，仅等待计数归零         | 无协作，仅控制并发量                   |
| **典型场景** | 替代 `synchronized` 的显式锁控制 | 主线程等待子线程初始化完成         | 数据库连接池、限流场景                 |

**【参考材料】**

1. [从ReentrantLock的实现看AQS的原理及应用](https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html)
2. [【基本功】不可不说的Java“锁”事](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651749434&idx=3&sn=5ffa63ad47fe166f2f1a9f604ed10091&chksm=bd12a5778a652c61509d9e718ab086ff27ad8768586ea9b38c3dcf9e017a8e49bcae3df9bcc8&scene=38#wechat_redirect)

## 5. Java 中 ReentrantLock 的实现原理是什么？

`ReentrantLock` 是基于 `AQS` 实现的一个可重入锁，支持公平锁和非公平锁两种方式。

【`ReentrantLock` 结构】

- **一个 `state` 变量**: 用于表示锁的状态, `state = 0` 表示没有锁占用，`state = 1` 表示已经被占用
- **同步队列**：用于锁的获取和释放。对于非公平锁，没有获取到锁的就会被放在这个队列里面。对于公平锁，所有的线程都是直接放到这个队列，依次等待获取锁。
- **条件队列**：用于等待某个条件满足了之后，才让这些线程逐个获取锁。比如指定所有的线程必须在晚上2点开始扫描检查是否存在异常订单的情况，必须要等到晚上2点，这些线程才开始获取锁。

![ReentrantLock工作流程](https://oss.swimmingliu.cn/f23cc434-fb59-11ef-8ca6-c858c0c1deba)

**【CLH队列】**

`CLH` 是一种基于队列的自选锁，它适用于多处理器环境下的高并发场景。原理是通过维护一个**隐式队列**，让线程在等待锁的时候自旋在本地变量上，从而减少对共享变量的争用和缓存一致性流量。
它将争抢的线程组织成一个队列，通过排队的方式按序争抢锁。每个线程不再`CAS` 争抢一个变量，而是自选判断排在它前面的线程的状态，如果前面的线程状态为释放锁，那么后续的线程就抢锁。

![CLH队列结构](https://oss.swimmingliu.cn/f25c2db6-fb59-11ef-8a71-c858c0c1deba)

`CLH` 通过排队按序争抢解决了锁饥饿的问题 (锁饥饿就是有的线程长期抢不到锁)，通过 `CAS` 自选监听前面现成的状态避免总线风暴问题的产生。总线风暴是指短时间内大量并发CAS操作和缓存一致性协议产生的总线流量激增，会导致CPU利用率下降，内存访问延迟增加。 不过, `CLH` 的自旋期间线程会一直占用CPU资源，只适合锁等待比较短的场景。

**【AQS对CLH进行改造】**

为了防止`CLH` 自旋期间长期占用CPU资源的问题，`AQS`将自旋等待前置节点改成了阻塞线程。后续的线程没法儿主动去发现前面的线程是否释放了锁，只能等待前面线程通知后续线程锁被释放了，它采用去争夺锁。所以，`AQS` 把 `CLH` 改成了一个双向队列，让前面的线程可以通知后续的线程。如果后面的线程等待超时或者主动取消，则从队列中移除，后面的线程顶上来。

![AQS改进CLH结构工作流](https://oss.swimmingliu.cn/f27716e4-fb59-11ef-836d-c858c0c1deba)

## 6. 什么是 Java 的 CAS（Compare-And-Swap）操作？

`CAS` 是一种硬件级别的原子操作，用于实现无锁并发编程，使用比较和交换的方式确保线程安全。`CAS` 的作用就是保证无锁并发，而且`CAS`操作是原子的，保证了线程安全。

**【CAS过程】** 

1. **比较**：`CAS` 在每次操作之前，会先检查内存当中的某个值是否与预期值相等
2. **交换**：如果发现同预期值相同，则将内存中的值更新为新值
3. **自旋重试**：如果发现和预期值不相同，说明其他线程已经修改了该值。它会在一定次数内，不断获取最新的内存值，更新预期值，然后再次尝试 `CAS` 操作。

**【CAS存在的问题】**

1. **ABA问题**：`CAS` 操作中，如果一个变量值被其他线程从A变成B，然后又变回A. `CAS` 无法检测到这种变化，可能会导致错误。
2. **自旋开销**：`CAS` 操作通常是通过自旋完成的，可能会导致CPU资源浪费，尤其是在高并发的情况下。
3. **单变量控制**：`CAS` 操作只适用单个变量的更新，不能涉及多个变量的复杂操作。
4. **总线风暴**：如果`CAS`修改同一个变量的并发很高，可能会导致总线风暴。

**【总线风暴】**

总线风暴是指短时间内大量并发CAS操作和缓存一致性协议产生的总线流量激增，会导致CPU利用率下降，内存访问延迟增加。因为`lock` 前缀指令会把写缓冲区的所有数据立即刷新到主内存中，在对称多处理架构下，每个CPU都会通过嗅探总线来检查自己的缓存是否过期。如果某个CPU刷新自己的数据到驻村，就会通过总线通知其他CPU过期对应的缓存，实现内存屏障，保证一致性。
通过总线来回通信成为`Cache` 一致性流量，如果这个流量过大，总线就会成为瓶颈，导致本地缓存更新延迟。 所以，如果 `CAS`  修改同一个变量并发很高，就会导致总线风暴，出现性能瓶颈。

## 7. 什么是 Java 中的 ABA 问题？

`ABA` 问题是指多线程环境下，某个变量的值在一顿时间内经历了从 `A` 到 `B` 再到 `A` 的变化，这种变化可能会被`CAS` 判定为值没有变化，从而导致线程发生错误的判断和操作。

**【ABA线程示例】**

下面的代码当中，如果线程A读取了`oldHead`， 此时另外一个线程B修改了栈的内容，然后将 `oldHead` 移除 (将栈顶元素从`A` 变成 `B` ， 再变成`A`)， 当线程A再次执行 `compareAndSet` 函数的时候，尽管值是一样的(`oldHead`没有变化)，但实际上栈的状态已经被修改过，可能导致数据不一致的问题。

```java
public class LockFreeStack<T> {
    private AtomicReference<Node<T>> head = new AtomicReference<>();
    public T pop() {
        Node<T> oldHead;
        Node<T> newHead;
        do {
            oldHead = head.get();
            if (oldHead == null) {
                return null;
            }
            newHead = oldHead.next;
        } while (!head.compareAndSet(oldHead, newHead));
        return oldHead.value;
    }
}
```

**【解决ABA问题的办法】**

1. **引入版本号**：在每次更新一个变量的时候，不仅更新变量的值，还更新一个版本号。`CAS` 操作在比较的时候，除了比较值是否一致，还比较版本号是否匹配。Java 当中的 `AtomicStampReference` 提供了版本号机制来避免 `ABA` 问题。每次更新`stampedRef` 的时候，都会一起更新对应的版本号

   ```java
   AtomicStampedReference<Integer> stampedRef = new AtomicStampedReference<>(1, 0);
   int[] stampHolder = new int[1];
   Integer ref = stampedRef.get(stampHolder);
   ```

2. **使用`AtomicMarkableReference`**：在变量的引用上标记一个布尔值，用来区分是否发生了特定的变化。不使用版本号，直接用标记位来追踪状态的变化。

   ```java
   AtomicMarkableReference<Integer> markableRef = new AtomicMarkableReference<>(1, false);
   ```

