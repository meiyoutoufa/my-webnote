---
title: 多叉树的递归/层序遍历
date: 2023-09-01
author: mikez
description: 多叉树的递归/层序遍历
weight: 21
tags:
  - 数据结构的基础
---

_一句话总结_

多叉树结构就是[二叉树结构](18-二叉树基础及常见类型.md)的延伸，二叉树是特殊的多叉树。森林是指多个多叉树的集合。

多叉树的遍历就是[二叉树遍历](19-二叉树的递归和层序遍历.md)的延伸。

二叉树的节点长这样，每个节点有两个子节点：

```go
type TreeNode struct {
    Val   int
    Left  *TreeNode
    Right *TreeNode
}
```

多叉树的节点长这样，每个节点有任意个子节点：

```go
type Node struct {
    val      int
    children []*Node
}
```

就这点区别，其他没了。

## 森林

这里介绍一下「森林」这个名词，后面讲到 [UnionFind] 并查集算法 时，会用到这个概念。

顾名思义，**森林就是多个多叉树的集合（一棵多叉树是一个特殊的森林）**，用代码表示就是多个多叉树的根节点列表，类似这样：

```java
List<TreeNode> forest;
```

只需对每个根节点分别进行 DFS/BFS 遍历，即可遍历森林的所有节点。

在并查集算法中，我们会同时持有多棵多叉树的根节点，那么这些根节点的集合就是一个森林。

接下来说下多叉树的遍历，和二叉树一样，也就递归遍历（DFS）和层序遍历（BFS）两种。

### 递归遍历（DFS）

对比二叉树的遍历框架看多叉树的遍历框架吧：

```go
// 二叉树的遍历框架
func traverse(root *TreeNode) {
    if root == nil {
        return
    }
    // 前序位置
    traverse(root.Left)
    // 中序位置
    traverse(root.Right)
    // 后序位置
}

// N 叉树的遍历框架
func traverseNary(root *Node) {
    if root == nil {
        return
    }
    // 前序位置
    for _, child := range root.Children {
        traverseNary(child)
    }
    // 后序位置
}
```

唯一的区别是，多叉树没有了中序位置，因为可能有多个节点嘛，所谓的中序位置也就没什么意义了。

### 层序遍历（BFS）

多叉树的层序遍历和[二叉树的层序遍历](19-二叉树的递归和层序遍历.md)一样，都是用队列来实现，无非就是把二叉树的左右子节点换成了多叉树的所有子节点。所以多叉树的层序遍历也有三种写法，下面一一列举。

#### 写法一

第一种层序遍历写法，无法记录节点深度：

```go
func levelOrderTraverse(root *Node) {
	if root == nil {
		return
	}
	q := []*Node{}
	q = append(q, root)
	for len(q) > 0 {
		cur := q[0]
		q = q[1:]
		// 访问 cur 节点
		fmt.Println(cur.Val)

		// 把 cur 的所有子节点加入队列
		for _, child := range cur.Children {
			q = append(q, child)
		}
	}
}
```

#### 写法二

第二种层序遍历写法，能够记录节点深度：

```go
func levelOrderTraverse(root *Node) {
    if root == nil {
        return
    }
    q := []*Node{root}
    // 记录当前遍历到的层数（根节点视为第 1 层）
    depth := 1

    for len(q) > 0 {
        sz := len(q)
        for i := 0; i < sz; i++ {
            cur := q[0]
            q = q[1:]
            // 访问 cur 节点，同时知道它所在的层数
            fmt.Printf("depth = %d, val = %d\n", depth, cur.val)

            for _, child := range cur.children {
                q = append(q, child)
            }
        }
        depth++
    }
}
```

#### 写法三

第三种能够适配不同权重边的写法：

```go
// 多叉树的层序遍历
// 每个节点自行维护 State 类，记录深度等信息
type State struct {
    node  *Node
    depth int
}

func levelOrderTraverse(root *Node) {
    if root == nil {
        return
    }
    q := []State{}
    // 记录当前遍历到的层数（根节点视为第 1 层）
    q = append(q, State{root, 1})

    for len(q) > 0 {
        state := q[0]
        q = q[1:]
        cur := state.node
        depth := state.depth
        // 访问 cur 节点，同时知道它所在的层数
        fmt.Printf("depth = %d, val = %d\n", depth, cur.Val)

        for _, child := range cur.Children {
            q = append(q, State{child, depth + 1})
        }
    }
}
```

没啥好说的，有不明白的可以对照前文[二叉树遍历](19-二叉树的递归和层序遍历.md)的层序遍历代码和可视化面板。
