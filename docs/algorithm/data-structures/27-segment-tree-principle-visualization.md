---
title: 线段树核心原理及可视化
date: 2023-09-01
author: mikez
description: 线段树核心原理及可视化
weight: 27
tags:
  - 数据结构的基础
---

_一句话总结_

线段树是[二叉树结构](18-binary-tree-basics-types.md)的衍生，用于高效解决数组的区间查询和区间动态修改问题。

线段树可以在 O(logN) 的时间复杂度查询任意长度的区间元素聚合值，在 O(logN) 的时间复杂度对任意长度的区间元素进行动态修改，其中 N 为数组中的元素个数。

### 使用场景

在[选择排序]中，我们会尝试解决一个需求，就是计算 `nums` 数组中从索引 `i` 开始到末尾的最小值。

我们将提出一种使用 `suffixMin` 数组的优化尝试，即提前预计算一个 `suffixMin` 数组，使得 `suffixMin[i] = min(nums[i..])`，这样就可以在 O(1) 时间内查询 `nums[i..]` 的最小值：

```java
int[] nums = new int[]{3, 1, 4, 2};
// suffixMin[i] 表示 nums[i..] 中的最小值
int[] suffixMin = new int[nums.length];

// 从后往前计算 suffixMin
suffixMin[nums.length - 1] = nums[nums.length - 1];
for (int i = nums.length - 2; i >= 0; i--) {
    suffixMin[i] = Math.min(nums[i], suffixMin[i + 1]);
}

// [1, 1, 2, 2]
System.out.println(suffixMin);

// 有了 suffixMin 数组，可以在 O(1) 时间内查询任意 nums[0..i] 后缀的最小值

// 查询 nums[1..] 的最小值
System.out.println(suffixMin[1]); // 1
```

其实这种预计算的思路是非常常见的，不仅是求最小值，求和、求最大值、求乘积等场景，都可以通过预计算数组来优化查询效率。

后面的章节你就会学到，这种技巧可以统一归为前缀和技巧，主要用于处理区间查询问题。

但这个技巧有它的局限性，即 `nums` 数组本身不能变化。一旦 nums[i] 变化了，那么 `suffixMin[0..i]` 的值都会失效，需要 O(N) 的时间复杂度重新计算 `suffixMin` 数组。

**对于这种希望对整个区间进行查询，同时支持动态修改元素的场景，是线段树结构的应用场景**。

### 线段树的核心 API

线段树结构可以有多种变体及复杂的优化，我们这里先聚焦最核心的两个 API：

```java
class SegmentTree {
    // 构造函数，给定一个数组，初始化线段树，时间复杂度 O(N)
    // merge 是一个函数，用于定义 query 方法的行为
    // 通过修改这个函数，可以让 query 函数返回区间的元素和、最大值、最小值等
    public SegmentTree(int[] nums, Function<Integer, Integer> merge) {}

    // 查询闭区间 [i, j] 的元素和（也可能是最大最小值，取决于 merge 函数），时间复杂度 O(logN)
    public int query(int i, int j) {}

    // 更新 nums[i] = val，时间复杂度 O(logN)
    public void update(int i, int val) {}
}
```

类比之前说的 `suffixMin` 数组，假设 `num`s 数组的元素个数为 N：

- `suffixMin[i]` 可以在 O(1) 时间内查询 `nums[i..]` 后缀的最小值；线段树的 `query` 方法不仅可以查询后缀，还可以查询任意 `[i, j]` 区间，时间复杂度均为 O(logN)。
- 当底层 `nums` 数组中的任意元素变化时，需要重新计算 `suffixMin` 数组，时间复杂度为 O(N)；而线段树的 `update` 方法可以在 O(logN) 时间内完成元素的修改。

线段树不仅仅支持计算区间的最小值，只要修改 merge 函数，就可以支持计算区间元素和、最大值、乘积等。

### 线段树的核心原理

##### 树高为什么是 O(logN)

在代码实现部分你会看到，最简单的线段树构造方法，是递归地将 nums 数组从正中间切分，然后递归地将左右子数组构造成线段树，直到区间长度为 1。

由于每次递归构造线段树都是从正中间切分的，左右子树的元素个数基本相同，所以整棵二叉树是平衡的，即树高是 O(logN)，其中 N 是 nums 数组的元素个数。

#### query 为什么是 O(logN)

树的高度为 O(logN)，`query` 方法的本质是从根节点开始遍历这棵二叉树，需要遍历的节点个数是是 logN 的常数倍，所以 `query` 方法的时间复杂度是 O(logN)。

如果恰好有一个非叶子节点记录着区间 `[i, j]` 的元素和，那么就可以直接返回；如果没有，`[i, j]` 区间会被分裂成两个子区间，递归查询左右子区间的元素和，然后合并结果。

综上，query 方法遍历的节点总数是 logN 的常数倍，所以时间复杂度是 O(logN)。

**update 为什么是 O(logN)**
数组的元素就是叶子节点，`update` 方法的本质是修改叶子节点的值，顺带会更新从根节点到该叶子节点的路径上的所有非叶子节点的值。因为树高为 O(logN)，所以 `update` 方法的时间复杂度也是 O(logN)。
