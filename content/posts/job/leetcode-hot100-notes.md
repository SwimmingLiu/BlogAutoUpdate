---
title: "Leetcode Hot100 刷题笔记"
date: 2025-07-20T23:27:35+08:00
lastmod: 2025-07-20T23:27:35+08:00
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

## 链表

### 1. 两数相加

题目链接：[两数相加](https://leetcode.cn/problems/add-two-numbers/description/)

【思路】

 引入一个临时变量 `carry` 记录进位的值，默认是0。开一个新的链表 `l` ，同时遍历两个链表的值。`l1.val` + `l2.val` +  `carry` = `l.val`。注意，最后有多余的进位时，要新增一个节点。

整个过程可以用递归来实现，递归的边界条件是当 `l1` 、`l2` 为 `null` 且 `carry` 为 `0` 的时候 。然后，返回值是 `new ListNode(carry % 10, addTwo(l1, l2, carry / 10))` 。其中, `carry % 10` 表示当前值， `carry / 10` 表示进位值。计算过程是`l1` 和 `l2` 都获取 `val` 和 `carry` 相加，并且向前遍历。 

【伪代码】

``` java
// l1 和 l2 为当前遍历的节点，carry 为进位， 默认为0
private ListNode addTwo(ListNode l1, ListNode l2, int carry) {
    if (l1 == null && l2 == null && carry == 0) { // 递归边界
        return null;
    }
    int s = carry;
    if (l1 != null) {
        s += l1.val; // 累加进位与节点值
        l1 = l1.next;
    }
    if (l2 != null) {
        s += l2.val;
        l2 = l2.next;
    }

    // s 除以 10 的余数为当前节点值，商为进位
    return new ListNode(s % 10, addTwo(l1, l2, s / 10));
}
```

### 2. 删除链表的倒数第 N 个结点

题目链接：[删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/description/)

【思路】

 链表类型找倒数第 `x` 个节点，可以肌肉反应想到是用左右指针。右指针先走 `x` 步，然后左右指针一起向前遍历，直到右指针为 `null` ，则左指针位置就为倒数第 `x` 个节点。题目要删除倒数第 `n` 个节点，可以转换为找倒数第 `n + 1` 个节点。为了防止链表长度刚好为 `n + 1`， 可以让左右指针和额外的 `dummy` 指针指向头节点。然后用左指针找到倒数第 `n + 1` 个节点，删除后一个节点，最后返回 `dummy` 的下一个节点，就是头节点。

【伪代码】

``` java
public ListNode removeNthFromEnd(ListNode head, int n) {
    // 由于可能会删除链表头部，用哨兵节点简化代码
    ListNode dummy = new ListNode(0, head);
    ListNode left = dummy;
    ListNode right = dummy;
    while (n-- > 0) {
        right = right.next; // 右指针先向右走 n 步
    }
    while (right.next != null) {
        left = left.next;
        right = right.next; // 左右指针一起走
    }
    left.next = left.next.next; // 左指针的下一个节点就是倒数第 n 个节点
    return dummy.next;
}
```

### 3. 合并两个有序链表

题目链接：[合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/?favorite=2cktkvj)

【思路】创建一个哨兵节点  `dummy`  和 新链表指针 `cur`，指向 `dummy` 对应的头 节点 。同时遍历两个有序链表，比较值的大小。将值小的节点作为 `cur` 的 `next` , 然后让 `cur` 和 值小的链表同时向前遍历一个。 最后，判断哪一个链表不会空，将其接到 `cur` 后面，再返回 `dummy.next` 即可。

【伪代码】

``` java
public ListNode mergeTwoLists(ListNode list1, ListNode list2) {
        ListNode dummy = new ListNode(); // 用哨兵节点简化代码逻辑
        ListNode cur = dummy; // cur 指向新链表的末尾
        while (list1 != null && list2 != null) {
            if (list1.val < list2.val) {
                cur.next = list1; // 把 list1 加到新链表中
                list1 = list1.next;
            } else { // 注：相等的情况加哪个节点都是可以的
                cur.next = list2; // 把 list2 加到新链表中
                list2 = list2.next;
            }
            cur = cur.next;
        }
        cur.next = list1 != null ? list1 : list2; // 拼接剩余链表
        return dummy.next;
}
```



## 4. 合并K个升序链表

题目链接：[合并K个升序链表](https://leetcode.cn/problems/merge-k-sorted-lists/description/?favorite=2cktkvj)

【思路】

要将K个升序链表合成一个升序链表，合成的顺序肯定是，依次找最小的节点。第一个最小的节点，肯定是在某个升序链表的表头。但是，第二个最小的节点，可能是在升序链表的表头，也可能是第一个最小节点的后一个节点。

所以合并顺序就是从 `K` 数中找出最小的值加入新链表，然后插入最小节点的后一个节点，反复执行这个步骤。其实就是一个最小堆的概念。所以，我们只需要维护一个最小堆，先将 `K` 个链表的表头节点插入最小堆。然后选出最小节点加入新链表，再将最小节点的后一个节点加入最小堆里面。反复执行上面的操作，直到最小堆中所有值被取出来，就组成了新的链表。

【伪代码】

【注】`new PriorityQueue<>((a, b) -> a.val - b.val)` 中，`PriorityQueue` 的优先级取决于 `lambda` 表达式的正负。如果 `lambda` 表达式为负数，则优先级高，反之亦然。

``` java
public ListNode mergeKLists(ListNode[] lists) {
      // 用PriorityQueue优先队列构建最小堆
      PriorityQueue<ListNode> pq = new PriorityQueue<>((a, b) -> a.val - b.val);
      for (ListNode head : lists) {
          if (head != null) {
              pq.offer(head); // 把所有非空链表的头节点入堆
          }
      }

      ListNode dummy = new ListNode(); // 哨兵节点，作为合并后链表头节点的前一个节点
      ListNode cur = dummy;
      while (!pq.isEmpty()) { // 循环直到堆为空
          ListNode node = pq.poll(); // 剩余节点中的最小节点
          if (node.next != null) { // 下一个节点不为空
              pq.offer(node.next); // 下一个节点有可能是最小节点，入堆
          }
          cur.next = node; // 把 node 添加到新链表的末尾
          cur = cur.next; // 准备合并下一个节点
      }
      return dummy.next; // 哨兵节点的下一个节点就是新链表的头节点
}
```

## 5. 环形链表

题目链接：[环形链表](https://leetcode.cn/problems/linked-list-cycle/?favorite=2cktkvj)

【思路】

判断是否有链表是否有环形可以采用快慢指针，可以参考龟兔赛跑。如果乌龟和兔子同时出发，跑道是环形的，那么兔子一定会追上乌龟。同样，一个有环形的链表，快指针一定可以追上慢指针。

【伪代码】

``` java
public boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head; // 乌龟和兔子同时从起点出发
    while (fast != null && fast.next != null) {
        slow = slow.next; // 乌龟走一步
        fast = fast.next.next; // 兔子走两步
        if (fast == slow) { // 兔子追上乌龟（套圈），说明有环
            return true;
        }
    }
    return false; // 访问到了链表末尾，无环
}
```



## 6. 环形链表 II

题目链接：[环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/description/?favorite=2cktkvj)

【口诀记忆】**快慢相遇，头慢同步。** **再会之处，便是环口。**

【思路】

二级结论：环形链表的入环位置就是快慢指针相遇后，慢指针和头指针相遇的位置。	

分析：假设快慢指针相遇的时候，慢指针走了 `b` 步，快指针走了 `2b` 步，再设入环的位置需要走 `a` 步，环的长度为 `c` 。 因为快指针和慢指针都会走入环的这段距离 ( `a` 步)，剩下的路程都是在环内绕圈。则他们相差的距离 `2b - b = kc` （龟兔赛跑中，兔子一定比乌龟多跑 `k` 圈，才会相遇）->  `b = kc`。又因为 `b - a = kc - a` -> `(b - a) + a = (kc - a) + a = kc`，其中 `b - a` 是快慢指针第一次相遇，慢指针在环中走的步数。走 `kc`步，刚好回到起点。 说明从相遇位置再走 `a` 步就是入环的位置。 

![图解环形链表](https://oss.swimmingliu.cn/12784a00-6894-11f0-bd98-caaeffceb345)

注 1：因为 `(kc − a) + a = kc`，从 `kc − a` 开始，再走 `a` 步，就可以走满 `k` 圈。想象你在操场上跑步，从入环口开始跑，跑满 k 圈，你现在人在哪？刚好在入环口。

注 2：慢指针从相遇点开始，移动 a 步后恰好走到入环口，但在这个过程中，可能会多次经过入环口。

【伪代码】

``` java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (fast == slow) { // 相遇
            while (slow != head) { // 再走 a 步 (头指针走a步 = 慢指针从相遇位置走a步)
                slow = slow.next;
                head = head.next;
            }
            return slow;
        }
    }
    return null;
}
```

## 7. 排序链表

题目链接：[排序链表](https://leetcode.cn/problems/sort-list/description/)

【思路】

1. 按照分而治之的思想，将链表从中间分为两段，确保左右两段都有序。再合并两个有序链表即可。对于两段链表进行排序，可以再采取这个思路，将链表分为两段有序链表，再进行合并。一直划分到只有一个节点或者链表没有节点（奇数个）为止
2. 链表找中点：快慢指针一起走，快指针结束，慢指针刚好在中点
3. 合并两个有序链表：双指针比较大小

【复杂度分析】

- 时间复杂度：`O(nlogn)`，其中 n 是链表长度。递归式 `T(n) = 2T(n/2) + O(n)`，由主定理可得时间复杂度为 `O(nlogn)`。
- 空间复杂度：`O(logn)`。递归需要 `O(logn)` 的栈开销。

【伪代码】

``` java
public ListNode sortList(ListNode head) {
    // 如果链表为空或者只有一个节点，无需排序
    if (head == null || head.next == null) {
        return head;
    }
    // 找到中间节点 head2，并断开 head2 与其前一个节点的连接
    // 比如 head=[4,2,1,3]，那么 middleNode 调用结束后 head=[4,2] head2=[1,3]
    ListNode head2 = middleNode(head);
    // 分别排序左边和右边
    head = sortList(head);
    head2 = sortList(head2);
    // 合并 -> 双指针 + 比较大小
    return mergeTwoLists(head, head2);
}
```