---
title: "美团历年算法真题合集"
date: 2025-08-28T21:27:35+08:00
lastmod: 2025-08-28T21:27:35+08:00
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

# 美团笔试合集

## 2024.09.07

### 1. 第一题

小美有一个长度为n的数组，她每次操作会执行如下：选定一个ai，把这个数加上一个任意的x(x > 0)，花费的代价为ai + x。现在小美想要把整个数组变成全部奇数或者全部偶数的最小代价是多少？

#### 输入描述

第一行输入一个整数n，代表数组的长度

第二行输入n个数

#### 输出描述

输出最小花费

#### 样例输入

> 3
>
> 1 2 3

#### 样例输出

> 3

#### 答案

> 要么统一把奇数编程偶数，要么统一把偶数变成奇数，只需要计算两种情况的最小值就行

对变成奇数和变成偶数的两种情况分别进行计算。 

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        List<Integer> a = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            a.add(scanner.nextInt());
        }

        int oddCost = 0, evenCost = 0;
        for (int i = 0; i < n; i++) {
            if (a.get(i) % 2 == 1) {
                evenCost += a.get(i) + 1;
            } else {
                oddCost += a.get(i) + 1;
            }
        }

        System.out.println(Math.min(oddCost, evenCost));
    }
}
```

### 2. 第二题

对于给定的由n个节点构成，根结点为1的有根树中，我们定义节点u和v是相似的节点，当且仅当节点u的子节点数量与节点v的子节点数量相等。输出相似节点的对数。

#### 输入描述

每个测试文件均包含多组测试数据；

第一行输入一个整数T(1 <= T <= 10^4)，代表数据的组数，每组测试数据描述如下：

第一行输入一个整数n，表示节点数量；

此后n - 1行，第i行输入两个整数ui和vi(1 <= ui, vi <= n)，表示树上第i条边连接ui和vi，保证树联通，没有重边。

除此之外，保证所有n之和不超过2 * (10 ^ 5)。

#### 输出描述

对于每一个测试用例，输出相似节点的对数。

#### 样例输入

> 1
>
> 7
>
> 1 2
>
> 1 3
>
> 3 5
>
> 3 7
>
> 2 4
>
> 2 6

#### 样例输出

> 9

#### 答案

> 根结点的子节点数量为根节点的度数，其他节点的子节点数量为度数 - 1。根据子节点数量分组，然后对每一组计算对数即可。

```java
import java.util.*;

public class Main {
    
    public static void solve() {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();

        // 节点编号从1到n，所以数组大小为 n+1
        int[] degree = new int[n + 1]; 
        for (int i = 0; i < n - 1; i++) {
            int u = scanner.nextInt();
            int v = scanner.nextInt();
            // 每读入一条边，连接的两个节点的度数都加1
            degree[u]++;
            degree[v]++;
        }

        // Key: 子节点的数量, Value: 拥有该数量子节点的节点个数
        Map<Integer, Integer> counts = new HashMap<>();
        
        // 遍历所有节点，计算其子节点数并放入map中计数
        for (int i = 1; i <= n; i++) {
            int childrenNum;
            if (i == 1) { // 根节点
                childrenNum = degree[i];
            } else { // 非根节点 （非根节点都有一个父节点占一个度）
                childrenNum = degree[i] - 1;
            }
            // getOrDefault是一个方便的API，如果key已存在，就返回值；否则返回默认值0
            counts.put(childrenNum, counts.getOrDefault(childrenNum, 0) + 1);
        }

        long totalPairs = 0;
        // 遍历map中的所有value（即每组节点的个数）
        for (int k : counts.values()) {
            // 如果一组中少于2个节点，无法形成对
            if (k >= 2) {
                // 从k个节点中选2个的组合数：C(k, 2)
                totalPairs += (long)k * (k - 1) / 2;
            }
        }
        
        System.out.println(totalPairs);
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int t;
        // 确保在多组输入时不会因为scanner问题出错
        if (scanner.hasNextInt()) {
            t = scanner.nextInt();
        } else {
            return;
        }
        
        while (t-- > 0) {
            solve();
        }
    }
}
```

### 3. 第三题 (太难)

小美和小团在玩一个卡牌游戏，初始时桌面上有n种卡牌，每种卡牌有ai张，这些牌都是背面朝上的。玩家操作时会翻两张牌，如果这两张牌的类型相同，则获胜，否则，将两张牌翻回背面朝上，两个玩家轮流进行操作。小美和小团总共会玩q + 1轮游戏，第1轮的卡牌为初始卡牌，后续每一轮会在上一轮的基础上，增加或减少一些卡牌，然后将所有卡牌背面朝上并重新打乱。增加卡牌的操作为：+ l r x，表示第l种到第r种卡牌各增加x张。减少卡牌的操作为：- l r x，表示第l种到第r种卡牌各减少x张。每一轮都是由小美先手，小美想让小团获胜，小美想知道她至少需要偷看多少张牌才能保证小团一定获胜？若无法保证小团一定获胜，则输出-1。

#### 输入描述

第一行输入2个正整数n,q(1 <= n, q <= 10^5)，表示卡牌的数量和游戏轮数。

第二行输入n个整数，代表初始各种卡牌数量。

接下来q行，每行输入 op l r x，表示操作数据保证任意一轮开始之前，每种卡牌的数量非负，且所有卡牌数量不小于2张。

#### 输出描述

输出q + 1行，每行输出一个整数表示答案。

#### 样例输入

> 2 1
>
> 1 1
>
> \+ 2 2 1

#### 样例输出

> -1
>
> 1

**说明**

第一轮，很显然，谁也赢不了

第二轮，小美可以偷看一张牌，如果这张牌时种类1，那么小美选择这张牌和另外其他一张牌，如果是种类2，则选择另外两张牌。

#### 答案

线段树。如果序列中最大数为1，那么小美和小团谁也赢不了，因此输出-1。如果序列中的最大数等于序列的总和，小美必赢，因此输出-1。如果序列中最大数>1，如果其他数不是0，就是1，那么小美只需要偷看max - 1张牌就行了（max是最大的数，且这样的max只能有一个）。如果序列中最大数>1并且有其他数>1，那么小美至少应该偷看max张牌。

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.util.StringTokenizer;

public class CardGameOptimal {

    static class Node {
        int oddCount;
        int evenCount;
        boolean lazyFlip; // 懒惰标记，true表示需要翻转奇偶

        Node(int odd, int even) {
            this.oddCount = odd;
            this.evenCount = even;
            this.lazyFlip = false;
        }
    }

    static long[] a;
    static Node[] tree;
    static long totalSum = 0;

    public static void main(String[] args) throws IOException {
        FastReader sc = new FastReader();
        PrintWriter out = new PrintWriter(System.out);

        int n = sc.nextInt();
        int q = sc.nextInt();

        a = new long[n + 1];
        tree = new Node[4 * (n + 1)];

        for (int i = 1; i <= n; i++) {
            a[i] = sc.nextLong();
            totalSum += a[i];
        }

        build(1, 1, n);
        solveAndPrint();

        for (int i = 0; i < q; i++) {
            char op = sc.next().charAt(0);
            int l = sc.nextInt();
            int r = sc.nextInt();
            long x = sc.nextLong();

            totalSum += (op == '+') ? (r - l + 1) * x : -(r - l + 1) * x;
            
            // 只有当加/减的数是奇数时，奇偶性才会变化
            if (x % 2 != 0) {
                update(1, 1, n, l, r);
            }
            solveAndPrint();
        }
        out.flush();
    }

    static void solveAndPrint() {
        long U = tree[1].oddCount;
        long totalPairs = (totalSum - U) / 2;

        if (totalPairs == 0 || U == 0) {
            System.out.println(-1);
        } else {
            System.out.println(totalSum - U);
        }
    }

    // 构建线段树
    static void build(int node, int start, int end) {
        if (start == end) {
            if (a[start] % 2 != 0) {
                tree[node] = new Node(1, 0);
            } else {
                tree[node] = new Node(0, 1);
            }
            return;
        }
        int mid = (start + end) / 2;
        build(2 * node, start, mid);
        build(2 * node + 1, mid + 1, end);
        tree[node] = new Node(
            tree[2 * node].oddCount + tree[2 * node + 1].oddCount,
            tree[2 * node].evenCount + tree[2 * node + 1].evenCount
        );
    }

    // 下推懒惰标记
    static void push(int node) {
        if (tree[node].lazyFlip) {
            // 交换子节点的奇偶计数
            Node leftChild = tree[2 * node];
            int tempLeft = leftChild.oddCount;
            leftChild.oddCount = leftChild.evenCount;
            leftChild.evenCount = tempLeft;
            leftChild.lazyFlip = !leftChild.lazyFlip;

            Node rightChild = tree[2 * node + 1];
            int tempRight = rightChild.oddCount;
            rightChild.oddCount = rightChild.evenCount;
            rightChild.evenCount = tempRight;
            rightChild.lazyFlip = !rightChild.lazyFlip;

            // 清除当前节点的标记
            tree[node].lazyFlip = false;
        }
    }

    // 区间更新
    static void update(int node, int start, int end, int l, int r) {
        if (start > r || end < l) {
            return;
        }
        if (start >= l && end <= r) {
            // 完全覆盖，交换奇偶计数并打上懒惰标记
            int temp = tree[node].oddCount;
            tree[node].oddCount = tree[node].evenCount;
            tree[node].evenCount = temp;
            tree[node].lazyFlip = !tree[node].lazyFlip;
            return;
        }

        push(node); // 下推标记

        int mid = (start + end) / 2;
        update(2 * node, start, mid, l, r);
        update(2 * node + 1, mid + 1, end, l, r);

        // 更新父节点信息
        tree[node].oddCount = tree[2 * node].oddCount + tree[2 * node + 1].oddCount;
        tree[node].evenCount = tree[2 * node].evenCount + tree[2 * node + 1].evenCount;
    }
    
    // 快速输入模板
    static class FastReader {
        BufferedReader br;
        StringTokenizer st;

        public FastReader() {
            br = new BufferedReader(new InputStreamReader(System.in));
        }

        String next() {
            while (st == null || !st.hasMoreElements()) {
                try {
                    st = new StringTokenizer(br.readLine());
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            return st.nextToken();
        }

        int nextInt() {
            return Integer.parseInt(next());
        }

        long nextLong() {
            return Long.parseLong(next());
        }
    }
}
```

## 2024.08.31

### 1. 第一题：小美的姓名统计

小美写单词喜欢横着写，她记录了若干个人的名字，但是不小心加进去了一些无关的单词。一个名字单词以大写字母开头，请你帮助她统计共有多少个人的名字。

#### 输入描述

在一行上输入一个长度为n(1<=n<=10^5) 、且由大小写字母和空格混合构成的字符串 s代表小美的全部单词，每个单词之间使用空格间隔。除此之外，保证字符串的开头与结尾字符不为空格。

#### 输出描述

在一行上输出一个整数，代表人名的个数。

#### 样例输入一

> ABC abc Abc

#### 样例输出一

> 2

#### 样例输入二

> A A c

#### 样例输出二

> 2

#### 答案

判断每一个字符串的首字母是否为大写即可。

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        String input = scanner.nextLine();
        
        String[] names = input.split("\\s+");
        int cnt = 0;

        for (String name : names) {
            if (Character.isUpperCase(name.charAt(0))) {
                cnt++;
            }
        }

        System.out.println(cnt);
        scanner.close();
    }
}
```

### 2. 第二题：小美种树（二）

长度无限长的公路上，小美雇佣了 n 位工人来种树，每个点最多种一棵树。从左向右数，工人所站的位置为 a1,a2,...,an 。已知每位工人都会将自己所在位置的右侧一段长度的区间种满树，且每位工人的种树区间长度相同。现在小美希望公路上至少有 k棵树，为了节约成本，他希望每位工人种树的区间长度尽可能短，请你帮他求出，工人们的种树区间至少多长，才能使得公路被种上至少 k棵树。

#### 输入描述

第一行输入两个正整数 n,k(1<=n,k<=2*10^5),分别表示工人的数量，以及小美要求树的最少数量。

第二行输入 n个正整数 a1,a2,...,an(1<=ai<=2*10^5)，表示每名工人的位置。

#### 输出描述

在一行上输出一个整数，代表工人们最短的种树区间长度。

#### 样例输入

> 3 6
>
> 1 2 5

#### 样例输出

> 3

**说明**

每位工人种树的区间长度至少为 3。

这样以来：

第一名工人种：1,2,3 点的树。

第二名工人种：2,3,4 点的树。

第三名工人种：5,6,7点的树。

由于每个位置最多种一棵树，因此共有：1,2,3,4,5,6,7 这些点有树，满足至少 k=6棵树。

可以证明，不存在比 3 更小的答案。

#### 答案

二分答案。我们来二分枚举每个人种树的区间，用一个check函数来判断种树的区间为x的适合，是否可以种k棵树？首先将数组进行排序，对于每一个人可以种的树的逻辑是：尽量往后种，在不超过之前的覆盖的前提下。bd记录的是上一次种到的边界，当前可以种的边界应该是 a+x，只要保证不会超过边界即可。

```java
import java.util.Arrays;
import java.util.Scanner;

public class Main {

    // 判断是否可以在给定的间隔范围内种下至少 k 棵树
    public static boolean check(int[] A, int x, int k) {
        int cnt = 0;
        int bd = 0; // 上一次种到的边界
        for (int a : A) {
            if (a + x > Math.max(a, bd)) {
                cnt += (a + x) - Math.max(a, bd);
            }
            bd = Math.max(a + x, bd);
        }
        return cnt >= k;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int k = scanner.nextInt();
        int[] A = new int[n];

        for (int i = 0; i < n; i++) {
            A[i] = scanner.nextInt();
        }

        Arrays.sort(A);

        int l = 0, r = 300000; // Java 中设置边界

        while (l < r) {
            int mid = (l + r) >> 1;
            if (check(A, mid, k)) {
                r = mid;
            } else {
                l = mid + 1;
            }
        }

        System.out.println(r);
        scanner.close();
    }
}
```

### 3. 第三题：小美和小团的游戏 (太难)

小美和小团在玩一个游戏，游戏中有一个长度为 n 的数组 a ，她们会玩 q 轮游戏，每轮游戏都是独立的。游戏规则如下，双方都会执行最优策略：1） 第一步，游戏给出一个区间 [l,r]。2） 第二步，小团在 [l,r] 区间中选择一个数。3） 第三步，小美将区间扩展成 [L,R] （ [L,R] 必须包含 [l,r] ），然后在 [L,R] 区间中选择一个数，但不能跟小团选同一个数。4） 第四步，小美和小团选择的数字较大的一方获胜，若相同则平局。小美想知道她每一轮的输赢状态，并且她想知道要达到输赢状态所需的 [L,R] 区间长度最小是多少。

#### 输入描述

第一行输入两个整数 n,q(2<=n<=2*10^5) ，表示数组长度和询问次数。

第二行输入 n 个整数 a(1<=ai<=10^9) ，表示数组。

接下来 q 行，每行输入两个整数 (1<=l<=r<=n) ，表示询问。

#### 输出描述

对于每个询问先输出一行，若小美可以获胜则输出 "win" ，若平局则输出 "draw" ，多失败则输出 "lose" 。

第二行输出达到最终状态所需的区间长度的最小值。

#### 样例输入

> 6 2
>
> 1 1 4 5 1 4
>
> 1 3
>
> 4 4

#### 样例输出

> win
>
> 4
>
> lose
>
> 2

**说明**

第1个询问，小团会选择数字4，小美将区间扩展成[1,4]，选择数字5，小美获胜，扩展后的区间长度为4。第2个询问，小团会选择数字5，小美将区间扩展成[3,4]，选择数字4，小团获胜，扩展后的区间长度为2。

#### 答案

ST表+单调栈+二分。使用ST表预处理，便于后续求出[l,r]区间的最大值的下标。对于每一个元素使用单调栈处理，方便求每一个元素下一个更大的元素的下标。如果[l,r]的区间的最大值已经是整个数组的最大值，那么此时的判断就非常清晰：最大值的数量超过2个，那么必然是平局，我们只需要搜索距离区间最近的最大值即可。否则一定是输的，此时需要特判一下，如果区间大小只有1，那么最少需要扩大一个才可以选择。否则，应该使用st表找到当前区间的最大值的下标mx_index，并且使用单调栈找到mx_index的下一个更大的元素/前一个更大的元素，比较哪个更接近即可。

## 2024.08.17

### 1. 第一题

![meituan-08-17-1-1](https://oss.swimmingliu.cn/9fdcada4-8415-11f0-814f-caaeffceb345)

![meituan-08-17-1-2](https://oss.swimmingliu.cn/a0098ec8-8415-11f0-814f-caaeffceb345)

#### 答案

这个正则表达式用于匹配国际电话号码，其中号码部分可以包含数字和井号。

```java
import java.util.Scanner;
import java.util.ArrayList;
import java.util.regex.Pattern;
import java.util.regex.Matcher;

public class StringClassifier {

    // 检查是否为有效的电子邮件地址
    public static boolean isEmail(String str) {
        Pattern pattern = Pattern.compile("^[a-zA-Z0-9_]+@[a-zA-Z0-9_]+\\.com$");
        Matcher matcher = pattern.matcher(str);
        return matcher.matches();
    }

    // 检查是否为有效的IP地址
    public static boolean isIP(String str) {
        Pattern pattern = Pattern.compile("^(\\d{1,2}|1\\d{2}|2[0-4]\\d|25[0-5])\\.(\\d{1,2}|1\\d{2}|2[0-4]\\d|25[0-5])\\.(\\d{1,2}|1\\d{2}|2[0-4]\\d|25[0-5])\\.(\\d{1,2}|1\\d{2}|2[0-4]\\d|25[0-5])$");
        Matcher matcher = pattern.matcher(str);
        return matcher.matches();
    }

    // 检查是否为有效的电话号码
    public static boolean isPhone(String str) {
        Pattern pattern = Pattern.compile("^\\+\\d+-\\d+-[\\d#]+$");
        Matcher matcher = pattern.matcher(str);
        return matcher.matches();
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();  // 读取输入的组数
        scanner.nextLine();  // 忽略整数后的换行符

        ArrayList<String> results = new ArrayList<>();
        while (n-- > 0) {
            String input = scanner.nextLine();

            if (isEmail(input)) {
                results.add("email");
            } else if (isIP(input)) {
                results.add("ip");
            } else if (isPhone(input)) {
                results.add("phone");
            } else {
                results.add("invalid");
            }
        }

        for (String result : results) {
            System.out.println(result);
        }

        scanner.close();
    }
}
```

### 2. 第二题

小美对 gcd (最大公约数) 很感兴趣，她会询问你t次。 每次询问给出一个大于1的正整数n，你是否找到一个数字 m (2 ≤ m ≤ n)，使得 gcd(n,m)为素数。

#### 输入描述

每个测试文件均包含多组测试数据。

第一行输入一个整数 T (1 ≤ T ≤ 100) 代表数据组数，每组测试数据描述如下：

在一行上输入一个整数 n (2 ≤ n ≤ 10^5)代表给定的数字。

#### 输出描述

对于每一组测试数据，在一行上输出一个整数，代表数字m。如果有多种合法答案，您可以输出任意一种。

#### 样例输入

> 2
>
> 114
>
> 15

#### 样例输出

> 57
>
> 5

#### 答案

基本思路就是在[2,n]这个范围内寻找一个数m，满足gcd(n,m)是一个素数。

```java
import java.util.Scanner;
import java.util.ArrayList;

public class GCDPrime {
    // 函数：计算两个数的最大公约数（GCD）
    public static int gcd(int a, int b) {
        while (b != 0) {
            int temp = b;
            b = a % b;
            a = temp;
        }
        return a;
    }

    // 函数：检查一个数是否为素数
    public static boolean isPrime(int num) {
        if (num <= 1) return false;
        if (num <= 3) return true;
        if (num % 2 == 0 || num % 3 == 0) return false;
        for (int i = 5; i * i <= num; i += 6) {
            if (num % i == 0 || num % (i + 2) == 0) return false;
        }
        return true;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int T = scanner.nextInt(); 

        ArrayList<Integer> data = new ArrayList<>();
        for (int i = 0; i < T; i++) {
            data.add(scanner.nextInt()); 
        }

        ArrayList<Integer> result = new ArrayList<>();

        for (int n : data) {
            boolean found = false;
            for (int m = 2; m <= n; ++m) {
                int gcdVal = gcd(n, m); // 计算gcd(n, m)
                if (isPrime(gcdVal)) { // 检查gcd是否为素数
                    result.add(m);
                    found = true;
                    break; 
                }
            }
            if (!found) {
                result.add(-1); 
            }
        }

        for (int res : result) {
            System.out.println(res);
        }

        scanner.close();
    }
}
```

###  3. 第三题 (太难)

小美有一个长度为 n 的数组，每次操作可以选择两个下标i和 j，将 ai 减去 1，将 aj 加上 1。小美想知道最少需要多少次操作，可以使数组极差最小。 数组的极差为数组中最大值和最小值的差。

#### 输入描述

第一行输入一个整数 n (2 ≤  n ≤ 10^5)代表数组的长度。

第二行输入几 个整数 a1,a2,...,an (1 ≤ ai ≤ 10^9) 代表数组的元素。

#### 输出描述

在一行上输出一个整数，表示最少需要多少次操作。

#### 样例输入

> 5
>
> 1 2 3 4 5

#### 样例输出

> 3

**说明**

三次操作分别为 (a1,a5), (a2,a5), (a1,a4)。最终数组为[3,3,3,3,3]，极差为0。

#### 参考题解

通过一系列增减操作来使数组中的最大值和最小值尽可能接近，从而最小化数组的极差。这可以通过计算数组中每个元素与数组平均值之间的差异来完成。 要实现目标，我们应考虑将所有元素调整到一个共同的目标值上，这个目标值最接近整个数组的平均值。由于每次操作是将一个元素减去 1，另一个元素加上 1，这意味着数组的总和在操作前后保持不变。因此，最佳策略是尝试使所有元素达到数组总和除以数组长度的结果（向下取整的结果），即平均值。

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();  // 读取数组长度
        long[] a = new long[n];
        long sum = 0;

        // 读取数组元素并计算总和
        for (int i = 0; i < n; i++) {
            a[i] = scanner.nextLong();
            sum += a[i];
        }

        long avg = sum / n;  // 计算平均值（向下取整）
        long extra = sum % n;  // 计算余数，尽管这个变量在此代码中未使用

        // 计算需要操作的最小次数
        long minMoves = 0;
        long need = 0;
        for (int i = 0; i < n; i++) {
            need += a[i] - avg;  // 更新差值累计
            minMoves = Math.max(minMoves, Math.abs(need));  // 更新最大操作次数
        }

        // 输出最少需要的操作次数
        System.out.println(minMoves);
        scanner.close();
    }
}
```

## 2024.08.10

### 1. 第一题: 小美的密码

小美准备登录美团，需要输入密码，小美忘记了密码，只记得密码可能是 n个字符串中的一个。小美会按照密码的长度从小到大依次尝试每个字符串，对于相同长度的字符串，小美随机尝试，并且相同的密码只会尝试一次。小美想知道，她最少需要尝试多少次才能登录成功，最多需要尝试多少次才能登录成功。小美不会重新尝试已经尝试过的字符串。成功登录后会立即停止尝试。

#### 输入描述

第一行输入一个整数 n(1<=n<=1000)代表密码字符串的个数。

第二行输入一个只由小写字母组成的字符串 s(1<=|s|<=1000)代表正确的密码。

接下来 n 行，每行输入一个长度不超过 1000的字符串，代表小美记得的密码。

#### 输出描述

在一行上输出两个整数，表示最少和最多尝试次数。

#### 样例输入

> 4
>
> ab
>
> abc
>
> ab
>
> ac
>
> ac

#### 样例输出

> 1 2

**说明**

小美可能按照 ["ab", "ac", "abc"] 的顺序尝试，第一次尝试成功，也可能按照 ["ac", "ab", "abc"] 的顺序尝试，第二次尝试成功。

小美在尝试 "ac" 发现不正确后不会继续尝试 "ac"。

#### 答案

哈希表

```java
import java.util.*;

public class Main {

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        
        int n = scanner.nextInt();
        String ans = scanner.next();
        
        Map<Integer, Set<String>> pos = new HashMap<>();
        for (int i = 0; i < n; i++) {
            String p = scanner.next();
            pos.computeIfAbsent(p.length(), k -> new HashSet<>()).add(p);
        }
        
        List<Map.Entry<Integer, Set<String>>> sortedPos = new ArrayList<>(pos.entrySet());
        sortedPos.sort(Map.Entry.comparingByKey());
        
        int step = 0;
        int MIN = -1, MAX = -1;
        for (Map.Entry<Integer, Set<String>> entry : sortedPos) {
            Set<String> v = entry.getValue();
            if (v.contains(ans)) {
                MIN = step + 1;
                MAX = step + v.size();
            } else {
                step += v.size();
            }
        }
        
        System.out.println(MIN + " " + MAX);
        
        scanner.close();
    }
}
```

### 2. 第二题：小美的数组删除

小美有一个长度为 n 的数组 a1,a2,....,an ，他可以对数组进行如下操作：

- 删除第一个元素 a1，同时数组的长度减一，花费为 x。
- 删除整个数组，花费为 k*MEX(a) （其中 MEX(a) 表示 a 中未出现过的最小非负整数。例如 [0,1,2,4] 的 MEX 为 3 ）。

小美想知道将 a 数组全部清空的最小代价是多少，请你帮帮他吧。

### 输入描述

每个测试文件均包含多组测试数据。第一行输入一个整数 T(1<=T<=1000) 代表数据组数，每组测试数据描述如下：

第一行输入三个正整数 n,k,x(1<=n<=2*10^5, 1<=k,x<=10^9) 代表数组中的元素数量、删除整个数组的花费系数、删除单个元素的花费。

第二行输入 n 个整数 a1,a2,....,an(0<=ai<=n)，表示数组元素。

除此之外，保证所有的 n 之和不超过 2*10^5。

### 输出描述

对于每一组测试数据，在一行上输出一个整数表示将数组中所有元素全部删除的最小花费。

### 样例输入

> 1
>
> 6 3 3
>
> 4 5 2 3 1 0

### 样例输出

> 15

**说明：**

若不执行操作一就全部删除，MEX{4,5,2,3,1,0} = 6，花费 18 ；

若执行一次操作一后全部删除，MEX{5,2,3,1,0} = 4，花费 3+12；

若执行两次操作一后全部删除，MEX{2,3,1,0} = 4，花费 6+12 ；

若执行三次操作一后全部删除，MEX{3,1,0} = 2，花费 9+6 ；

若执行四次操作一后全部删除，MEX{1,0} = 2，花费 12+6 ；

若执行五次操作一后全部删除，MEX{0} = 1，花费 15+3；

若执行六次操作一，MEX{} = 0，花费 18；

### 参考题解

动态规划+维护动态最小未出现的整数。f[i]表示从i往后考虑的最小花费，选择就是选择第一个元素或者直接删除后续所有的元素。对于删除后续所有的元素的选项，我们必须要直到MEX是多少，我们可以用在更新dp的过程中，用一个suffix不断地更新当前的最小未出现的整数。虽然这里出现了两层循环的嵌套，但是并不会重置参数，因此复杂度是O(n).

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int T = scanner.nextInt();
        
        while (T-- > 0) {
            long n = scanner.nextLong();
            long k = scanner.nextLong();
            long x = scanner.nextLong();
            
            long[] A = new long[(int) n];
            for (int i = 0; i < n; ++i) {
                A[i] = scanner.nextLong();
            }
            
            long[] dp = new long[(int) (n + 1)];
            Arrays.fill(dp, Long.MAX_VALUE);
            dp[(int) n] = 0;
            
            Set<Long> vst = new HashSet<>();
            long suffix_MEX = 0;
            
            for (int i = (int) (n - 1); i >= 0; --i) {
                vst.add(A[i]);
                while (vst.contains(suffix_MEX)) {
                    suffix_MEX++;
                }
                dp[i] = Math.min(dp[i + 1] + x, k * suffix_MEX);
            }
            
            System.out.println(dp[0]);
        }
        
        scanner.close();
    }
}
```

### 3. 第三题: 小美的彩带 （太难）

小美的彩带是由一条长度为 n 的彩带一直无限循环得到的，彩带的每一个位置都有一个颜色，用 ai 表示。因此当 i>n 时，ai = ai-n 。

小美每次会从左往后或从右往左剪一段长度为 x 的彩带，她想知道她每次剪下来的彩带有多少种颜色。

### 输入描述

第一行输入两个整数 n,q(1<=n,q<=2*10^5) 代表彩带长度、剪彩带次数。

第二行输入 n 个整数 a1,a2,...,an(1<=ai<=10^9) 代表彩带每一个位置的颜色。

此后 q 行，每行输入一个字符 c 和一个整数 x(1<=x<=10^9; c∈L,R) 代表裁剪方向和裁剪长度，其中 'L' 说明从左往右剪， 'R' 说明从右往左剪 。

### 输出描述

对于每一次裁剪彩带，在一行上输出一个整数代表颜色数量。

### 样例输入

> 6 3
>
> 1 1 4 5 1 4
>
> L 2
>
> L 3
>
> R 12

### 样例输出

> 1
>
> 3
>
> 3

**说明：**

第一次剪彩带，剪下来的是 [1,1] ，有 {1} 这 1 种颜色；

第二次剪彩带，剪下来的是 [4,5,1] ，有 {1,4,5} 这 3 种颜色；

第三次剪彩带，剪下来的是 [1,1,4,5,1,4,1,1,4,5,1,4] ，有 {1,4,5} 这 3 种颜色。

### 参考题解

离散化+离线处理（莫队）+树状数组。

```java
import java.util.*;

public class Main {
    private static final int MAXN = 2000007;
    
    private static long[] a = new long[MAXN];
    private static long[] ans = new long[MAXN];
    private static long[] hsh = new long[MAXN];
    private static long[] vis = new long[MAXN];
    private static long[] c = new long[MAXN];

    private static class Query {
        int l, r, id;
        Query(int l, int r, int id) {
            this.l = l;
            this.r = r;
            this.id = id;
        }
    }

    private static int lowbit(int x) {
        return x & -x;
    }

    private static void update(int x, int k) {
        for (int i = x; i < MAXN; i += lowbit(i)) {
            c[i] += k;
        }
    }

    private static long getsum(int x) {
        long sum = 0;
        for (int i = x; i > 0; i -= lowbit(i)) {
            sum += c[i];
        }
        return sum;
    }

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int n = scanner.nextInt();
        int q = scanner.nextInt();
        
        List<Query> queries = new ArrayList<>();

        for (int i = 1; i <= n; i++) {
            a[i] = scanner.nextLong();
            hsh[i] = a[i];
        }
        

        Arrays.sort(hsh, 1, n + 1);
        int len = 1;
        for (int i = 2; i <= n; i++) {
            if (hsh[i] != hsh[len]) {
                hsh[++len] = hsh[i];
            }
        }
        

        for (int i = 1; i <= n; i++) {
            a[i] = Arrays.binarySearch(hsh, 1, len + 1, a[i]) + 1;
            a[n + i] = a[i];
        }
        
        int nl = 1, nr = n * 2;
        for (int i = 1; i <= q; i++) {
            char c = scanner.next().charAt(0);
            int x = scanner.nextInt();
            x = Math.min(x, n);
            if (c == 'L') {
                if (nl > n) {
                    nl -= n;
                }
                queries.add(new Query(nl, nl + x - 1, i));
                nl += x;
            } else {
                if (nr <= n) {
                    nr += n;
                }
                queries.add(new Query(nr - x + 1, nr, i));
                nr -= x;
            }
        }
        

        queries.sort(Comparator.comparingInt(a -> a.r));
        
        int cur = 1;

        for (Query query : queries) {
            for (int i = cur; i <= query.r; i++) {
                if (vis[(int) a[i]] > 0) {
                    update((int) vis[(int) a[i]], -1);
                }
                update(i, 1);
                vis[(int) a[i]] = i;
            }
            cur = query.r + 1;
            ans[query.id] = getsum(query.r) - getsum(query.l - 1);
        }
        

        for (int i = 1; i <= q; i++) {
            System.out.println(ans[i]);
        }
        
        scanner.close();
    }
}
```

