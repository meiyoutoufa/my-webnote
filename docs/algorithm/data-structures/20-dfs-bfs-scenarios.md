---
title: DFS 和 BFS 的适用场景
date: 2023-09-01
author: mikez
description: DFS 和 BFS 的适用场景
weight: 20
tags:
  - 数据结构的基础
---

在实际的算法问题中，DFS 算法常用来穷举所有路径，BFS 算法常用来寻找最短路径，这是什么原因呢？

因为二叉树的递归遍历和层序遍历就是最简单的 DFS 算法和 BFS 算法，所以本文就用一道简单的二叉树例题，说明其中的道理。

为什么 BFS 常用来寻找最短路径
用可视化面板结合一道例题，你立刻就能明白了。

来看力扣第 111 题[二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree)：

二叉树的最小深度即「根节点到最近的叶子节点的距离」，所以这道题本质上就是让你求最短距离。

DFS 递归遍历和 BFS 层序遍历都可以解决这道题，先看 DFS 递归遍历的解法：

```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }

    // 记录最小深度（根节点到最近的叶子节点的距离）
    minDepthValue := math.MaxInt32
    // 记录当前遍历到的节点深度
    currentDepth := 0

    var traverse func(*TreeNode)
    traverse = func(root *TreeNode) {
        if root == nil {
            return
        }

        // 前序位置进入节点时增加当前深度
        currentDepth++

        // 如果当前节点是叶子节点，更新最小深度
        if root.Left == nil && root.Right == nil {
            minDepthValue = min(minDepthValue, currentDepth)
        }

        traverse(root.Left)
        traverse(root.Right)

        // 后序位置离开节点时减少当前深度
        currentDepth--
    }

    // 从根节点开始 DFS 遍历
    traverse(root)
    return minDepthValue
}
```

每当遍历到一条树枝的叶子节点，就会更新最小深度，**当遍历完整棵树后**，就能算出整棵树的最小深度。

你能不能在不遍历完整棵树的情况下，提前结束算法？不可以，因为你必须确切的知道每条树枝的深度（根节点到叶子节点的距离），才能找到最小的那个。

下面来看 BFS 层序遍历的解法。按照 BFS 从上到下逐层遍历二叉树的特点，当遍历到第一个叶子节点时，就能得到最小深度：

```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    q := []*TreeNode{root}
    // root 本身就是一层，depth 初始化为 1
    depth := 1

    for len(q) > 0 {
        sz := len(q)
        // 遍历当前层的节点
        for i := 0; i < sz; i++ {
            cur := q[0]
            q = q[1:]
            // 判断是否到达叶子结点
            if cur.Left == nil && cur.Right == nil {
                return depth
            }
            // 将下一层节点加入队列
            if cur.Left != nil {
                q = append(q, cur.Left)
            }
            if cur.Right != nil {
                q = append(q, cur.Right)
            }
        }
        // 这里增加步数
        depth++
    }
    return depth
}
```

当它遍历到第二行的时候，就遇到第一个叶子节点了，这个叶子节点就是距离根节点最近的叶子节点，所以此时算法就结束了。BFS 算法并没有遍历整棵树就找到了最小深度。

综上，你应该能理解为啥 BFS 算法经常用来寻找最短路径了：

**由于 BFS 逐层遍历的逻辑，第一次遇到目标节点时，所经过的路径就是最短路径，算法可能并不需要遍历完所有节点就能提前结束**。

DFS 遍历当然也可以用来寻找最短路径，但必须遍历完所有节点才能得到最短路径。

从时间复杂度的角度来看，两种算法在最坏情况下都会遍历所有节点，时间复杂度都是 O(N)，但在一般情况下，显然 BFS 算法的实际效率会更高。所以在寻找最短路径的问题中，BFS 算法是首选。

### 为什么 DFS 常用来寻找所有路径

在寻找所有路径的问题中，你会发现 DFS 算法用的比较多，BFS 算法似乎用的不多。

理论上两种遍历算法都是可以的，只不过 BFS 算法寻找所有路径的代码比较复杂，DFS 算法代码比较简洁。

你想啊，就以二叉树为例，如果要用 BFS 算法来寻找所有路径（根节点到每个叶子节点都是一条路径），队列里面就不能只放节点了，而需要使用
[二叉树层序遍历](./19-二叉树的递归和层序遍历.md)的第三种写法，新建一个 `State` 类，把当前节点以及到达当前节点的路径都存进去，这样才能正确维护每个节点的路径，最终计算出所有路径。

而使用 DFS 算法就简单了，它本就是一条树枝一条树枝从左往右遍历的，每条树枝就是一条路径，递归遍历到叶子节点的时候递归路径就是一条树枝，所以 DFS 算法天然适合寻找所有路径。

综上，DFS 算法在寻找所有路径的问题中更常用，而 BFS 算法在寻找最短路径的问题中更常用。
