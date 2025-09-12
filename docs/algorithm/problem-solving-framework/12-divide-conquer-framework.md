---
title: 分治算法解题套路框架
date: 2023-09-01
author: mikez
description: 分治算法解题套路框架
weight: 12
tags:
  - 核心刷题框架
---

本文主要介绍分治算法的核心原理和解题技巧。

**分治算法**和**分治思想**这两个概念不太一样，下面通过简单的示例来解释。

## 分治思想

广义的分治思想是一个宽泛的概念，本站教程中也经常称之为「**分解问题的思路**」。

分治思想就是把一个问题分解成若干个子问题，然后分别解决这些子问题，最后合并子问题的解得到原问题的解，这种思想广泛存在于递归算法中。

比如斐波那契数列的递归解法，把原问题 `fib(n)` 分解成 `fib(n-1)` 和 `fib(n-2)` 两个子问题，根据子问题的解合并得到原问题的解，这就是分解问题的思路呀：

```java
int fib(int n) {
    // base case
    if (n == 0 || n == 1) {
        return n;
    }
    return fib(n - 1) + fib(n - 2);
}
```

普通的二叉树算法，比如让你计算一棵二叉树总共有多少个节点：

```java
// 定义：输入一棵二叉树的根节点，返回这棵树的节点总数
int count(TreeNode root) {
    // base case
    if (root == null) {
        return 0;
    }
    // 先算出左右子树的节点个数
    int leftCount = count(root.left);
    int rightCount = count(root.right);

    // 左右子树的节点个数加上自己，就是整棵树的节点个数
    return leftCount + rightCount + 1;
}
```

这种解法也是分解问题的思路：你把整棵树的节点个数（原问题）分解成了左子树的节点个数和右子树的节点个数（子问题），然后递归计算左右子树的节点个数，最后根据左右子树的节点个数得到整棵树的节点个数。

再比方说
动态规划算法 属不属于分治思想？

也属于，因为所有动态规划算法都是把大问题分解成了结构相同规模更小的子问题，通过子问题的最优解合并得到原问题的最优解，只不过这个过程中有一些特殊的优化操作罢了。

还可以举出很多例子。其实我在
二叉树心法（纲领篇） 中已经总结过了，**递归算法只有两种思路，一种是遍历的思路，另一种是分解问题的思路（分治思想）。**

遍历思路的典型代表就是
回溯算法，那么除了回溯算法之外，其他递归算法都可以归为分解问题的思路（分治思想）。

以此观之，「分治思想」占据了递归算法的半壁江山，那么当我们说「分治算法」的时候，具体是指什么呢？是不是可以说上面列举的这些递归算法都是「分治算法」呢？其实不是的。

## 分治算法

狭义的分治算法也是运用分治思想的递归算法，但它有一个特征，是上面列举的算法所不具备的：

把问题分解后进行求解，相比于不分解直接求解，时间复杂度更低。

符合这个特征的算法，我们才称之为「分治算法」。

上面列举的算法，它们本身就只能分解求解，不存在「直接求解」的解法，所以只说它们运用了分治思想，不说它们是分治算法。

比如
桶排序算法，桶排序的思路是把待排序数组分成若干个桶，然后对每个桶分别进行插入排序，最后把所有有序桶合并，这样时间复杂度能降到 O(n)。

直接用 插入排序 的复杂度是 O(n²)，而分解后再用插入排序，总的时间复杂度就能降到 O(n)，这种才算分治算法。

**那么这里面是什么道理，为什么分而治之的复杂度更低呢？如果把所有问题都分而治之，是不是都能得到更低的复杂度呢？**

下面就来详细地对比探究一下，什么情况下分治思想能降低复杂度，什么时候不可以，以及其中的原理所在。

### 无效的分治

理论上讲，很多算法都可以用分解问题的思路改写成递归算法，但大部分情况下这种改写是无意义的。

看一个非常简单的例子，求一个数组的和，这个算法的时间复杂度是 O(n)，空间复杂度是 O(1)：

```java
int getSum(int[] nums) {
    int sum = 0;
    for (int i = 0; i < nums.length; i++) {
        sum += nums[i];
    }
    return sum;
}
```

我完全可以用分解问题的思路把这个问题改写成递归算法：

```java
// 定义：返回 nums[start..] 的元素和
int getSum2(int[] nums, int start) {
    // base case
    if (start == nums.length) {
        return 0;
    }
    // nums[start..] 的元素和可以分解成第一个元素和剩余元素的和
    return nums[start] + getSum2(nums, start + 1);
}
```

你问我所有元素的和，我就把这个问题分解成第一个元素和剩余元素的和，这就是分治思想呀。

关于这个算法的可视化，注意两个重点：

1、递归树的形态类似一条链表，高度为 O(n)，这是因为每次递归调用都是 `start + 1`，所以递归树退化成了链表。

2、注意 `nums` 数组中元素染色的顺序，因为递归结束的 base case 是数组的最后一个元素，所以元素和是从后往前累加的。

如果不考虑子数组复制所产生的复杂度，这个算法的时间复杂度是 O(n)，空间复杂度是 O(n)。

递归调用需要 O(n) 的堆栈空间，所以空间复杂度是 O(n)；时间复杂度等于递归调用的次数 x 每次递归调用的时间复杂度，递归调用的次数是 n，每次递归调用只做一次加法操作，时间复杂度是 O(1)，所以总的时间复杂度是 O(n)。

可以看到，这个算法的时间复杂度并没有比迭代算法更低，而且还多了 O(n) 的空间复杂度。

有读者可能说，如果从中间二分，效率会不会高一些？

我们来试试，把数组分成两半，分别求和，最后把两半的和相加：

```java
int getSum3(int[] nums, int start, int end) {
    // base case
    if (start == end) {
        return nums[start];
    }

    int mid = start + (end - start) / 2;
    // 计算 nums[start..mid] 的和
    int leftSum = getSum3(nums, start, mid);
    // 计算 nums[mid+1..end] 的和
    int rightSum = getSum3(nums, mid + 1, end);

    // 合并得到 nums[start..end] 的和
    return leftSum + rightSum;
}
```

对比 getSum3 和 getSum2 的可视化，有以下几个关键区别：

1、getSum2 算法的递归树退化成了链表，堆栈（树高）的空间复杂度是 O(n)；而这个 getSum3 算法从中间二分，递归树就是一个较为平衡的二叉树，所以堆栈（树高）的空间复杂度是 O(logn)。

2、注意 nums 中元素染色的顺序，getSum2 是从后往前染色，这个 getSum3 是从前往后染色。因为 getSum3 算法中叶子节点是数组元素，二叉树遍历叶子节点的顺序是从左到右的。

时间复杂度还是 O(n)，因为递归调用的次数（二叉树节点数）是 O(n)，每次递归调用只做几次加减法，时间复杂度是 O(1)，所以总的时间复杂度是 O(n)。

综上，这两种分治算法改写都属于无效的分治，没有降低时间复杂度，反而由于递归而增加了空间复杂度。

这也是预期之内的事情，数组元素求和，时间复杂度在怎么优化都不可能低于 O(n)，因为你至少得遍历一遍所有元素对吧，这么遍历一次就要 O(n) 的时间了，怎么可能优化呢？

那既然这样，为啥还要写这么多来讲解这么简单的一个问题呢？因为我主要想给你论证以下要点：

1、**分治的思想是广泛存在的**，几乎所有算法都可以改写成递归分治的形式。

2、**分治思想不等于高效**。不要听到 XX 算法就觉得高大上，很多时候，改写成分治解法并不能带来什么实际的好处，甚至可能增加空间复杂度，因为递归调用需要堆栈空间。

3、**用二分的方式进行分治可以将递归树的深度从 O(n) 降低到 O(logn)，确实有优化效果**。对于上面这个元素求和的例子，无论怎么分治都不如原解法高效，但可以看出二分的分治方式是确实有助于减少递归树的高度。

我对这个简单的算法还专门配了可视化面板，就是希望你注意 nums 数组中元素染色的顺序，你会发现即便改写成了递归分治算法，本质上和 for 循环是一样的效果，只不过改成了递归形式，遍历的顺序不同而已。

下面，就来看看什么情况下分治思想能带来实际的好处。

### 有效的分治

这里重新探讨 单链表双指针技巧汇总 中的一个问题，[合并 k 个有序链表](https://leetcode.cn/problems/merge-k-sorted-lists/)：

在 单链表双指针技巧汇总 中，我介绍的解法是利用
优先级队列 这种数据结构对链表节点进行动态排序，这种解法的时间复杂度是 O(Nlogk)，空间复杂度是 O(k)，其中 k 代表链表的条数，N 代表 k 条链表节点的总数，

在本文中，我们不再依赖额外的数据结构，而是直接用分治算法解决这个问题，时间复杂度依然是 O(Nlogk)。

首先，我们要解决[合并两个有序链表](https://leetcode.cn/problems/merge-two-sorted-lists/)的问题，也就是力扣第 21 题：

这道题也是 单链表双指针技巧汇总 中的例题，标准的双指针解法，这里直接贴出解法代码，就不多讲了：

```go
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
    // 虚拟头结点
    dummy := &ListNode{-1, nil}
    p := dummy
    p1 := l1
    p2 := l2

    for p1 != nil && p2 != nil {
        // 比较 p1 和 p2 两个指针
        // 将值较小的的节点接到 p 指针
        if p1.Val > p2.Val {
            p.Next = p2
            p2 = p2.Next
        } else {
            p.Next = p1
            p1 = p1.Next
        }
        // p 指针不断前进
        p = p.Next
    }

    if p1 != nil {
        p.Next = p1
    }

    if p2 != nil {
        p.Next = p2
    }

    return dummy.Next
}
```

这个算法使用两个指针，把两个链表都遍历了一遍，所以时间复杂度是 O(l1+l2)，l1 和 l2 分别是两个链表的长度。

下面我们来思考如何合并 k 个有序链表，先想一个暴力解吧，运用上面的这个 `mergeTwoLists` 函数把 k 个链表两两合并，都合并到第一个链表上：

```java
ListNode mergeKLists(ListNode[] lists) {
    if (lists.length == 0) {
        return null;
    }
    // 把 k 个有序链表都合并到 lists[0] 上
    ListNode l0 = lists[0];
    for (int i = 1; i < lists.length; i++) {
        l0 = mergeTwoLists(l0, lists[i]);
    }
    return l0;
}

ListNode mergeTwoLists(ListNode l1, ListNode l2) {
    // 见上文
}
```

这样肯定是能得到正确答案的，我尝试去力扣上提交 Java 代码，可以通过，但是速度非常慢，这是什么原因呢？

![hexinkuangjia68](/images/algorithm/hexinkuangjia68.jpg)

```java
// 定义：合并 lists[start..] 为一个有序链表
ListNode mergeKLists2(ListNode[] lists, int start) {
    if (start == lists.length - 1) {
        return lists[start];
    }
    // 合并 lists[start + 1..] 为一个有序链表
    ListNode subProblem = mergeKLists2(lists, start + 1);

    // 合并 lists[start] 和 subProblem，就得到了 lists[start..] 的有序链表
    return mergeTwoLists(lists[start], subProblem);
}
```

结合可视化面板可以更好地理解。请你点开下面的可视化，其中输入了 k=4 条链表，递归树的形态类似一个单链表，高度为 O(k)，把鼠标移动到递归树的每个节点上，会显示每次递归需要合并的链表。

不难发现重复的次数取决于树高，上面这个算法的递归树很不平衡，导致递归树退化成链表，树高变为 O(k)。

如果能让递归树尽可能地平衡，就能减小树高，进而减少链表的重复遍历次数，提高算法的效率。

如何让递归树平衡呢？就类似上面 `getSum3` 函数的思路，把链表从中间分成两部分，分别递归合并为有两个序链表，最后再将这两部分合并成一个有序链表。

请看完整的解法代码：

```go
// 用分治算法合并 k 个有序链表
func mergeKLists(lists []*ListNode) *ListNode {
    if len(lists) == 0 {
        return nil
    }
    return mergeKLists3(lists, 0, len(lists)-1)
}

// 定义：合并 lists[start..end] 为一个有序链表
func mergeKLists3(lists []*ListNode, start, end int) *ListNode {
    if start == end {
        return lists[start]
    }
    mid := start + (end-start)/2
    // 合并左半边 lists[start..mid] 为一个有序链表
    left := mergeKLists3(lists, start, mid)
    // 合并右半边 lists[mid+1..end] 为一个有序链表
    right := mergeKLists3(lists, mid+1, end)
    // 合并左右两个有序链表
    return mergeTwoLists(left, right)
}

// 双指针技巧合并两个有序链表
// https://labuladong.online/algo/essential-technique/linked-list-skills-summary/
func mergeTwoLists(l1, l2 *ListNode) *ListNode {
    dummy := &ListNode{Val: -1}
    p := dummy
    p1, p2 := l1, l2

    for p1 != nil && p2 != nil {
        if p1.Val > p2.Val {
            p.Next = p2
            p2 = p2.Next
        } else {
            p.Next = p1
            p1 = p1.Next
        }
        p = p.Next
    }

    if p1 != nil {
        p.Next = p1
    }

    if p2 != nil {
        p.Next = p2
    }

    return dummy.Next
}
```

```md
时空复杂度分析

该算法的时间复杂度相当于是把 k 条链表分别遍历 O(logk) 次。

那么假设 k 条链表的元素总数是 N，该算法的时间复杂度就是 O(Nlogk)，和
单链表双指针技巧汇总 中介绍的优先级队列解法相同。

再来看空间复杂度，该算法的空间复杂度只有递归树堆栈的开销，也就是 O(logk)，要优于优先级队列解法的 O(k)。
```

## 总结

本文主要介绍了分治算法的核心原理和解题技巧。

分治思想在递归算法中是广泛存在的，甚至一些非递归算法，都可以强行改写成分治递归的形式，但并不是所有算法都能用分治思想提升效率。

那为什么有些算法可以通过分治思想来优化时间复杂度呢？

**把递归算法抽象成递归树，如果递归树节点的时间复杂度和树的深度相关，那么使用分治思想对问题进行二分，就可以使递归树尽可能平衡，进而优化总的时间复杂度。**

反之，如果递归树节点的时间复杂度和树的深度无关，那么使用分治思想就没有意义，反而可能引入额外的空间复杂度。

本文的两个例子中，`getSum` 函数即便改为递归形式，每个递归节点做的事情无非就是一些加减运算，所以递归节点的时间复杂度总是 O(1)，和树的深度无关，所以分治思想不起作用。

而 `mergeKLists` 函数中，每个递归节点都需要合并两个链表，这两个链表是子节点返回的，其长度和递归树的高度相关，所以使用分治思想可以优化时间复杂度。

你看，说了半天，本质上又回到二叉树的遍历了。所以我说了一万遍，二叉树非常非常重要，把二叉树玩明白，算法简直不要太简单。
