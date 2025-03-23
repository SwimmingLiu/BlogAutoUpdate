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

## 1. 什么是 Java 内存模型（JMM）？

`JMM` 是 Java内存模型 ， 它是Java虚拟机 `JVM` 定义的一种规范，用于描述多线程程序中的变量，像实例字段、静态字段和数组元素，他们如何在内存中存储和传递的规则。也就是规定线程啥时候从住内存里面读数据，啥时候把数据写回到住内存。`JMM` 的核心目标是确保多线程环境下的**可见性**、**有序性**和**原子性**, 避免硬件和编译器优化带来的不一致问题。

`JMM`  当中包含**主内存** (所有线程共享) 和 **工作内存** （每个线程私有）两种内存。

- **主内存**：主内存是Java堆内存的一部分，是线程共享的区域，存储全局变量，比如静态变量、实例字段、数组元素等等。线程不能直接操作主内存，必须通过工作内存间接访问。
- **工作内存**：每个线程私有的本地内存，缓存主内存的变量副本。线程对变量的读写操作均在工作内存种进行，修改后的结果需要同步会主内存。线程是通过内存屏障 ( `volatile` 关键字) 或者 同步操作 ( `synchronized` ) 实现主内存和工作内存的数据一致性的。

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
- **轻量级锁**：当有另外的线程尝试去获取已经被偏向的锁时，锁会升级为轻量级锁。此时，申请上锁的过程中，JVM会在当前线程的栈帧（包括所有的局部变量、参数和返回地址）中创建一个锁记录`Lock Record`，让锁记录指向锁对象。然后用 `CAS` 替换锁对象的`Mark Word`， 并将`Mark Word` 的值存入锁记录。如果替换成功，锁对象的`Mark Word` 就变成当前线程的所锁记录。使用 `CAS` 操作的目的是减少锁竞争的开销。
- **重量级锁**：当 `CAS` 失败无法获取锁的时候，JVM判定其为多个线程竞争锁激烈。锁会升级成为重量锁，会使用操作系统的互斥量 `Mutex` 来实现线程的阻塞和环形。若果获取锁成功，线程会被放入Monitor的 `owner` 当中

![Monitor结构](https://oss.swimmingliu.cn/4a41e56d-f9c8-11ef-8eab-c858c0c1deba)

**【锁消除和锁粗化】**

- **锁消除**：JVM判断对象是否只在当前线程使用，如果只在当前线程使用，则消除不必要加的锁
- **锁粗化**：多个锁频繁操作出现时，JVM会将这些锁操作合并，见锁获取和释放的开销。

**【参考材料】** [黑马程序员-synchronized锁升级](https://blog.csdn.net/qq_38246328/article/details/124654374)

**【Synchronized 性能优化】**

`synchronized` 在 `JDK 1.6` 之后进行了很多性能优化，主要包括下面的四种：

- **偏向锁**：如果一个锁被同一个线程多次获得， `JVM` 会把这个锁设置为偏向锁，减少获取锁的代价
- **轻量级锁**：如果没有线程竞争，`JVM` 会将锁设置为轻量级锁，使用 `CAS` 操作代替互斥同步
- **锁粗化**：`JVM` 在 `JIT` 编译的时候，会检测到一些没有竞争的锁，并将这些锁去掉，减少同步的开销
- **锁消除**：`JVM` 会将一些短时间内连续的锁操作合并为一个锁操作，减少锁操作的开销。

## [补充] 3.  Java 中的 synchronized 轻量级锁是否会进行自旋？

`synchronized` 轻量级锁的阶段，只会通过 `CAS` 操作来替换对象头当中的 `Mark Word` 来获取锁，不会有自旋操作。如果 `CAS` 失败了之后，会直接膨胀为 重量级锁。如果获取重量级锁失败，会进行自旋操作。

**【synchronized 锁的四个阶段】**

- **无锁阶段**：最开始，多个线程可能都不需要同步访问共享资源，因此可能不存在显式的锁。不存在任何锁的开销，这个阶段性能最高
- **偏向锁阶段**：当某一个线程多次访问共享资源的时候，`JVM` 会认为这个线程在将来还会继续访问这个资源，所以使用偏向锁。偏向锁的上锁方式是，修改锁定对象的对象头，标记当前线程的 `线程id`。这样其他线程就没法儿获取这个锁，只有这个线程可以无竞争的访问资源。
- **轻量级锁阶段**：当某一个线程尝试获取一个已经被偏向锁锁定的对象的时候，就会进入轻量级锁阶段。此时， `JVM` 会使用 `CAS` 操作修改对象头的`Mark Word` 为线程的`Lock Record`，来尝试获取锁。如果 `CAS` 操作成功，那么这个线程就获取了锁。 如果 `CAS` 失败，那么就会进入重量级锁阶段
- **重量级阶段**：当轻量级锁阶段的 `CAS` 操作失败的时候，会升级为重量级锁。重量级锁意味着线程会进入阻塞状态，等待锁的释放。其他线程在获取重量级锁的时候，需要阻塞等待或进行上下文切换，可能会导致性能下降。如果重量级锁获取失败，会进行自选的操作。

## 4. AQS了解过吗?

`AQS` (`Abstract Queued Synchronizer`简称，抽象的队列式同步器), 它其实就是把跟锁相关的操作进行抽象和封装的一个工具类。将排队、入队、加锁、中断这些方法提供出来，方便其他相关`JUC` 锁的使用，具体的加锁时机、入队时机都需要实现类自己控制。
`AQS` 通过维护一个共享状态 `state` 、一个先进先出 `FIFO` 的等待队列和一个条件队列，来管理线程对共享资源的访问。共享状态 `state` 用 `volatile` 修饰，表示当前资源的状态，保证了可见性和有序性。在独占锁中 (`ReentrantLock` 或者 `synchronized`), `state` 为 `0` 表示未被占用，为`1` 表示已被占用。假如当前共享资源已经被占用，线程获取资源失败，会被加入到`AQS` 的等待队列里面。这个队列是一个变体的 `CLH` 队列， 采用双向链表结构，其节点包含线程的引用、等待状态、前驱节点和后继节点的指针。`AQS` 的常见实现类有 `ReentrantLock`、`CountDownLatch` 、`Semaphore` 等等
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

**【`ReentrantLock` 使用方法】**

**`ReentrantLock` 可重入锁**：需要通过手动调用 `lock()` 和 `unlock()` 方法来获取和释放锁。支持更多高级功能，比如**可中断的锁等待**、**定时锁等待**、公平锁等待等

```java
public class ReentrantLockDemo {
    // 构造时传入true表示使用公平锁
    private final ReentrantLock lock = new ReentrantLock(true);
    // 可中断锁等待示例
    public void interruptibleLockMethod() throws InterruptedException {
        lock.lockInterruptibly();  // 可中断等待
        try {
            // 临界区代码
            System.out.println("获取锁成功（可中断等待）");
        } finally {
            lock.unlock();
        }
    }
    // 定时锁等待示例
    public void timedLockMethod() throws InterruptedException {
        if (lock.tryLock(5, TimeUnit.SECONDS)) { // 等待5秒
            try {
                // 临界区代码
                System.out.println("在规定时间内获取到锁");
            } finally {
                lock.unlock();
            }
        } else {
            System.out.println("未在规定时间内获取到锁");
        }
    }
    public static void main(String[] args) {
        ReentrantLockDemo demo = new ReentrantLockDemo();
        Thread t1 = new Thread(() -> {
            try {
                demo.interruptibleLockMethod();
            } catch (InterruptedException e) {
                System.out.println("线程1等待时被中断");
            }
        });
        Thread t2 = new Thread(() -> {
            try {
                demo.timedLockMethod();
            } catch (InterruptedException e) {
                System.out.println("线程2等待时被中断");
            }
        });
        t1.start();
        t2.start();
        // 可根据需要中断线程
        t1.interrupt();
    }
}
```

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

## 8. 你了解 Java 线程池的原理吗？

线程池是用来管理线程的资源池，当需要用线程的时候，直接从线程池里面取就可以了。使用完线程之后，不会立即销毁，而是等待下一个任务。

**【为什么要使用线程池】**

- **减少资源的消耗**：通过重复利用已经创建的线程，避免重复创建线程和销毁线程造成的消耗
- **提高响应速度**
- **避免线程抢占过多资源**：如果线程频繁创建系统资源，会降低系统的稳定性。而且使用线程池进行统一分配、调优和监控。

**【如何创建线程池】**

1. 通过 `ThreadPoolExecutor` 构造函数创建

   ```java
   // ThreadPoolExecutor 构造函数
   public ThreadPoolExecutor(int corePoolSize,
                             int maximumPoolSize,
                             long keepAliveTime,
                             TimeUnit unit,
                             BlockingQueue<Runnable> workQueue) {
       this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
            Executors.defaultThreadFactory(), defaultHandler);
   }
   // 用ThreadPoolExecutor创建看对象
   ThreadPoolExecutor executor = new ThreadPoolExecutor(
       2, // corePoolSize 核心线程数，即使空闲也不会被回收
       5, // maximumPoolSize 最大线程数，当任务队列满时会创建新线程，但不能超过此值。
       60L, TimeUnit.SECONDS, // keepAliveTime 非核心线程的空闲存活时间(单位由unit参数指定)
       new LinkedBlockingQueue<>(10), // 任务队列 (如LinkedBlockingQueue/ArrayBlockingQueue)
       Executors.defaultThreadFactory(), // 线程工厂（可设置线程名称、优先级等）
       new ThreadPoolExecutor.CallerRunsPolicy() // 拒绝策略(如 AbortPolicy 抛出异常，CallerRunsPolicy 由提交线程执行任务)
   );
   ```

2. 通过 `Executor` 框架的工具类 `Executors` 来创建

   `Executor` 当中包含五种不同的线程池

   | 线程池类型                                   | 特性                                                         |
   | -------------------------------------------- | ------------------------------------------------------------ |
   | 固定大小线程池 `FixedThreadPool`             | 固定线程数量的线程池。核心线程数量和最大线程数量是一致的。有新任务提交并且有空闲线程就立即执行，否则存放到请求队列当中 |
   | 单线程线程池 `SingleThreadExecutor`          | 只有一个线程的线程池。如果线程执行发生异常，线程池会重新创建一个线程来执行后续的任务。适合用于保证任务按照提交顺序执行 |
   | 可缓存线程池 `CachedThreadPool`              | 根据实际情况调整线程数量的线程池，优先使用可复用的线程。如果当前线程不够用，会创建新的线程，最多创建的个数是`int` 最大值 |
   | 定时任务线程池 `ScheduledThreadPool`         | 在一定时间后，或者定时去执行任务的线程池                     |
   | 单线程定时任务线程池 `SingleThreadScheduled` | 和定时任务线程池基本一样，相当于定时任务线程池里面只有一个线程 |

   【注意】 一般不推荐使用 `Executors` 去创建线程，而是使用`ThreadPoolExecutor` 的方式创建，避免资源耗尽的风险。

   >【Executors 的弊端】
   >
   >1. 固定大小线程池 `FiexedThreadPool` 和 `SingleThreadPool`：允许的请求队列长度为 `Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致 `OOM` 内存不足
   >2. 可缓存线程池 `CachedThreadPool` 和 定时任务线程池 `ScheduledThreadPool`： 允许创建线程的数量为 `Integer.MAX_VALUE`， 可能会创建大量线程，从而导致 `OOM` 内存不足
   >3. 定时任务线程池 `ScheduledThreadPool` 和 单线程定时任务线程池 `SingleScheduledThreadPool` :  使用的无界的延迟阻塞队列 `DelayedWorkQueue` 用来存放后续的线程，任务队列的最大长度为 `Integer.MAX_VALUE`， 可能会堆积大量的请求，从而导致 `OOM` 内存不足

   ```java
   // 固定大小线程池 newFixedThreadPool
   ExecutorService executor = Executors.newFixedThreadPool(5); // 核心线程数=最大线程数=5
   // 单线程线程池 newSingleThreadExecutor
   ExecutorService executor = Executors.newSingleThreadExecutor();
   // 可缓存线程池 newCachedThreadPool
   ExecutorService executor = Executors.newCachedThreadPool();
   // 定时任务线程池 newScheduledThreadPool
   ScheduledExecutorService executor = Executors.newScheduledThreadPool(3);
   // 延迟执行
   executor.schedule(() -> System.out.println("Delayed Task"), 3, TimeUnit.SECONDS);
   // 周期性执行
   executor.scheduleAtFixedRate(() -> System.out.println("Periodic Task"), 1, 5, TimeUnit.SECONDS);
   ```

**【线程池的主要参数】**

线程池的主要参数有 `7` 个，分别是核心线程池大小 `corePoolSize`、 最大线程池大小 `maximumPoolSize`、线程存活时间和它的单位、存放执行任务的阻塞队列 `workQueue`、线程工厂 `threadFactory` 、拒绝策略 `handler`

| 参数名称                             | 类型                       | 特性                                                         |
| ------------------------------------ | -------------------------- | ------------------------------------------------------------ |
| **核心线程池大小 `corePoolSize`**    | `int`                      | 线程池核心线程数量。默认情况下，线程池中的线程数量 `<=` 核心线程数量 `coolPoolSize`，即使线程是空闲状态，都不会删除的线程。 |
| **最大线程池大小 `maximumPoolSize`** | `int`                      | 线程池最多可以容纳的线程数量。如果阻塞队列已经满了，但是当前线程数 `<` 最大线程池大小 `maximumPoolSize`，则会继续创建线程。如果大于等于，就执行拒绝策略。 |
| **线程存活时间 `keepAliveTime`**     | `long`                     | 线程池中大于核心线程 `corePoolSize` 的其他 `非核心线程` 处于空闲状态下的存活时间。超过该时间就会被回收销毁 |
| **存活时间单位 `unit`**              | `TimeUnit`                 | 线程存活时间的单位                                           |
| **阻塞队列 `workQueue`**             | `BlockingQueue<Runnable>`  | 阻塞工作队列，没有空闲的线程可以执行的时候，新的任务会被放在阻塞队列里面 |
| **线程工厂 `threadFactory`**         | `ThreadFactory`            | `Executor` 创建新线程的时候会用到，可以给线程取名字          |
| **拒绝策略 `handler`**               | `RejectedExecutionHandler` | 拒绝策略                                                     |

![线程池的工作流程图](https://oss.swimmingliu.cn/7291ad61-07e8-11f0-a47f-c858c0c1debd)

```java
public ThreadPoolExecutor(int corePoolSize,  // 核心线程池大小
                          int maximumPoolSize,  // 最大线程池大小
                          long keepAliveTime,  // 线程存活时间
                          TimeUnit unit,  // 时间单位（如秒、毫秒等）
                          BlockingQueue<Runnable> workQueue,//用于存放待执行任务的阻塞队列
                          ThreadFactory threadFactory,  // 线程工厂，用于创建新线程
                          RejectedExecutionHandler handler) {  // 拒绝策略处理器
    if (corePoolSize < 0 ||  // 核心线程池大小必须大于或等于0
        maximumPoolSize <= 0 ||  // 最大线程池大小必须大于0
        maximumPoolSize < corePoolSize ||  // 最大线程池大小不能小于核心线程池大小
        keepAliveTime < 0)  // 线程存活时间不能小于0
        throw new IllegalArgumentException();  // 参数不合法，抛出异常
    if (workQueue == null || threadFactory == null || handler == null)  // 参数不能为空
        throw new NullPointerException();  // 参数为空，抛出空指针异常
    this.corePoolSize = corePoolSize;  // 设置核心线程池大小
    this.maximumPoolSize = maximumPoolSize;  // 设置最大线程池大小
    this.workQueue = workQueue;  // 设置任务队列
    this.keepAliveTime = unit.toNanos(keepAliveTime);  // 将线程存活时间转换为纳秒
    this.threadFactory = threadFactory;  // 设置线程工厂
    this.handler = handler;  // 设置拒绝策略处理器
}
```

**【线程池的工作原理】**

线程池的工作原理， 可以理解为去银行办业务。具体的工作流程如下：

1. 当前线程提交任务 （准备去存钱）
2. 判断线程池是否还在运行，如果没有直接调用拒绝策略 （银行柜台今天都没人， 快滚）
3. 如果线程池还在运行，则判断已有线程数是否小于核心线程数，如果小于核心线程数，添加工作线程直接执行 （有空闲的柜台小姐姐，可以帮忙办理业务）
4. 如果已有线程数大于等于核心线程数，则判断阻塞队列是否满了，如果没满，将其放入阻塞队列当中。（柜姐忙不过来了，先去排队等着）
5. 如果阻塞队列已经满了，则判断当前线程数是否超过了最大线程数。如果没有，则创建新的线程。（排队的人太多了，已经排不下了。多开一个柜台，分配一个柜姐去办理业务）
6. 如果超过了最大线程数，直接根据拒绝策略进行处理。（已经没有多的柜姐了，只能让他滚蛋了）

![线程池的工作原理](https://oss.swimmingliu.cn/72c9f193-07e8-11f0-abd9-c858c0c1debd)

**【核心线程支持被回收吗？】**

如果线程池用于周期性使用、频率不高（周期间有明显空闲时间）的场景，可以将 `allowCoreThreadTimeOut (boolean value)` 方法参数设为 `true`， 回收空闲的核心线程 (时间间隔还是由 `keepAliveTime` 指定)

```java
public void allowCoreThreadTimeOut(boolean value) {
    if (value && keepAliveTime <= 0) // 检查存活是否合法
        throw new IllegalArgumentException("Core threads must have nonzero keep alive times");
    if (value != allowCoreThreadTimeOut) {
        allowCoreThreadTimeOut = value;
        if (value)
            interruptIdleWorkers();
    }
}
```

**【线程池满了，有哪些拒绝策略?】**

| 策略名称                                    | 特性                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| **终止策略 `AbortPolicy` (默认)**           | 抛出异常 `RejectExecutionException` 来拒绝新任务的处理       |
| **调用者运行策略 `CallerRunsPolicy`**       | 让调用线程池的`execute`方法的线程，自己执行任务。如果调用的线程已经被关闭，则丢弃该任务。这种策略会降低新任务的提交速度，影响整体的性能。如果在承受延迟的范围内，所有任务都可以正常执行，可以选择该策略 |
| **丢弃策略`DiscardPolicy`**                 | 不处理新任务，直接丢弃                                       |
| **最早未处理丢弃策略`DiscardOldestPolicy`** | 丢弃最早没有被处理的任务请求                                 |

**【调用者运行策略 `CallerRunsPolicy` 有什么风险？如何解决？】**

如果采用 `CallerRunsPolicy` 策略的任务执行时间比较长，并且处理提交任务的是主进程。可能会阻塞主进程处理其他的任务，影响程序正常运行。

**【解决方案】** 

1. **尽量避免调用拒绝策略 **（利用服务器资源）

   - 调大 `maximumPoolSize` 最大线程参数，多添加一些线程，充分利用 `CPU`。提高任务的处理速度，避免在 阻塞队列`BlockingQueue` 的任务过多导致内存用完
   - 增加阻塞队列 `BlockingQueue` 的大小并调整堆内存（新增的数据，放在堆内存），容纳更多的任务

   ![解决CallerRunsPolicy风险的方案-避免调用拒绝策略](https://oss.swimmingliu.cn/72f5edfa-07e8-11f0-b1f7-c858c0c1debd)

2. **如果服务器资源有限，需要保证任务不被丢弃**

   - 实现 `RejectExecutionHandler` 接口自定义拒绝策略，当线程池的阻塞队列满了知乎，将没法儿处理的任务存储到 `MySQL` 数据库里面
   - 实现一个继承阻塞队列 `BlockingQueue` 的混合队列，该队列结合 `ArrayBlockingQueue` 和数据库。重写 `take()` 方法，会优先从数据库读取最高的任务，如果没有，则从 `ArrayBlockingQueue` 中取任务

   ![解决CallerRunsPolicy风险的方案-保证任务不丢](https://oss.swimmingliu.cn/731416f8-07e8-11f0-b20a-c858c0c1debd)

**【线程池常用的阻塞队列有哪些？】**

| 阻塞队列名称                           | 特性                                                         | 绑定的线程池类                                     |
| -------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| **有界阻塞队列 `LinkedBlockingQueue`** | 底层由链表实现，容量为 `int` 的最大值                        | `FixedThreadPool`、`SinglThreadExecutor`           |
| **同步队列 `SynchronousQueue` **       | 保证对于提交的任务，如果有空闲线程，就用空闲线程处理。否则新建线程来处理任务 | `CachedThreadPool`                                 |
| **延迟队列 `DelayedWorkQueue`**        | 任务按照延迟时间长短进行排序，不按照原来的放入顺序进行排序。内部采用 “堆” 的数据结构，确保每次出队任务是当前队列汇总执行时间最靠前的 | `ScheduledThreadPool`、`SingleScheduledThreadPool` |
| **有界阻塞队列 `ArrayBlockingQueue`**  | 底层由数组实现，容量一旦创建就不能修改                       |                                                    |

**【线程池的参数一般如何设置？】**

线程池的参数可以根据任务的类型来区分，可以分为两种不同的情况，分别为 `CPU` 密集型和 `I/O` 密集型

- **`CPU` 密集型**：`corePoolSize` = `CPU` 核心数目 + 1

  这一类的任务消耗的主要是 `CPU` 资源，比 `CPU` 核心数多的一个线程，用来防止进程因为中断而被暂停（比如说缺页中断）

- **`I/O` 密集型**：`corePoolSize` = `CPU` 核心数目 * 2

  这一类的任务，系统大部分时间在处理 `I/O` 交互， 线程处理 `I/O` 的时候，不占 `CPU` 资源。所以，可以把`CPU` 让给其他线程。最极端的情况就是有 `CPU` 核心数目这么多的线程在处理 `I/O`， 刚好 `CPU` 全部空出来了。

**【注意】** 如果线程数目设置的过多，会增加上下文切换的成本。上下文切换就是当线程的时间片用完了重新处于就绪状态，把`CPU`让给其他线程，这个过程就是一次上下文切换。

**【线程池的 `shutdown()` 和 `showdownNow()` 的作用和区别】**

- **`shutdown()`**：设置状态为`shutdown`，让正在执行的任务继续执行，还没有执行的任务中断。此时不能往线程池中添加任务，否则抛出`RejectedExecutionException` 异常。

  ```java
  public void shutdown() {
      // 获取线程池的主锁（ReentrantLock），用于同步关闭操作
      final ReentrantLock mainLock = this.mainLock;
      mainLock.lock(); // 加锁，防止并发修改线程池状态
      try {
          // 1. 检查是否有权限关闭线程池（例如安全管理器限制）
          checkShutdownAccess();
          // 2. 将线程池状态推进到 SHUTDOWN，阻止新任务提交（但允许处理队列中的任务）
          advanceRunState(SHUTDOWN);
          // 3. 中断所有空闲的工作线程（正在等待任务的线程）
          interruptIdleWorkers();
          // 4. 调用钩子方法，供子类扩展（如 ScheduledThreadPoolExecutor 取消延迟任务）
          onShutdown(); // hook for ScheduledThreadPoolExecutor
      } finally {
          mainLock.unlock(); // 释放锁
      }
      // 5. 尝试终止线程池（若队列为空且没有活动线程）
      tryTerminate();
  }
  ```

- **`shutdownNow()`**：返回哪些还没有被执行的任务，然后尝试终止线程。尝试终止线程是调用 `Thread.interrupt()` 方法来实现的。`shutdownNow()` 不代表线程池一定立即退出，可能要等所有正在执行的任务完成之后才退出。

  ```java
  public List<Runnable> shutdownNow() {
      List<Runnable> tasks;
      final ReentrantLock mainLock = this.mainLock;
      mainLock.lock();
      try {
          checkShutdownAccess();
          advanceRunState(STOP);
          interruptWorkers();
          tasks = drainQueue();
      } finally {
          mainLock.unlock();
      }
      tryTerminate();
      return tasks;
  }
  ```

**【线程池当中的线程执行异常之后，销毁还是复用？】**

此处需要分成两种提交任务的方式来区分，分别是 `execute()` 和 `submit()` 方式

- **使用 `execute()` 提交任务，新线程替换老线程**：当任务通过 `execute()` 提交到线程池并且在执行过程中，抛出异常的时候。如果异常没有在任务当中进行`try-catch`，会导致当前线程终止。异常会打印在控制台或者日志文件里面。线程池检测到线程终止之后，会创建新线程替换原来的线程，维持配置的线程数不变。
- **使用 `submit()` 提交任务，线程不终止**：如果在任务执行的过程中发生异常，不会终止当前的线程，也不会直接打印异常。异常会被封装在 `submit()` 返回的`Future` 对象里面。调用 `Future.get()` 可以看到 `ExecutionException`。此时线程不会因为异常而终止，还在线程池里面，准备执行后续任务。

**【如何给线程命名？】**

1. 借助 `guava` 中的 `ThreadFactoryBuilder`

   ```java
   public class ThreadFactoryBuilderDemo {
       static final String threadNamePrefix = "test-thread-name-prefix:";
       public static void main(String[] args) {
           ThreadFactory threadFactory = new ThreadFactoryBuilder()
                   .setNameFormat(threadNamePrefix + "---- SwimmingLiu")
                   .setDaemon(true)
                   .build();
           ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(
                   1,
                   2,
                   60, TimeUnit.SECONDS,
                   new ArrayBlockingQueue<>(1),
                   threadFactory);
           Runnable target = () -> {
               String name = Thread.currentThread().getName();
               System.out.println(name);
           };
           poolExecutor.execute(target);
           poolExecutor.execute(target);
           poolExecutor.shutdown();
           /*
            输出:
               test-thread-name-prefix:---- SwimmingLiu
               test-thread-name-prefix:---- SwimmingLiu
           */
       }
   }
   ```

2. 自定义实现 `ThreadFactory` 接口

   ```java
   // 线程工厂，它设置线程名称，有利于我们定位问题。
   public final class NamingThreadFactory implements ThreadFactory {
       private final AtomicInteger threadNum = new AtomicInteger();
       private final String name;
       // 创建一个带名字的线程池生产工厂
       public NamingThreadFactory(String name) {
           this.name = name;
       }
       @Override
       public Thread newThread(Runnable r) {
           Thread t = new Thread(r);
           t.setName(name + " [#" + threadNum.incrementAndGet() + "]");
           return t;
       }
   }
   ```

**【如何用两个线程按照顺序打印奇数和偶数】**

```java
public class PrintOddorEven {
    // 两个线程按照顺序打印奇数和偶数
    private static final Object lock = new Object(); // synchronized锁对象
    private static int count = 1;
    private static final int MAX_COUNT = 10;

    public static void main(String[] args) {
        Runnable printOdd = () -> {
            synchronized (lock) {
                while (count <= MAX_COUNT) {
                    if (count % 2 != 0) {
                        System.out.println(Thread.currentThread().getName() + ":" + count ++);
                        lock.notify();
                    } else {
                        try {
                            lock.wait();
                        } catch (Exception error) {
                            error.printStackTrace();
                        }
                    }
                }
            }
        };
        Runnable printEven = () -> {
            synchronized (lock) {
                while (count <= MAX_COUNT) {
                    if (count % 2 == 0) {
                        System.out.println(Thread.currentThread().getName() + ":" + count ++);
                        lock.notify();
                    } else {
                        try {
                            lock.wait();
                        } catch (Exception error) {
                            error.printStackTrace();
                        }
                    }
                }
            }
        };
        Thread printOddThread = new Thread(printOdd, "printOdd");
        Thread printEvenThread = new Thread(printEven, "printEven");
        printOddThread.start();
        printEvenThread.start();
    }
}
```

## 9. 你使用过哪些 Java 并发工具类？

面试官，您好。我使用过 `ConcurrentHashMap`、`BlockingQueue`、`CountdownLatch`、`AtomicInteger`、`Semaphore` 、`CyclicBarrier` 这些工具类。

**【 `CountDownLatch` 】**

`CountDownLatch` 是 `JUC` 包中的一个同步工具类，用来协调多个线程之间的同步。允许一个或者多个线程等待，直到其他线程中执行的一组操作完成。一般适用于主线成需要等待多个子线程完成任务的场景。`CountDownLatch` 是通过一个计数器实现的，该计数器最开始记录需要等待的线程的总个数，由线程来进行递减，直至为 `0`

```java
// 王者荣耀，等待五位英雄加载完成之后，才能开始游戏
// countDownLatch.countDown(); 就相当于 notify()
// countDownLatch.await(); 就相当于 wait()
// new CountDownLatch(int count), 创建一个带有给定计数器的 CountDownLatch
public static void main(String[] args) throws InterruptedException {
    CountDownLatch countDownLatch = new CountDownLatch(5);
    Thread daqiao = new Thread(() -> {
        System.out.println("大乔已就位！");
        countDownLatch.countDown(); // 释放信号
    });
    Thread lanlingwang = new Thread(() -> {
        System.out.println("兰陵王已就位！");
        countDownLatch.countDown();
    });
    Thread anqila = new Thread(() -> {
        System.out.println("安其拉已就位！");
        countDownLatch.countDown();
    });
    Thread nezha = new Thread(() -> {
        System.out.println("哪吒已就位！");
        countDownLatch.countDown();
    });
    Thread kai = new Thread(() -> {
        System.out.println("铠已就位！");
        countDownLatch.countDown();
    });
    daqiao.start();
    lanlingwang.start();
    anqila.start();
    nezha.start();
    kai.start();
    countDownLatch.await(); //等待信号
    System.out.println("全员就位，开始游戏！");
}
```

**【场景题】** 假如要查10万多条数据，用线程池分成20个线程去执行，怎么做到等所有的线程都查找完之后，即最后一条结果查找结束了，才输出结果。

```java
public class CountDownLatchTest {
    private static int count = 100000;
    private static final int THREAD_TOTAL_NUMS = 20;
    private static final int BATCH_SIZE = count / THREAD_TOTAL_NUMS;
    @Test
    public void test() throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(THREAD_TOTAL_NUMS);
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_TOTAL_NUMS);
        ConcurrentLinkedQueue<String> results = new ConcurrentLinkedQueue<>(); // 存放查询结果
        for (int i = 0; i < THREAD_TOTAL_NUMS; i++) {
            int start = i * BATCH_SIZE;
            int end = (i == THREAD_TOTAL_NUMS - 1) ? count : (start + BATCH_SIZE);
            executor.submit(() -> {
                try {
                    for (int k = start; k < end; k++) {
                        results.add("Data - " + k);
                    }
                    System.out.println(Thread.currentThread().getName() + "已处理数据：[" + start + ", " + (end - 1) + "]");
                } finally {
                    latch.countDown();
                }
            });
        }
        latch.await();
        executor.shutdown();
        System.out.println("查询完毕，查询结果的总数：" + results.size());
    }
}
```

**【 `CyclicBarrier` 】**

`CyclicBarrier` 就是可循环使用的屏障，主要用于让一组线程到达一个屏障（或者叫同步点）的时候，阻塞这部分线程。直到最后一个线程到达屏障，屏障才会开门，所有被屏障拦截的线程才会继续运行。 `CyclicBarrier` 和 `CountDownLatch` 很相似，都可以协调多线程的结束动作。

![CyclicBarrier的流程图](https://oss.swimmingliu.cn/73347f5e-07e8-11f0-8203-c858c0c1debd)

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;
public class CyclicBarrierDemo {
    final static int threadCount = 3;
    static CyclicBarrier barrier = new CyclicBarrier(threadCount, () -> System.out.println("所有线程都抵达该屏障，执行后续...."));
    public static void main(String[] args) {
        for (int i = 0; i < threadCount; i++) {
            new T(barrier).start();
        }
        /*输出:
            Thread-2 正在执行任务...
            Thread-1 正在执行任务...
            Thread-0 正在执行任务...
            Thread-0 到达屏障
            Thread-1 到达屏障
            Thread-2 到达屏障
            所有线程都抵达该屏障，执行后续....
        */
    }
}

class T extends Thread {
    private CyclicBarrier barrier;
    public T(CyclicBarrier barrier) {
        this.barrier = barrier;
    }
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 正在执行任务...");
        try {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " 到达屏障");
            this.barrier.await(); // 等待其他线程
        } catch (InterruptedException | BrokenBarrierException e) {
            throw new RuntimeException(e);
        }
    }
}
```

**【 `CountDownLatch` 和  `CyclicBarrier` 区别】**

- **使用次数**：`CountDownLatch` 是一次性的（达到条件就结束了），`CyclicBarrier` 是可以重复利用的，可以设置多个屏障 （多个结束条件）
- **是否能够等待其他线程**：`CountDownLatch` 中的各个子线程都不会等待其他线程，只能完成自己的任务。 `CyclicBarrier` 的各个线程可以等待其他线程

| 特性         | CountDownLatch                                             | CyclicBarrier                                                |
| ------------ | ---------------------------------------------------------- | ------------------------------------------------------------ |
| **使用次数** | 一次性的，不同的线程在同一个计数器上工作，直到计数器为 `0` | 可重复利用，每个线程都会等待其他线程完成任务，然后拆除对应的屏障 |
| **面向对象** | 任务数                                                     | 线程数 （所有线程都完成任务，拆除屏障）                      |
| **定义方式** | 指定任务数，这部分任务由哪些线程完成无所谓                 | 指定需要参与协作的线程数，这些线程必须调用`await()`          |
| **结束条件** | 计数器为 `0` ，就直接结束了                                | 所有线程完成任务之后，可以重复利用                           |
| **出现异常** | 某个线程出现问题，其他线程不受影响                         | 某个线程出现问题（中断、超时），则处于`await()` 的线程会出问题 |

**【`Semaphore` 信号量】**

`Semaphore` 信号量是用来控制同时访问t特定资源的线程量，通过协调各个线程，保证合理的使用公共资源。`Semaphore` 的常规用途是用作流量控制，比如说连接数据库的线程数量, 数据库连接数量是需要控制的。
假如有一个需求，需要读取 `10w` 个文件，然后存入数据库当中。因为大部分都是 `I/O` 操作，属于 `I/O` 密集型，可以采用多线程进行操作。但是，数据库的连接数量是有限的（假如是 `10` 个），所以必须要控制只有 `10` 个线程来连接数据库。这个时候，就需要用`Semaphore` 信号量来控制流量。

```java
// 下面设置的线程是30个，但是只允许10个线程并发执行。使用Semaphore当中的acquire和release来进行控制。
public class SemaphoreTest {
    private static final int THREAD_COUNT = 30;
    private static ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);
    private static Semaphore s = new Semaphore(10);
    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(new Runnable() {
                @Override
                public void run() {
                    try {
                        s.acquire();
                        System.out.println("save data");
                        s.release();
                    } catch (InterruptedException e) {
                    }
                }
            });
        }
        threadPool.shutdown();
    }
}
```

## 10. Synchronized 和 ReentrantLock 有什么区别？

| 特性           | Synchronized                                                 | ReentrantLock                                                |
| -------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| **实现机制**   | Java内置的关键字，依赖 `JVM` 实现基本的同步机制, 基于监管器 `Monitor` （重量级） 和 对象头（偏向锁和轻量级锁）实现 | `JUC` 类库提供的，基于 `AQS` （ `AbstactQueuedSynchronizer` ）抽象队列同步器实现 |
| **加锁/解锁**  | `JVM` 自动对代码块/方法进行加锁和释放锁                      | 显示锁机制，需要通过 `lock()` 和 `unlock()` 手动管理锁的获取和释放 |
| **是否可重入** | 可重入                                                       | 可重入                                                       |
| **灵活性**     | 不太灵活，只支持非公平锁，不支持设置超时时间、不支持中断，以及多条件判断。 | 比较灵活，支持公平锁和非公平锁，支持设置超时时间（避免死锁），可以中断，支持多条件判断 |

**【Java 当中常用的锁有哪些？】**

- **`synchronized` 同步锁**：修饰方法或者代码块。线程进入`synchronized` 修饰的方法或者代码块的时候，会获取关联对象的锁 (一般会有一个 `Object` 类型的锁对象)。当线程退出该代码块或者方法的时候，锁会被释放。其他线程尝试获取同一个对象的锁，会被阻塞知道锁被释放。

- **`ReentrantLock` 可重入锁**：需要通过手动调用 `lock()` 和 `unlock()` 方法来获取和释放锁。支持更多高级功能，比如可中断的锁等待、定时锁等待、公平锁等待等

- **`ReadWriteLock` 读写锁**：允许多个读取线程同时访问共享资源，但是只允许一个写入者。读写锁一般用于读多写少的场景，可以提高并发性

- **乐观锁和悲观锁**

  - **乐观锁**：通常不会锁定资源，只有在更新数据的时候，检查数据是否被其他线程修改。一般通常使用版本号和时间戳来实现 （ `CAS` 就是乐观锁）

  - **悲观锁**：在访问数据前就锁定资源，就假设会出现最坏的情况，也就是数据可能被其他线程修改。 `synchronized` 和 `ReentrantLock` 都是悲观锁

## 11. Java 中 volatile 关键字的作用是什么？

`volatile` 关键字可以保证变量的**可见性**和**有序性**，也就是保证线程对变量的修改是可见的，并且禁止指令重排序优化。当变量被声明为 `volatile`， 则指示 `JVM` 这个变量是共享且异变的，每次使用它的时候，需要从主存房中进行获取最新的变量值。

- **可见性**：当某个线程修改了 `volatile` 变量的值，新值会立即被刷新到主内存。其他线程在读取该变量的时候，可以获取最新的值。这样可以避免线程间由于缓存一致性问题导致读取到旧值的现象。（因为每个线程内部的栈帧会记录主内存中的读取的）
- **有序性（禁止指令重排序）**: `volatile` 通过内存屏障来禁止特定情况下的指令重排序 （写屏障和读屏障），从而保证程序的执行顺序符合预期。对 `volatile` 变量的写操作会再前面加插入一个 `StoreStore` 写屏障，对 `volatile` 变量的读操作，则会在变量的后面插入一个 `LoadLoad` 读屏障。确保在多线程环境下，某些代码块的执行顺序的可预测性。

![volatile关键字变量和线程关系示意图](https://oss.swimmingliu.cn/735ef9a9-07e8-11f0-a703-c858c0c1debd)

**【volatile 关键字如何实现禁止指令重排序？】**

在主存当中，主要通过内存屏障类禁止特定类型的指令重新排序。(写前读后，`volatile` 禁止重排)

1. **写-写屏障 (`Write-Write`)**：对 `volatile` 变量进行写操作之前，会插入一个写屏障。确保在该变量写操作之前的所有普通写操作都完成了。
2. **读-写屏障 (`Read-Write`)**：对 `volatile` 变量执行读操作之后，会插入一个读屏障。确保了对 `volatile` 变量的读操作后的普通读操作，不会提前到 `volatile` 读之前，确保读到最新的数据。
3. **写-读屏障 (`Write-Read`)** ：写-读屏障是最重要的屏障，这个屏障可以确保 `volatile` 写操作之前的内存操作 （包括写操作） 不会重排序到 `volatile` 读操作后，并且 `volatile` 读后的内存操作（包括读操作）不会重排序到`volatile` 写操作之前

【注意】 为了性能优化，`JMM` Java内存模型在不改变正确语义的情况下，允许编译器和处理器对指令序列进行重排序。

**【volatile 和 synchronized 区别】**

- **synchronized** ：`synchronized` 是排他性同步机制，确保多线程访问共享资源时的互斥性，同一时刻只允许一个线程访问。通过在代码块或者方法上，添加该关键字实现同步。可以确保原子性
- **volatile** ：`volatile` 是轻量级同步机制，保证变量可见性和有序性 (禁止指令重排序)。变量声明成 `volatile` 的时候，线程读该变量的时候，直接从主内存当中读取，不使用工作内存当中的缓存。写操作立即刷回主内存，不缓存在本地内存

**【`volatile` 应用场景】**

1. 用于指示发生一个重要的一次性事件，比如完成了初始化或者请求停机

   ```java
   volatile boolean shutdownRequested;
   public void shutdown() {
       shutdownRequested = true;
   }
   public void doWork() {
       while(!shutdownRequested) {
           // 执行任务
       }
   }
   ```

2. **`volatile bean` 模式**：`JavaBean` 中的所有数据成员都是 `volatile` 类型的，并且 `getter` 和 `setter` 方法，除了获取或者设置相应的属性之外，不能包含任何逻辑。对于引用的数据成员，引用的对象必须是有效且不可变的。

   ```java
   public class Person {
       private volatile String firstName;
       private volatile String lastName;
       private volatile int age;
       public String getFirstName() { return firstName; }
       public String getLastName() { return lastName; }
       public int getAge() { return age; }
       public void setFirstName(String firstName) { 
           this.firstName = firstName;
       }
       public void setLastName(String lastName) { 
           this.lastName = lastName;
       }
       public void setAge(int age) { 
           this.age = age;
       }
   }
   ```

3. **写锁策略**：通过内部锁和 `volatile` 变量减少公共代码路径的开销。计算器使用 `synchronized` 保证增量操作的原子性，并用 `volatile` 确保结果的可见性。如果更新不频繁，该方法性能更优。因为读操作只涉及 `volatile` 读取，比无竞争的锁获取开销还要小

   ```java
   @ThreadSafe  // 标识该类是线程安全的，需通过同步机制保证多线程访问的正确性
   public class CheesyCounter {
       // 使用 volatile 保证可见性，但复合操作（如 value++）仍需同步
       @GuardedBy("this")  // 该字段受对象内部锁（this）保护，写操作必须持有锁
       private volatile int value;
       // 读操作未加锁，依赖 volatile 的可见性保证
       public int getValue() { 
           return value;  // volatile 保证读取最新值，但多线程下需注意"先验条件"的线程安全问题
       }
       // 写操作通过 synchronized 保证原子性和可见性
       public synchronized int increment() {
           return value++;  // 同步锁确保复合操作（读-改-写）的原子性
       }
   }
   ```

   