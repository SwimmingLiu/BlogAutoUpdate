---
title: "Java集合面试题笔记"
date: 2025-02-20T21:20:50+08:00
lastmod: 2025-02-20T21:20:50+08:00
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

## 1. 说说 Java 中 HashMap 的原理？

**【HashMap定义】**

- 结构：数组 + 链表 + 红黑树 (`JDK 1.8` 之后)

- 默认值：初始容量为16 (数组长度)，负载因子为 0.75。当存储的元素为 16 * 0.75 = 12个时，会触发`Resize()` 扩容操作，容量 x 2 并重新分配位置。但是扩容是有一定开销的，频繁扩容会影响性能。另外，`TREEIFY_THRESHOLD` 转换为红黑树的默认链表长度阈值为 8, `UNTREEIFY_THRESHOLD` 从红黑树转换为链表的阈值为 6。 两个阈值采用不同值的原因是防止刚转换为红黑树，又变成链表，反复横跳，消耗资源。

- 数组下标位置计算方法：首先使用key的`hashCode()`方法计算下标位置，然后通过  `indexFor()` (`JDK 1.7` 以前) 计算下标值。 `JDK 1.7`后，为了提高计算效率采用 `(len - 1) & hash` 来确定下标值。

  【注】数组的长度`len` 是2的幂次方时，`(len - 1) & hash` 等价于 `hash % len`。 这也是为什么数组长度必须是2的幂次方。

![HashMap底层结构](https://oss.swimmingliu.cn/65f25922-ef8d-11ef-827a-c858c0c1deba)

【**HashMap线程不安全**】

为了保证HashMap的读写效率高，它的操作是非同步的，也就是说读写操作没有锁保护。所以多线程场景下是线程不安全的。

**【HashMap不同版本区别】**

- JDK 1.7: 数组 + 链表，链表部分采用头插法，多线程会导致出现环形链表。扩容会计算每个元素hash值，并分配到新的位置，开销大。
- JDK 1.8：数组 + 链表 + 红黑树，采用高低位置来分配位置，即判断`(e.hash & oldCap) == 0`， 减少了计算hash的次数

**【HashMap的PUT方法】**

HashMap在存储数据时，按照如下流程：

1. 判断数组table是否为空或长度为0，如果是第一次插入，需要对数组进行扩容 `Resize()`
2. 计算key的数组索引值 `index = hash(key) & (len - 1)` 得到索引`i` , `len` 表示数组长度
3. 判断table[i]是否为空？
   - 若为空，则直接插入 -> 第7步 
   - 若不为空 -> 第4步
4. 判断key是否存在
   - 若存在，直接覆盖 -> 第7步
   - 若不存在 -> 第5步
5. 判断table[i]是否为TreeNode
   - 如果是TreeNode ，在红黑树中插入/覆盖  (同第4步，判断key是否存在) -> 第7步
   - 如果不是 -> 遍历链表 -> 在链表中插入/覆盖  (同第4步，判断key是否存在) -> 第6步
6. 如果是链表插入节点，判断链表长度`listLen` 是否 `>= TREEIFY_THRESHOLD`, 默认为8
   - 如果 `listLen >= 8`，转换为红黑树 -> 第7步
   - 如果 `listLen < 8` -> 第7步
7. **元素个数**自增，判断是否大于阈值 `threshold`， 其中`threshold = len * factor` 默认为 16 * 0.75 = 12。若超过阈值，则对数组扩容 `Resize()`

![HashMap原理之put操作](https://oss.swimmingliu.cn/662bfe39-ef8d-11ef-be05-c858c0c1deba)

**【HashMap的GET方法】**

1. 计算key的数组索引值 `index = hash(key) & (len - 1)` 得到索引`i` , `len` 表示数组长度
2. 判断table[i]是否直接`key.equals(k)`命中
   - 命中 -> 返回结果
   - 未命中 -> 第3步
3. 判断第一个节点是否为TreeNode
   - 若是TreeNode，在红黑树中查找
   - 若不是，遍历链表查找
4. 返回查找结果

![HashMap原理之get操作](https://oss.swimmingliu.cn/66448b28-ef8d-11ef-a7ba-c858c0c1deba)

**【Resize扩容操作】**

`Resize() ` 源码如下

``` java
/**
 * Initializes or doubles table size.  If null, allocates in
 * accord with initial capacity target held in field threshold.
 * Otherwise, because we are using power-of-two expansion, the
 * elements from each bin must either stay at same index, or move
 * with a power of two offset in the new table.
 *
 * @return the table
 */
final HashMap.Node<K,V>[] resize() {
    HashMap.Node<K,V>[] oldTab = table;
    // 记录Map当前的容量
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    // 记录Map允许存储的元素数量，即阈值（容量*负载因子）
    int oldThr = threshold;
    // 声明两个变量，用来记录新的容量和阈值
    int newCap, newThr = 0;

    // 若当前容量不为0，表示存储数据的数组已经被初始化过
    if (oldCap > 0) {
        // 判断当前容量是否超过了允许的最大容量
        if (oldCap >= MAXIMUM_CAPACITY) {
            // 若超过最大容量，表示无法再进行扩容
            // 则更新当前的阈值为int的最大值，并返回旧数组
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 将旧容量*2得到新容量，若新容量未超过最大值，并且旧容量大于默认初始容量（16），
        // 才则将旧阈值*2得到新阈值
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }

    // 若不满足上面的oldCap > 0，表示数组还未初始化，
    // 若当前阈值不为0，就将数组的新容量记录为当前的阈值；
    // 为什么这里的oldThr在未初始化数组的时候就有值呢？
    // 这是因为HashMap有两个带参构造器，可以指定初始容量，
    // 若你调用了这两个可以指定初始容量的构造器，
    // 这两个构造器就会将阈值记录为第一个大于等于你指定容量，且满足2^n的数（可以看看这两个构造器）
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    // 若上面的条件都不满足，表示你是调用默认构造器创建的HashMap，且还没有初始化table数组
    else {               // zero initial threshold signifies using defaults
        // 则将新容量更新为默认初始容量（16）
        // 阈值即为（容量*负载因子）
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }

    // 经过上面的步骤后，newCap一定有值，但是若运行的是上面的第二个分支时，newThr还是0
    // 所以若当前newThr还是0，则计算出它的值（容量*负载因子）
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }

    // 将计算出的新阈值更新到成员变量threshold上
    threshold = newThr;

    // 创建一个记录新数组用来存HashMap中的元素
    // 若数组不是第一次初始化，则这里就是创建了一个两倍大小的新数组
    @SuppressWarnings({"rawtypes","unchecked"})
    HashMap.Node<K,V>[] newTab = (HashMap.Node<K,V>[])new HashMap.Node[newCap];
    // 将新数组的引用赋值给成员变量table
    table = newTab;

    // 开始将原来的数据加入到新数组中
    if (oldTab != null) {
        // 遍历原数组
        for (int j = 0; j < oldCap; ++j) {
            HashMap.Node<K,V> e;
            // 若原数组的j位置有节点存在，才进一步操作
            if ((e = oldTab[j]) != null) {
                // 清除旧数组对节点的引用
                oldTab[j] = null;
                // 若table数组的j位置只有一个节点，则直接将这个节点放入新数组
                // 使用 & 替代 % 计算出余数，即下标
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 若第一个节点是一个数节点，表示原数组这个位置的链表已经被转为了红黑树
                // 则调用红黑树的方法将节点加入到新数组中
                else if (e instanceof HashMap.TreeNode)
                    ((HashMap.TreeNode<K,V>)e).split(this, newTab, j, oldCap);

                // 上面两种情况都不满足，表示这个位置是一条不止一个节点的链表
                // 以下操作相对复杂，所以单独拿出来讲解
                else { // preserve order
                    HashMap.Node<K,V> loHead = null, loTail = null;
                    HashMap.Node<K,V> hiHead = null, hiTail = null;
                    HashMap.Node<K,V> next;
                    do {
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    // 将新创建的数组返回
    return newTab;
}
```

单独分析中间链表拆分的代码

1. 定义两个链表 (lo 和 hi)， 包括头节点和尾节点

```java
Node<K,V> loHead = null, loTail = null;
Node<K,V> hiHead = null, hiTail = null;
```

2. 按照顺序遍历table携带的链表的每个节点，如果`(e.hash & oldCap) == 0`，就放入`lo`链表，其他的放入`hi`链表

```java
do {
      next = e.next;
      // 根据元素的哈希值和旧容量的位运算结果将元素分类
      if ((e.hash & oldCap) == 0) { // 
          if (loTail == null)
              loHead = e;
          else
              loTail.next = e;
          loTail = e;
      } else {
          if (hiTail == null)
              hiHead = e;
          else
              hiTail.next = e;
          hiTail = e;
      }
  } while ((e = next)!= null);
```

3. 将原来的链表拆分为两个链表，然后将低位置元素存储到新数组原索引位置，将高位置元素存储到新数组原索引加旧容量的位置。 位置的高低按照 `(e.hash & oldCap) == 0` 来区分

```java
 // 将低位置元素存储到新数组原索引位置
    if (loTail!= null) {
        loTail.next = null;
        newTab[j] = loHead;
    }
    // 将高位置元素存储到新数组原索引加旧容量的位置
    if (hiTail!= null) {
        hiTail.next = null;
        newTab[j + oldCap] = hiHead;
    }
```

![HashMap原理之Resize操作](https://oss.swimmingliu.cn/6659dd7d-ef8d-11ef-9aac-c858c0c1deba)