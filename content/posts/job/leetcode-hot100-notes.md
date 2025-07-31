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



### 4. 合并K个升序链表

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

### 5. 环形链表

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



### 6. 环形链表 II

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

### 7. 排序链表

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

### 8. 相交链表

题目链接：[相交链表](https://leetcode.cn/problems/intersection-of-two-linked-lists/?favorite=2cktkvj)

【思路】

简单总结：两个人从不同的地方来，同时走过一段旅程之后，从另外一个人的源头再走一遍，一定会重逢。如果没有重逢说明，他们没有一起走过的旅程。

![相交链表](https://oss.swimmingliu.cn/69eb28b2-6aeb-11f0-96f8-caaeffceb345)

【伪代码】

``` java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
        ListNode p = headA;
        ListNode q = headB;
        while (p != q) {
            p = p != null ? p.next : headB;
            q = q != null ? q.next : headA;
        }
        return p;
    }
```

### 9. 反转链表

题目链接：[反转链表](https://leetcode.cn/problems/reverse-linked-list/description/)

【思路】

使用头节点的头插法，创建一个新的 `dummy` 节点，然后采用头插法将元素插入`dummy` 后面，最后`dummpy.next` 就是逆序后的头节点

【伪代码】

```java
public ListNode reverseList(ListNode head) {
      ListNode dummy = new ListNode();
      ListNode cur = head;
      while (cur != null) {
          ListNode nxt = cur.next;
          cur.next = dummy.next;
        	dummy.next = cur;
          cur = nxt;
      }
      return dummy.next;
  }
```

【第二种思路】

直接使用头插法，将后面一个节点放到当前节点的前面，同时将当前节点的下一个节点替换为上一个节点。

【伪代码】

``` java
public ListNode reverseList(ListNode head) {
      ListNode pre = null;
      ListNode cur = head;
      while (cur != null) {
          ListNode nxt = cur.next;
          cur.next = pre;
        	pre = cur;
          cur = nxt;
      }
      return pre;
  }
```



### 10. 回文链表

题目链接：[回文链表](https://leetcode.cn/problems/palindrome-linked-list/description/)

【思路】

这道题应该拆分为两道题来做，一个是找中间节点（快慢指针） + 后半部分链表反转。然后从中间节点开始比较，如果有某一处不相同就返回 `false`， 否则最后返回 `true`

【伪代码】

``` java
public boolean isPalindrome(ListNode head) {
      ListNode mid = middleNode(head); // 寻找中间节点
      ListNode head2 = reverseList(mid); // 反转链表
      while (head2 != null) {
          if (head.val != head2.val) { // 不是回文链表
              return false;
          }
          head = head.next;
          head2 = head2.next;
      }
      return true;
  }
```

## 二叉树

### 1. 二叉树的中序遍历

题目链接：[二叉树的中序遍历](https://leetcode.cn/problems/binary-tree-inorder-traversal/?favorite=2cktkvj)

【思路】

中序遍历：左根右，遍历过程：上下递归，中间输出，边界条件是空

【伪代码】

``` java
void dfs(List<Integer> res, TreeNode root) {
		if(root==null) {
			return;
		}
		//按照 左-打印-右的方式遍历
		dfs(res,root.left);
		res.add(root.val);
		dfs(res,root.right);
	}
```

### 2. 验证二叉搜索树

题目链接：[验证二叉搜索树](https://leetcode.cn/problems/validate-binary-search-tree/?favorite=2cktkvj)

【思路】

二叉搜索树就是根比左边大，比右边小。所以可以采用先序遍历的方式，传入左边合右边的值。递归遍历当前节点和左边节点、右边节点是否都符合规则。如果不符合，则输出 `fasle`。

【伪代码】

``` java
public boolean isValidBST(TreeNode root) {
    return isValidBST(root, Long.MIN_VALUE, Long.MAX_VALUE);
}

private boolean isValidBST(TreeNode node, long left, long right) {
    if (node == null) {
        return true;
    }
    long x = node.val;
    return left < x && x < right &&
           isValidBST(node.left, left, x) &&
           isValidBST(node.right, x, right);
}
```

### 3. 对称二叉树

题目链接：[对称二叉树](https://leetcode.cn/problems/symmetric-tree/description/)

【思路】

对称二叉树：二叉树相对轴是对称的

判断方法是递归判断当前节点的左右节点是否满足轴对称条件。两个节点左右对称的条件是 `l.left = r.right && l.right == r.left` 。边界判断条件为左右节点是否均为空，如果为空说明父节点是叶子节点，表明 满足轴对称。如果只有一个节点为空，或者两个节点的值不同，则说明他们不是轴堆成的。如果左右节点都有值且相同，则继续往下递归。

【伪代码】

``` java
public boolean isSymmetric(TreeNode root) {
    return root == null || recur(root.left, root.right);
}
boolean recur(TreeNode L, TreeNode R) {
    if (L == null && R == null) return true;
    if (L == null || R == null || L.val != R.val) return false;
    return recur(L.left, R.right) && recur(L.right, R.left);
}
```

### 4. 二叉树的层序遍历

题目链接：[二叉树的层序遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/description/?favorite=2cktkvj)

【思路】

二叉树的层序遍历就是用队列的方式实现，具体可以使用 `ArrayDeque` 来存储节点。每次循环都统计队列的大小 (表示有多少个同级的节点)，然后将这些节点放入数组中，并且将他们的左右子节点放入队列中，循环直到队列为空才结束。

【伪代码】

``` java
public List<List<Integer>> levelOrder(TreeNode root) {
    if (root == null) {
        return List.of();
    }
    List<List<Integer>> ans = new ArrayList<>();
    Queue<TreeNode> q = new ArrayDeque<>();
    q.add(root);
    while (!q.isEmpty()) {
        int n = q.size();
        List<Integer> vals = new ArrayList<>(n); // 预分配空间
        while (n-- > 0) {
            TreeNode node = q.poll();
            vals.add(node.val);
            if (node.left != null)  q.add(node.left);
            if (node.right != null) q.add(node.right);
        }
        ans.add(vals);
    }
    return ans;
}
```

### 5. 二叉树的最大深度

题目链接：[二叉树的最大深度](https://leetcode.cn/problems/maximum-depth-of-binary-tree/description/)

【思路】

最大深度可以采用后序遍历、先序遍历或者层序遍历，不过一般都使用后序遍历或者先序遍历来做。可以按照先序遍历和后序遍历分为两种方案，一种是自顶向下，一种是自底向上的方式。

- 自顶向下 (先序遍历)：先设置 `depth` 为 0，用一个全局的 `answer` 来记录答案。按照后序遍历的顺序，一定能遍历到最下面的一层，每次将 `answer` 更新为最大深度即可。
- 自底向上 (后序遍历)：采用后序遍历递归获取左右节点的最长深度，加上当前层的深度就是二叉树的最大深度

【伪代码】

``` java
// 自顶向下：先序遍历，ans是全局变量
public int maxDepth(TreeNode root) {
    dfs(root, 0);
    return ans;
}

private void dfs(TreeNode node, int depth) {
    if (node == null) {
        return;
    }
    depth++;
    ans = Math.max(ans, depth);
    dfs(node.left, depth);
    dfs(node.right, depth);
}
// 自底向上：后序遍历
public int maxDepth(TreeNode root) {
    if (root == null) {
        return 0;
    }
    int lDepth = maxDepth(root.left);
    int rDepth = maxDepth(root.right);
    return Math.max(lDepth, rDepth) + 1;
}
```

### 6. 从前序与中序遍历序列构造二叉树

题目链接：[从前序与中序遍历序列构造二叉树](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/description/)

【思路】

手推：先按照手推的方式思考，用先序和中序构成二叉树的方法：先序列表用于确定根节点，中序列表用于确定左右子树。然后逐个确定每一层的根节点，及其左右子树。

程序：分析手推的方式可以发现确定每一层根节点及其左右子树的方法是重复的，可以采用递归的方式完成。所以我们可以先通过先序找到根节点的位置，再按照根节点的位置将先序列表和中序列表一分为二。然后分别将先序和中序的两个左子树数组进行递归构建，再将两个右子树数组进行递归构建。

![从先序和中序构建二叉树](https://oss.swimmingliu.cn/6a8af4e6-6aeb-11f0-96f8-caaeffceb345)

【伪代码】

``` java
public TreeNode buildTree(int[] preorder, int[] inorder) {
    int n = preorder.length;
    if (n == 0) { // 空节点
        return null;
    }
    int leftSize = indexOf(inorder, preorder[0]); // 左子树的大小
    int[] pre1 = Arrays.copyOfRange(preorder, 1, 1 + leftSize); // 先序左子树
    int[] pre2 = Arrays.copyOfRange(preorder, 1 + leftSize, n); // 先序右子树
    int[] in1 = Arrays.copyOfRange(inorder, 0, leftSize);       // 中序左子树
    int[] in2 = Arrays.copyOfRange(inorder, 1 + leftSize, n);   // 中序右子树
    TreeNode left = buildTree(pre1, in1);
    TreeNode right = buildTree(pre2, in2);
    return new TreeNode(preorder[0], left, right);
}

// 获取左子树大小：返回 x 在 a 中的下标，保证 x 一定在 a 中
private int indexOf(int[] a, int x) {
    for (int i = 0; ; i++) {
        if (a[i] == x) {
            return i;
        }
    }
}
```



### 7. 将有序数组转换为二叉搜索树

题目链接：[将有序数组转换为二叉搜索树](https://leetcode.cn/problems/convert-sorted-array-to-binary-search-tree/description/)

【思路】

有序数组其实是二叉搜索树的中序遍历，则说明中间的位置就是根节点，左右两边的区间分别为左子树和右子树。从根节点开始，循环递归的构建左子树和右子树，就可以的到二叉搜索树。
【注】当数组长度 `n` 为偶数的时候，可以去中间左边的节点，也可以取中间右边的结果，所以答案不唯一。下面的伪代码是取得中间右边的节点。

【伪代码】

``` java
public TreeNode sortedArrayToBST(int[] nums) {
    return dfs(nums, 0, nums.length);
}

// 把 nums[left] 到 nums[right-1] 转成平衡二叉搜索树
private TreeNode dfs(int[] nums, int left, int right) {
    if (left == right) {
        return null;
    }
    int m = (left + right) >>> 1;
    return new TreeNode(nums[m], dfs(nums, left, m), dfs(nums, m + 1, right));
}
```

### 8. 二叉树展开为链表

题目链接：[二叉树展开为链表](https://leetcode.cn/problems/flatten-binary-tree-to-linked-list/?favorite=2cktkvj)

【思路】

将二叉树展开为链表（还是二叉树结构），其实是按照二叉树的先序遍历顺序进行展开(根-左-右)。如果要构建这个展开后的链表，可以按照相反的方向(右-左-根)的方向进行构建。

只需要采用头插法，从相反方向(右-左-根)的第一个元素开始，用 `pre` 记录上一个节点，递归构建展开后的链表即可。

【伪代码】

``` java
public void flatten(TreeNode root) {
    if (root == null) {
        return;
    }
    flatten(root.right); // 右节点
    flatten(root.left);  // 左节点
    root.left = null;  // 左子树置空
    root.right = pre; // 头插法，相当于链表的 root.next = head
    pre = root; // 现在链表头节点是 root
}
```

### 9. 翻转二叉树

题目链接：[翻转二叉树](https://leetcode.cn/problems/invert-binary-tree/description/?favorite=2cktkvj)

【思路】

翻转二叉树：翻转每个节点的左子树和右子树。所以，可以从根节点开始，递归翻转左子树和右子树，直到子树为空为止。

【伪代码】

``` java 
 public TreeNode invertTree(TreeNode root) {
 		if (root == null){
      	return null;
    }
    TreeNode left = invertTree(root.left);
   	TreeNode right = invertTree(root.right);
   	root.left = right; // 交换左右子树的值
    root.right = left;
    return root;
 }
```

### 10. 二叉树的最近公共祖先

题目链接：[二叉树的最近公共祖先](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/description/)

> 对于有根树 T 的两个节点 p、q，最近公共祖先表示为一个节点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。

【思路】

分析：该题目需要找到两个节点的公共祖先，应该是一个从下往上找的过程，所以应该选择后序遍历的方式。

首先基于某个节点，分析二叉树最近公共祖先可能出现的位置：

1. 当 `p` 和 `q` 位于当前节点的左右子树，则说明最近公共祖先就是当前节点
2. 当 `p` 和 `q` 均位于当前节点的某一个子树，则说明最近公共祖先就在这个子树中
3. 如果当前节点是 `p` 和 `q` 中的某一个，并且另外一个节点在当前子树中，则说明当前节点是最近公共祖先节点。

解决方案：对于从底向上的某一个节点，应该判断下面的几个条件：

1. 如果当前节点为 `null` 或者为 `p` 和 `q` 中的某一个，则可以直接返回节点。 因为是自底向上的，如果子树存在另外一个节点，说明当前节点是最近公共祖先。如果不存在，则也没有必要去探寻一遍。
2. 如果当前节点的左右子树分别包含 `p`  和  `q` ，则说明当前节点就是最近公共祖先，直接返回当前节点。
3. 如果当前节点的左右子树中，某一个子树包含 `p` 和 `q` ，则说明当前节点不是最近公共祖先，返回该子树中的结果值。因为可能他的子树中，同时包含 `p` 和 `q` ，则向上传递的节点就是最新公共祖先。如果不是，则还需要向上继续判断。

【伪代码】

``` java 
public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
    // 基本情况处理：
    // 1.如果当前节点为空，返回null
    // 2.如果当前节点是p或q中的任一个，直接返回该节点
    if (root == null || root == p || root == q) {
        return root;
    }
    // 递归搜索左子树 
    TreeNode findInLeftTree = lowestCommonAncestor(root.left, p, q);
    // 递归搜索右子树
    TreeNode findInRightTree = lowestCommonAncestor(root.right, p, q);
    // 情况1: p和q分别位于当前节点的左右子树
    // 此时当前节点就是最近公共祖先
    if (findInLeftTree != null && findInRightTree != null) {
        return root;
    }
    // 情况2: p和q都在左子树中
    if (findInLeftTree != null) {
        return findInLeftTree;
    }
    // 情况3: p和q都在右子树中
    // 情况4: p和q都不在当前子树中(此时right为null)
    return findInRightTree;
}
```

### 11. 把二叉搜索树转换为累加树

题目链接：[把二叉搜索树转换为累加树](https://leetcode.cn/problems/convert-bst-to-greater-tree/description/)

【思路】

题目的意思是在每个节点上计算累加和，从最大的节点开始进行递归计算。因为原来的树为二叉搜索树，所以最大的节点在最右下方。二叉搜索树中，右子树节点 > 根节点 > 左子树节点。所以可以按照 `右-根-左` 的遍历方式进行遍历，用一个全局变量来记录累加值。

【伪代码】

``` java
private int s = 0; // 全局变量记录累加值
public TreeNode convertBST(TreeNode root) {
    dfs(root);
    return root;
}

private void dfs(TreeNode node) {
    if (node == null) {
        return;
    }
    dfs(node.right); // 递归右子树
    s += node.val;
    node.val = s; // 此时 s 就是 >= node.val 的所有数之和
    dfs(node.left); // 递归左子树
}
```

### 12. 二叉树的直径

题目链接：[二叉树的直径](https://leetcode.cn/problems/diameter-of-binary-tree/description/)

【思路】

链长：定义从最底的叶子节点到当前节点的距离为链长，则空节点的链长为 `-1`，因为它还在叶子节点下面。

当前节点的直径：当前节点的左子树链长+当前节点的右子树链长

所以，只需要设置一个全局的变量 `ans` ，然后从根节点递归计算每一个节点的左右子树链长，然后获取 `ans` 和当前节点直径的最大值，每次返回左右子树的最大链长即可。边界条件是当节点为空的时候，返回的链长为 `-1`。

【伪代码】

``` java
private int ans; // 全局变量记录结果
public int diameterOfBinaryTree(TreeNode root) {
    dfs(root);
    return ans;
}

private int dfs(TreeNode node) {
    if (node == null) {
        return -1; // 对于叶子来说，链长就是 -1+1=0
    }
    int lLen = dfs(node.left) + 1; // 左子树最大链长+1
    int rLen = dfs(node.right) + 1; // 右子树最大链长+1
    ans = Math.max(ans, lLen + rLen); // 两条链拼成路径
    return Math.max(lLen, rLen); // 当前子树最大链长
}
```

### 13. 合并二叉树

题目链接：[合并二叉树](https://leetcode.cn/problems/merge-two-binary-trees/?favorite=2cktkvj)

【思路】

如果 `root1` 为空，则直接返回 `root2` 。如果 `root2` 为空，则直接返回 `root1`。如果都不为空，则返回一个新的节点，并递归合并他的左右子树。 

【伪代码】

``` java 
public TreeNode mergeTrees(TreeNode root1, TreeNode root2) {
    if (root1 == null) return root2;
    if (root2 == null) return root1;
    return new TreeNode(root1.val + root2.val,
        mergeTrees(root1.left, root2.left),    // 合并左子树
        mergeTrees(root1.right, root2.right)); // 合并右子树
}
```

## DFS/BFS

### 1. 单词搜索

题目链接：[单词搜索](https://leetcode.cn/problems/word-search/description/)

【思路】

遍历 `board` 网格中的所有字符，对每个字符进行递归遍历 `dfs(i, j, k)` ，其中 `i` 和 `j` 分别表示 `board` 网格中的位置，`k` 表示当前需要验证的 `word[k]` 。判断 `board[i][j] == word[k]`， 如果不相等，则直接返回 `false`。如果相等，则查询四周是否存在 `word [k + 1]`，即遍历 `dfs (x, y, k + 1)` 。遍历 `dfs(x, y, k + 1)` 前，将 `board[i][j]` 标记为 `-1`， 遍历后再恢复现场，用于表示标识 `board[i][j]` 是否被使用。最后返回当 `k == word.size() - 1` ，则返回 `true`。

- 优化点1: 可以先统计 `word` 中所有字符出现的次数，以及 `board` 网格所有字符出现的次数。如果 `word` 中某个字符出现的次数大于 `board` 网格中出现的次数，则直接返回 `false`

- 优化点2:  判断 `word` 中首尾单词出现的次数，从次数较少的一头开始比较，可以减少比较次数。注意，如果要用末尾的开始，需要将 `word` 数组进行逆序。 

【伪代码】

``` java
private static final int[][] DIRS = {{0, -1}, {0, 1}, {-1, 0}, {1, 0}}; // 四个方向
public boolean exist(char[][] board, String word) {
    // 为了方便，直接用数组代替哈希表
    int[] cnt = new int[128];
    for (char[] row : board) {
        for (char c : row) {
            cnt[c]++;
        }
    }

    // 优化一
    char[] w = word.toCharArray();
    int[] wordCnt = new int[128];
    for (char c : w) {
        if (++wordCnt[c] > cnt[c]) {
            return false;
        }
    }

    // 优化二
    if (cnt[w[w.length - 1]] < cnt[w[0]]) {
        w = new StringBuilder(word).reverse().toString().toCharArray();
    }

    for (int i = 0; i < board.length; i++) {
        for (int j = 0; j < board[i].length; j++) {
            if (dfs(i, j, 0, board, w)) {
                return true; // 搜到了！
            }
        }
    }
    return false; // 没搜到
}

private boolean dfs(int i, int j, int k, char[][] board, char[] word) {
    if (board[i][j] != word[k]) { // 匹配失败
        return false;
    }
    if (k == word.length - 1) { // 匹配成功！
        return true;
    }
    board[i][j] = 0; // 标记访问过
    for (int[] d : DIRS) {
        int x = i + d[0];
        int y = j + d[1]; // 相邻格子
        if (0 <= x && x < board.length && 0 <= y && y < board[x].length && dfs(x, y, k + 1, board, word)) {
            return true; // 搜到了！
        }
    }
    board[i][j] = word[k]; // 恢复现场
    return false; // 没搜到
}
```

### 2. 岛屿数量

题目链接：[岛屿数量](https://leetcode.cn/problems/number-of-islands/description/)

【思路】

遍历网格中所有为 `1` 的位置，递归判断上、下、左、右是否有为 `1` 的格子。如果没有，则直接返回。如果有，则标记当前的格子，并且继续往后递归判断。每遍历完一次，说明发现了一个完整的岛屿，则答案加 1。然后继续找下一个为 `1` 的位置，继续递归判断。
【注】 递归有回退的操作，案例1里面是会回退回来遍历所有部分的，不是最后一个不满足条件就直接结束了。

【伪代码】

``` java 
public int numIslands(char[][] grid) {
    int ans = 0;
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[i].length; j++) {
            if (grid[i][j] == '1') { // 找到了一个新的岛
                dfs(grid, i, j); // 把这个岛插满旗子，这样后面遍历到的 '1' 一定是新的岛
                ans++;
            }
        }
    }
    return ans;
}

private void dfs(char[][] grid, int i, int j) {
    // 出界，或者不是 '1'，就不再往下递归
    if (i < 0 || i >= grid.length || j < 0 || j >= grid[0].length || grid[i][j] != '1') {
        return;
    }
    grid[i][j] = '2'; // 插旗！避免来回横跳无限递归
    dfs(grid, i, j - 1); // 往左走
    dfs(grid, i, j + 1); // 往右走
    dfs(grid, i - 1, j); // 往上走
    dfs(grid, i + 1, j); // 往下走
}
```

### 3. 路径总和 III

题目链接：[路径总和 III](https://leetcode.cn/problems/path-sum-iii/description/)

【思路】

首先，要找到一条路径的总合为 `targetNum`， 可以利用前缀和的特性。例如，当 `targetNum = 8`， 三个节点的前缀和数组为 `10, 15, 18` ，因为 `18 - 10 = 8 = targetNum`， 说明后面两个节点可以组成一条路径和为 `targetNum`。其次，判断的时候，以第三个节点为终点，判断存在多少条路径的前缀和为 `18 - targetNum = 10`，就说明有多少条路径和为 `targetNum` 。所以，按照先序遍历的顺序，用 `cnt` 的 Map对象记录前缀和的个数。遍历的过程中，将当前前缀和的值放入 `cnt` 中。

注意，为了防止出现 `targetNum = 8`， 第一个节点刚好为 `8`，导致漏算的情况。可以初始化 `cnt(0, 1)`，也就是 `8 - targeNum = 0`，本身也是一条路径。

【伪代码】

``` java
private int ans; // 全局记录答案
public int pathSum(TreeNode root, int targetSum) {
    Map<Long, Integer> cnt = new HashMap<>();
    cnt.put(0L, 1);
    dfs(root, 0, targetSum, cnt);
    return ans;
}
private void dfs(TreeNode node, long s, int targetSum, Map<Long, Integer> cnt) {
    if (node == null) {
        return;
    }

    s += node.val;
    // 把 node 当作路径的终点，统计有多少个起点
    ans += cnt.getOrDefault(s - targetSum, 0);

    cnt.merge(s, 1, Integer::sum); // cnt[s]++
    dfs(node.left, s, targetSum, cnt);
    dfs(node.right, s, targetSum, cnt);
    cnt.merge(s, -1, Integer::sum); // cnt[s]-- 恢复现场，防止遍历完左子树，遍历右子树的时候出现前缀合多加上左子树值的情况。
}
```



## 递归/回溯

### 1. 电话号码的字母组合

题目链接：[电话号码的字母组合](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/description/)

【思路】

用一个字符串数组 `MAP` 记录每个数字对应的多个字符，用 `path` 数组记录组合的字符串，长度等于 `digits` 的长度。从 `0` 开始，`dfs(i)`递归获取每个数字的可能性，递归的过程中循环递归  `MAP` 中的多种组合 `dfs(i + 1)`（需要恢复现场）。边界条件为 `i == len`， 直接将 `path` 中的结果添加到 `ans` 中。

【伪代码】

``` java
private static final String[] MAPPING = new String[]{"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};

public List<String> letterCombinations(String digits) {
    int n = digits.length();
    if (n == 0) {
        return List.of();
    }
    List<String> ans = new ArrayList<>();
    char[] path = new char[n]; // 注意 path 长度一开始就是 n，不是空数组
    dfs(0, ans, path, digits.toCharArray());
    return ans;
}

private void dfs(int i, List<String> ans, char[] path, char[] digits) {
    if (i == digits.length) {
        ans.add(new String(path));
        return;
    }
    String letters = MAPPING[digits[i] - '0'];
    for (char c : letters.toCharArray()) {
        path[i] = c; // 直接覆盖 (相当于赋值+恢复现场一体)
        dfs(i + 1, ans, path, digits);
    }
}
```

### 2. 括号生成

题目链接：[括号生成](https://leetcode.cn/problems/generate-parentheses/description/)

【思路】

`dfs(i)` 递归枚举 `n` 个位置中的每个位置，应该填左括号还是右括号, 用 `path` 数组记录括号的组成结果。根据括号的特性，对于每个位置，可以确定下面两个特性：

1. 左括号的数或者右括号的数都小于 `n`，当前位置才可以填左括号/右括号 
2. 右括号的个数一定小于左括号的个数，当前位置才可以填右括号

分别用 `left` 和 `right` 记录 `path` 数组中左括号和右括号的个数，如果满足上面的连个特性，就向下递归 `dfs(i + 1)`。边界条件为 `i == n` , 说明当前 `path` 符合括号的特性，则加入最终的 `ans` 数组 

【伪代码】

``` java
public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<>();
        char[] path = new char[n * 2]; // 所有括号长度都是一样的 2n
        dfs(0, 0, n, path, ans); // 一开始没有填括号
        return ans;
    }

// 目前填了 left 个左括号，right 个右括号
private void dfs(int left, int right, int n, char[] path, List<String> ans) {
    if (right == n) { // 填完 2n 个括号
        ans.add(new String(path));
        return;
    }
    if (left < n) { // 可以填左括号
        path[left + right] = '('; // 直接覆盖
        dfs(left + 1, right, n, path, ans);
    }
    if (right < left) { // 可以填右括号
        path[left + right] = ')'; // 直接覆盖
        dfs(left, right + 1, n, path, ans);
    }
}
```

【第二种思路】

递归枚举左括号的位置，假设当前的位置，已填写的括号为 `i` ，其中 `左括号 - 右括号` 的个数为 `balance` 。 说明后面还可以填写 `balance` 个右括号。那么，我们可以根据右括号出现的情况，来确定左括号的位置。例如从当前位置填写 `k` 个右括号，则左括号的位置为 `i + k`  ( `0 <= k <= balance`)。那么下一个需要确定的左括号的位置就是 `i + k + 1`。此时， `左括号 - 右括号` 的个数为 `balance - k + 1`  ，这里的 ` + 1` 表示 `i + k` 位置的左括号多了一个。 

用 `path` 数组记录每一个左括号的位置，每次递归之后恢复现场，移除最后一个元素 (上一个左括号的位置)。当 `path.size() == n` ，说明已经枚举完所有左括号的位置，则可以将结果添加到 `ans` 里面。 

【第二种伪代码】

``` java
public List<String> generateParenthesis(int n) {
        List<String> ans = new ArrayList<>();
        List<Integer> path = new ArrayList<>();
        dfs(0, 0, n, path, ans);
        return ans;
    }
// 目前填了 i 个括号
// 这 i 个括号中的左括号个数 - 右括号个数 = balance
private void dfs(int i, int balance, int n, List<Integer> path, List<String> ans) {
    if (path.size() == n) {
        char[] s = new char[n * 2];
        Arrays.fill(s, ')');
        for (int j : path) {
            s[j] = '(';
        }
        ans.add(new String(s));
        return;
    }
    // 枚举填 right=0,1,2,...,balance 个右括号
    for (int right = 0; right <= balance; right++) {
        // 先填 right 个右括号，然后填 1 个左括号，记录左括号的下标 i+right
        path.add(i + right);
        dfs(i + right + 1, balance - right + 1, n, path, ans);
        path.removeLast(); // path.remove(path.size() - 1);
    }
}
```

### 3. 组合总和

题目链接：[组合总和](https://leetcode.cn/problems/combination-sum/description/)

【思路】

按照 `candidate` 中的元素进行递归枚举，``dfs(i, left)` 递归枚举当前位置的数选还是不选，如果不选则直接 `dfs(i + 1, left)`。如果选，则将 `candidates[i]` 加入 `path` 数组中，继续递归 `dfs(i , left - candidates[i])`，递归后恢复现场，将 `path` 数组最后一个元素移除。边界条件为 `left == 0`，则将 `path` 数组中的结果加入 `ans` 数组中。如果 `i == candidates.size() || left < 0` ，则说明当前组合的和不可能为 `targetNum` ，直接返回。 其中， `left` 表示剩余需要判断的值，初始值为 `targetNum`。

优化点：可以将 `candidates` 数组先进行快速排序，然后判断 `left < candidates[i]` 。则说明后面的数，不可能满足条件。

【伪代码】

``` java 
public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> ans = new ArrayList<>();
        List<Integer> path = new ArrayList<>();
        dfs(0, target, candidates, ans, path);
        return ans;
    }
private void dfs(int i, int left, int[] candidates, List<List<Integer>> ans, List<Integer> path) {
    if (left == 0) {
        // 找到一个合法组合
        ans.add(new ArrayList<>(path));
        return;
    }
    if (i == candidates.length || left < candidates[i]) {
        // 如果candidates[i] > left，说明后面的数都比 left 要大，path数组的和不可能为left
        return;
    }

    // 不选
    dfs(i + 1, left, candidates, ans, path);

    // 选
    path.add(candidates[i]);
    dfs(i, left - candidates[i], candidates, ans, path);
    path.remove(path.size() - 1); // 恢复现场
}
```

【第二种思路】

按照位置进行枚举，递归枚举当前位置应该选哪一个数，从第 `i` 个位置开始，循环枚举 `candidates` 数组的所有元素。边界条件是 `left == 0`， 则说明找到了一个 `path` 组合能够组成 `targetNum`。

【第二种伪代码】

``` java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
        Arrays.sort(candidates);
        List<List<Integer>> ans = new ArrayList<>();
        List<Integer> path = new ArrayList<>();
        dfs(0, target, candidates, ans, path);
        return ans;
    }
private void dfs(int i, int left, int[] candidates, List<List<Integer>> ans, List<Integer> path) {
    if (left == 0) {
        // 找到一个合法组合
        ans.add(new ArrayList<>(path));
        return;
    }

    // 枚举当前位置，选哪个元素
    for (int k = i; k < candidates.length && candidates[k] <= left; k++) {
        path.add(candidates[k]);
        dfs(k, left - candidates[k], candidates, ans, path);
        path.remove(path.size() - 1); // 恢复现场
    }
}
```
