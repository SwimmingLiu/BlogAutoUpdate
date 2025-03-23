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

## [补充] 1. Java 中 HashMap 的扩容机制是怎样的？

**【简单理解】**

1. 每次扩容，新数组变成老数组的 `2` 倍， 所以新数组可以看成是老空间+一块一样大小的新空间。（两块数组）。
2. 扩容的时候，如果原数组是单一的节点，则直接放入新数组里面。假如原数组长度为`n`， 新数组中的位置是`e.hash & 2n -1`，其实就是 `e.hash % 2n`
3. 如果原数组是多个节点，就按照低位和高位来分。区分的方法，哈希值的二进制和原数组长度的二进制的最高位是否为`1`。如果为`0`，则为低位，否则为高位。（比如哈希值为`3`，二进制为`011`, 原数组长度为`4`， 二进制为`100`。就是看第 `3` 位是否为`1`）
4. 然后低位链表，就放在新数组老空间那边。高位链表就放在新数组新空间那边，具体位置是原位置+老数组的长度

**【具体流程】**

1. 判断当前的容量是否已经达到最大容量，如果是，则直接返回原来的哈希表对象，不予扩容。
2. 如果没有达到最大容量，则 `new` 一个原数组 `2` 倍大小的新数组，用来存放新的数据
3. 遍历原数组，将原数组的数据都迁移到新的数组里面。
4. 迁移的过程中，如果数组对应位置上，只有一个节点，直接用元素的哈希值和新数组的长度减`1` 进行与运算 (`index = e.hash & (newCap - 1)`)，得到的结果就是新数组中的位置，复制过去即可
5. 如果不止一个节点，会新建两个头节点和尾节点，可以看成两条链表，分为低位链表和高位链表。把原来的链表/红黑树拆分到低位链表和高位链表上
6. 判断元素是否为`TreeNode`, 来判断是红黑树还是链表结构。
7. 逐一遍历原数组红黑树/链表上的节点，每个节点的哈希值和原数组的长度进行与运算，判断结果是否为 `0` (`e.hash & oldCap == 0`)，如果等于 `0`，则插入低位链表，如果不等于`0`，则插入高位链表。
8. 最后，将低位链表，放入新数组中元素在原数组对应的下标位置。将高位链表放入，从旧位置往后移动原数组长度的位置，也就是`旧位置下标 + 原数组大小`作为高位链表的存放位置
9. 最后，返回新的哈希表对象

![HashMap原理之Resize操作流程图](https://oss.swimmingliu.cn/81948ac6-07e8-11f0-84d9-c858c0c1debd)

## 2.ConcurrentHashMap了解吗? / Java 中 ConcurrentHashMap 1.7 和 1.8 之间有哪些区别？

首先，提到 `ConcurrentHashMap` 我们要分成`JDK 1.7` 和 `JDK 1.8` 两个版本来看：

1. `JDK 1.7`: 在1.7中， `ConcurrentHashMap` 采用分段锁。就是分成不同的`segment`，默认有16个。每个`segment` 中都包含多个的 `HashEntry` (可以理解成一个`HashMap`)。 锁的方式源于`Segment`,这个类实际集成了`ReentrantLock`

   ![ConcurrentHashMapJava7](https://oss.swimmingliu.cn/2de6bbd4-f63f-11ef-a1c4-c858c0c1deba)

2. `JDK 1.8`：在1.8中，`ConcurrnetHashMap`的数据结构和`HashMap`一样，它做了更小范围的锁控制。它的数组的每个位置上都有一把锁。如果需要扩容，会使用`CAS` 自旋操作保证线程安全，避免锁整个数组。如果是在链表/红黑树插入某个`node`，只需要用`synchronize`进行上锁。

   ![ConcurrentHashMapJava8](https://oss.swimmingliu.cn/2e193b43-f63f-11ef-bdde-c858c0c1deba)

**【JDK1.7和1.8扩容区别】**

1. `JDK 1.7`：当某个`Segment`内的`HashMap` 达到扩容阈值的时候，单独为该`Segment`进行扩容。

2. `JDK 1.8`：大致可以分为三个特点全局扩容、基于`CAS`扩容、渐进式扩容

   - **全局扩容**：`1.8` 因为取消了 `1.7` 里面的`Segment`， 本身是数组+链表+红黑树的结构。所以是一个全局的数组，当任意位置的元素超过阈值时，整个数组都会被扩容。

   - **基于 `CAS` 的扩容**： 采用和 `HashMap` 相似的扩容机制，采用 `CAS ` 操作确保线程安全，同时避免锁住整个数组。
   - **渐进式扩容**：扩容不是一次性将所有数据重新分配，而是多个线程共同参与，逐步迁移就数据到新数组当中，降低扩容对性能的消耗。(假如当前数组长度为32，那么可以A线程负责`0~15`，B线程负责`16~31`)

## 3. 为什么 Java 的 ConcurrentHashMap 不支持 key 或 value 为 null？

`ConcurrentMap`不支持`key`或`value`为 `null` 是为了避免歧义和简化代码实现方式

因为多线程环境下，`get(key) ` 方法如果返回 `null` ，不知道其表示的是`key`不存在还是`value`本来就是 `null`。为了避免这个歧义，代码就需要频繁的判断null是代表`key`不存在还是 `value` 本来就是 `null`，增加复杂度。

**【为什么HashMap支持 `key `或 `value `为null】**

因为HashMap设计的初衷就是单线程模式使用的，本身就是线程不安全的。在 HashMap 的实现中，`null` 键被特殊处理。当 `key` 为 `null` 时，HashMap 不会调用 `hashCode()` 方法，而是直接将 `null` 键存储在表的第一个桶（`table[0]`）中。这样可以避免 `NullPointerException` 。

**【注意】** 像`HashTable`、`ConcurrentSkipListMap`、`CopyOnWriteArrayList`这些并发集合，都是线程安全的，都不支持`key`或`value`为 `null`

## 4 . Java 中 ConcurrentHashMap 的 get 方法是否需要加锁？

`ConcurrentHashMap` 的 `get` 方法不需要加锁。因为`get` 方式是读取操作，不需要对资源做任何处理，所以每次只需要保证读取到最新的数据即可，所以不需要加锁。

另外,`ConcurrentHashMap` 中 `get` 方法对于数组中的节点，是通过`Unsafe` 方法 `getObjectVolatile()` 来保证可见性的。对于链表或者红黑树节点，是采用`volatile` 关键字去修饰 `val` 和 `next` 节点的，也可以保证可见性。

## 5. Java 中有哪些集合类？请简单介绍

Java中的集合类都是在`java.util`。 主要可以分为单列类型 `Collection` 和 双列 `Map` 两类来看。 其中单列 `Collection` 里面包括 (`List`、`Set`、`Queue`），具体如下图。

![Java集合结构](https://oss.swimmingliu.cn/7f077950-f8f8-11ef-ad9c-c858c0c1deba)

**【两个基本接口】**

1. **`Collection` 单列集合接口**：一个由单个元素组成的序列，这些元素要符合一条或多条规则。其中，`List` 是有序的，`Set` 是去重的， `Queue` 是符合队列规则的。
2. **`Map` 双列集合接口**： 一组键值对，可以用 `key` 来检索 `value`。 上面的 `ArrayList` 是采用索引来查找一个值。`Map` 可以采用另外一个对象来查找某个对象。

**【List 系列】** `List` 接口的实现类用于存储有序的、允许重复的元素。必须按照元素插入顺序来保存他们。

1. `ArrayList`：擅长随机访问元素，但是在 `List` 的中间插入或者删除元素比较慢。适合读操作多的场景。
2. `LinkedList`：提供理想的顺序访问性能，在 `List` 的中间插入和删除元素的成本都比较低。 `LinkedList` 随机访问性能相对较差， 适合频繁插入和删除的场景。
3. `Vector`：基于动态数据实现，线程安全 (方法加锁)， 效率比较低，已经很少用了。

**【Set 系列】** `Set` 接口的实现类用于存储不重复的元素。继承于`Collection`

1. `HashSet`：无序，采用哈希表存储，查找和插入性能高。
2. `LinkedHashSet`：有序，采用哈希表 + 链表(存储插入顺序)
3. `TreeSet`：排序 (或自定义排序)，采用红黑树实现，查找、插入、删除操作性能高。

**【Queue 队列/优先系列】** `Queue` 接口的实现类用于处理先进先出的队列数据结构。

1. `PriorityQueue`： 基于堆实现，用于优先级队列。元素按自然顺序或自定义顺序排列。不是FIFO的顺序，优先处理优先级搞的元素
2. `LinkedList`： 也实现了`Queue` 接口，支持双端队列结构。
3. `Stack`： 双端队列

**【Map 系列】** 键唯一，值可重复

1. `HashMap`： 无序，数组 + 链表 + 红黑树， `key` 和 `value` 都可以为 `null`
2. `LinkedHashMap`：有序，数组 + 链表 + 红黑树，额外链表记录插入顺序
3. `TreeMap`：红黑树实现，对`key`进行自然排序(或者自定义排序)
4. `HashTable`: 数组 + 链表 + 红黑树，有`sychronized` 锁， 不允许`key` 和 `value` 为 `null` ，线程安全。

## 6.  Java 中的 CopyOnWriteArrayList 是什么？

`CopyOnWriteArrayList` 是一种线程安全的`ArrayList`， 其主要的原理就和名字一样，在写的时候复制，写时复制。

`CopyOnWirteArrayList` 的读操作不需要上锁，但是写操作会锁。而且进行写操作的时候，会复制一份原数组出来，然后在新的数组上进行写操作，读操作还是在老数组的基础上，适合读多写少的场景。到那时复制数组有一定的性能消耗，而且会消耗内存。

![CopyOnWirteArrayList写时复制操作](https://oss.swimmingliu.cn/7f4ec4b7-f8f8-11ef-9bfd-c858c0c1deba)

