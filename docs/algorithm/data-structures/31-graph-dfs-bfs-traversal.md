---
title: 图结构的 DFS/BFS 遍历
date: 2023-09-01
author: mikez
description: 图结构的 DFS/BFS 遍历
weight: 31
tags:
  - 数据结构的基础
---

_一句话总结_

图的遍历就是 [多叉树遍历](./21-n-ary-tree-traversal.md) 的延伸，主要遍历方式还是深度优先搜索（DFS）和广度优先搜索（BFS）。

唯一的区别是，树结构中不存在环，而图结构中可能存在环，所以我们需要标记遍历过的节点，避免遍历函数在环中死循环。

由于图结构的复杂性，可以细分为遍历图的「节点」、「边」和「路径」三种场景，每种场景的代码实现略有不同。

遍历图的「节点」和「边」时，需要 `visited` 数组在前序位置做标记，避免重复遍历；遍历图的「路径」时，需要 onPath 数组在前序位置标记节点，在后序位置撤销标记。

**图结构看起来虽然比树结构复杂，但图的遍历本质上还是树的遍历。**

## 深度优先搜索（DFS）

前文 [图结构基础和通用实现](30-graph-structure-implementation.md) 中说了，我们一般不用 `Vertex` 这样的类来存储图，但是这里我还是先用一下这个类，以便大家把图的遍历和多叉树的遍历做对比。后面我会给出基于邻接表/邻接矩阵的遍历代码。

### 遍历所有节点（visited 数组）

对比多叉树的遍历框架看图的遍历框架吧：

```go
// Node 多叉树节点
type Node struct {
    val      int
    children []*Node
}

// 多叉树的遍历框架
func traverse(root *Node) {
    // base case
    if root == nil {
        return
    }
    // 前序位置
    fmt.Println("visit", root.val)
    for _, child := range root.children {
        traverse(child)
    }
    // 后序位置
}

// Vertex 图节点
type Vertex struct {
    id        int
    neighbors []*Vertex
}


// 图的遍历框架
// 需要一个 visited 数组记录被遍历过的节点
// 避免走回头路陷入死循环
func traverse(s *Vertex, visited []bool) {
    // base case
    if s == nil {
        return
    }
    if visited[s.id] {
        // 防止死循环
        return
    }
    // 前序位置
    visited[s.id] = true
    fmt.Println("visit", s.id)
    for _, neighbor := range s.neighbors {
        traverse(neighbor, visited)
    }
    // 后序位置
}
```

可以看到，图的遍历比多叉树的遍历多了一个 `visited` 数组，用来记录被遍历过的节点，避免遇到环时陷入死循环。

````md
为什么成环会导致死循环

举个最简单的成环场景，有一条 `1 -> 2` 的边，同时有一条 `2 -> 1` 的边，节点 `1`, `2` 就形成了一个环：

```text
1 <=> 2
```

如果我们不标记遍历过的节点，那么从 `1` 开始遍历，会走到 `2`，再走到 `1`，再走到 `2`，再走到 1，如此 `1->2->1->2->...` 无限递归循环下去。

如果有了 visited 数组，第一次遍历到 `1` 时，会标记`1` 为已访问，出现 `1->2->1` 这种情况时，发现 `1` 已经被访问过，就会直接返回，从而终止递归，避免了死循环。
````

有了上面的铺垫，就可以写出基于邻接表/邻接矩阵的图遍历代码了。虽然邻接表/邻接矩阵的底层存储方式不同，但提供了统一的 API，所以直接使用 [图结构基础和通用实现](./30-graph-structure-implementation.md) 中那个 `Graph` 接口的方法即可：

```go
// 遍历图的所有节点
func traverse(graph Graph, s int, visited []bool) {
    // base case
    if s < 0 || s >= len(graph) {
        return
    }
    if visited[s] {
        // 防止死循环
        return
    }
    // 前序位置
    visited[s] = true
    fmt.Println("visit", s)
    for _, e := range graph.neighbors(s) {
        traverse(graph, e.to, visited)
    }
    // 后序位置
}
```

由于`visited`数组的剪枝作用，这个遍历函数会遍历一次图中的所有节点，并尝试遍历一次所有边，所以算法的时间复杂度是 O(E+V)，其中 E 是边的总数，V 是节点的总数。

```md
时间复杂度为什么是 O(E+V)？

我们之前讲解
二叉树的遍历 时说，二叉树的遍历函数时间复杂度是 O(N)，其中 N 是节点的总数。

这里图结构既然是树结构的延伸，为什么图的遍历函数时间复杂度是 O(E+V)，要把边的数量 E 也算进去呢？为什么不是 O(V) 呢？

其实二叉树/多叉树的遍历函数，也要算上边的数量，只不过对于树结构来说，边的数量和节点的数量是近似相等的，所以时间复杂度还是 O(N+N)=O(N)。

树结构中的边只能由父节点指向子节点，所以除了根节点，你可以把每个节点和它上面那条来自父节点的边配成一对儿，这样就可以比较直观地看出边的数量和节点的数量是近似相等的。

而对于图结构来说，任意两个节点之间都可以连接一条边，边的数量和节点的数量不再有特定的关系，所以我们要说图的遍历函数时间复杂度是 O(E+V)。
```

### 遍历所有边（二维 visited 数组）

对于图结构，遍历所有边的场景并不多见，主要是
计算欧拉路径 时会用到，所以这里简单提一下。

上面遍历所有节点的代码用一个一维的 `visited` 数组记录已经访问过的节点，确保每个节点只被遍历一次；那么最简单直接的实现思路就是用一个二维的 visited 数组来记录遍历过的边（`visited[u][v]` 表示边 `u->v` 已经被遍历过），从而确保每条边只被遍历一次。

先参考多叉树的遍历进行对比：

```go
// 从起点 s 开始遍历图的所有边
func traverseEdges(graph Graph, s int, visited [][]bool) {
    // base case
    if s < 0 || s >= len(graph) {
        return
    }
    for _, e := range graph.neighbors(s) {
        // 如果边已经被遍历过，则跳过
        if visited[s][e.to] {
            continue
        }
        // 标记并访问边
        visited[s][e.to] = true
        fmt.Printf("visit edge: %d -> %d\n", s, e.to)
        traverseEdges(graph, e.to, visited)
    }
}
```

```md
由于一条边由两个节点构成，所以我们需要把前序位置的相关代码放到 for 循环内部。
```

接下来，我们可以用 [图结构基础和通用实现](30-graph-structure-implementation.md) 中的 Graph 接口来实现：

```go
// 从起点 s 开始遍历图的所有边
func traverseEdges(graph Graph, s int, visited [][]bool) {
    // base case
    if s < 0 || s >= len(graph) {
        return
    }
    for _, e := range graph.neighbors(s) {
        // 如果边已经被遍历过，则跳过
        if visited[s][e.to] {
            continue
        }
        // 标记并访问边
        visited[s][e.to] = true
        fmt.Printf("visit edge: %d -> %d\n", s, e.to)
        traverseEdges(graph, e.to, visited)
    }
}
```

显然，使用二维 visited 数组并不是一个很高效的实现方式，因为需要创建二维 visited 数组，这个算法的时间复杂度是 O(E+V²)，空间复杂度是 O(V²)，其中 E 是边的数量，V 是节点的数量。

在讲解 Hierholzer 算法计算欧拉路径 时，我们会介绍一种简单的优化避免使用二维 `visited` 数组，这里暂不展开。

### 遍历所有路径（onPath 数组）

对于树结构，遍历所有「路径」和遍历所有「节点」是没什么区别的。而对于图结构，遍历所有「路径」和遍历所有「节点」稍有不同。

因为对于树结构来说，只能由父节点指向子节点，所以从根节点 `root` 出发，到任意一个节点 `targetNode` 的路径都是唯一的。换句话说，我遍历一遍树结构的所有节点之后，必然可以找到 `root` 到 `targetNode` 的唯一路径：

```go
// 多叉树的遍历框架，寻找从根节点到目标节点的路径
var path []*Node

func traverse(root *Node, targetNode *Node) {
	// base case
	if root == nil {
		return
	}
	if root.val == targetNode.val {
		// 找到目标节点
		fmt.Print("find path: ")
		for _, node := range path {
			fmt.Printf("%d->", node.val)
		}
		fmt.Printf("%d\n", targetNode.val)
		return
	}
	// 前序位置
	path = append(path, root)
	for _, child := range root.children {
		traverse(child, targetNode)
	}
	// 后序位置
	path = path[:len(path)-1]
}
```

而对于图结构来说，由起点 `src` 到目标节点 `dest` 的路径可能不止一条。我们需要一个 `onPath` 数组，在进入节点时（前序位置）标记为正在访问，退出节点时（后序位置）撤销标记，这样才能遍历图中的所有路径，从而找到 `src` 到 `dest` 的所有路径：

```go
// 下面的算法代码可以遍历图的所有路径，寻找从 src 到 dest 的所有路径

// onPath 和 path 记录当前递归路径上的节点
var onPath []bool = make([]bool, graph.size())
var path []int

func traverse(graph Graph, src int, dest int) {
    // base case
    if src < 0 || src >= graph.size() {
        return
    }
    if onPath[src] {
        // 防止死循环（成环）
        return
    }
    if src == dest {
        // 找到目标节点
        fmt.Print("find path: ")
        for _, node := range path {
            fmt.Printf("%d->", node)
        }
        fmt.Printf("%d\n", dest)
        return
    }

    // 前序位置
    onPath[src] = true
    path = append(path, src)
    for _, e := range graph.neighbors(src) {
        traverse(graph, e.to, dest)
    }
    // 后序位置
    path = path[:len(path)-1]
    onPath[src] = false
}
```

```md
关键区别在于后序位置撤销标记

为啥之前讲的遍历节点就不用撤销 `visited` 数组的标记，而这里要在后序位置撤销 onPath 数组的标记呢？

因为前文遍历节点的代码中，`visited` 数组的职责是保证每个节点只会被访问一次。而对于图结构来说，要想遍历所有路径，可能会多次访问同一个节点，这是关键的区别。

比方说下面这幅图，我们想求从节点 `1` 到节点 `4` 的全部路径：

![graph-path](/images/algorithm/graph-path.jpg)

如果使用前文遍历节点的算法，只在前序位置标记 `visited` 为 `true`，那么遍历完 `1->2->4` 和 `1->2->3->4` 之后，所有节点都已经被标记为已访问了，算法就会停止，visited 数组完成了自己的职责。

但是显然我们还没有遍历完所有路径，`1->3->2->4` 和 `1->3->4` 被漏掉了。

如果用 `onPath` 数组在离开节点时（后序位置）撤销标记，就可以找到 `1` 到 `4` 的所有路径。
```

由于这里使用的 `onPath` 数组会在后序位置撤销标记，所以这个函数可能重复遍历图中的节点和边，复杂度一般较高（阶乘或指数级），具体的时间复杂应该是所有路径的长度之和，取决于图的结构特点。

**同时使用 visited 和 onPath 数组**
按照上面的分析，`visited` 数组和 `onPath` 分别用于遍历所有节点和遍历所有路径。那么它们两个是否可能会同时出现呢？答案是可能的。

遍历所有路径的算法复杂度较高，大部分情况下我们可能并不需要穷举完所有路径，而是仅需要找到某一条符合条件的路径。这种场景下，我们可能会借助 visited 数组进行剪枝，提前排除一些不符合条件的路径，从而降低复杂度。

比如后文
拓扑排序 中会讲到如何判定图是否成环，就会同时利用 `visited` 和 `onPath` 数组来进行剪枝。

比方说判定成环的场景，在遍历所有路径的过程中，如果发现一个节点 `s` 被标记为 `visited`，那么说明从 `s` 这个起点出发的所有路径在之前都已经遍历过了。如果之前遍历的时候都没有找到环，我现在再去遍历一次，肯定也不会找到环，所以这里可以直接剪枝，不再继续遍历节点 `s`。

我会在后面的图论算法和习题中结合具体的案例讲解，这里就不展开了。

完全不用 `visited` 和 `onPath` 数组
是否有既不用 `visited` 数组，也不用 `onPath` 数组的场景呢？其实也是有的。

前面介绍了，`visited` 和 `onPath` 主要的作用就是处理成环的情况，避免死循环。那如果题目告诉你输入的图结构不包含环，那么你就不需要考虑成环的情况了。

```md
797. 所有可能的路径 | [力扣](https://leetcode.cn/problems/all-paths-from-source-to-target)
```

这个题目输入的是一个 [邻接表](./30-graph-structure-implementation.md)，且明确告诉你输入的图结构不包含环，所以不需要 visited 和 onPath 数组，直接使用 DFS 遍历图就行了：

```go
func allPathsSourceTarget(graph [][]int) [][]int {
    // 记录所有路径
    var res [][]int
    var path []int
    traverse(graph, 0, &path, &res)
    return res
}

// 图的遍历框架
func traverse(graph [][]int, s int, path *[]int, res *[][]int) {
    // 添加节点 s 到路径
    *path = append(*path, s)

    n := len(graph)
    if s == n-1 {
        // 到达终点
        *res = append(*res, append([]int(nil), *path...))
        *path = (*path)[:len(*path)-1]
        return
    }

    // 递归每个相邻节点
    for _, v := range graph[s] {
        traverse(graph, v, path, res)
    }

    // 从路径移出节点 s
    *path = (*path)[:len(*path)-1]
}
```

## 广度优先搜索（BFS）

图结构的广度优先搜索其实就是 [多叉树的层序遍历](./21-n-ary-tree-traversal.md)，无非就是加了一个 visited 数组来避免重复遍历节点。

理论上 BFS 遍历也需要区分遍历所有「节点」和遍历所有「路径」，但是实际上 BFS 算法一般只用来寻找那条**最短路径**，不会用来求所有路径。

当然 BFS 算法肯定也可以求所有路径，但是我们一般会选择用 DFS 算法求所有路径，具体原因我在 [二叉树的递归/层序遍历](./19-binary-tree-traversal.md) 中讲过，这里就不展开了。

那么如果只求最短路径的话，只需要遍历「节点」就可以了，因为按照 BFS 算法一层一层向四周扩散的逻辑，第一次遇到目标节点，必然就是最短路径。

和前文[多叉树的层序遍历](./21-n-ary-tree-traversal.md)介绍的一样，图结构的 BFS 算法框架也有三种不同的写法，下面我会对比着多叉树的层序遍历写一下图结构的三种 BFS 算法框架。

### 写法一

第一种写法是不记录遍历步数的。

多叉树的层序遍历写法是这样：

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

图结构的 BFS 遍历是类似的：

```go
// 图结构的 BFS 遍历，从节点 s 开始进行 BFS
func bfs(graph *Graph, s int) {
    visited := make([]bool, graph.size())
    q := list.New()
    q.PushBack(s)
    visited[s] = true

    for q.Len() > 0 {
        e := q.Front()
        cur := e.Value.(int)
        q.Remove(e)
        fmt.Println("visit", cur)
        for _, edge := range graph.neighbors(cur) {
            if visited[edge.to] {
                continue
            }
            q.PushBack(edge.to)
            visited[edge.to] = true
        }
    }
}
```

### 写法二

第二种能够记录遍历步数的写法。

多叉树的层序遍历写法是这样：

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

图结构的 BFS 遍历是类似的：

```go
// 从 s 开始 BFS 遍历图的所有节点，且记录遍历的步数
func bfs(graph Graph, s int) {
    visited := make([]bool, graph.size())
    q := []int{s}
    visited[s] = true
    // 记录从 s 开始走到当前节点的步数
    step := 0
    for len(q) > 0 {
        sz := len(q)
        for i := 0; i < sz; i++ {
            cur := q[0]
            q = q[1:]
            fmt.Printf("visit %d at step %d\n", cur, step)
            for _, e := range graph.neighbors(cur) {
                if visited[e.to] {
                    continue
                }
                q = append(q, e.to)
                visited[e.to] = true
            }
        }
        step++
    }
}
```

### 写法三

第三种能够适配不同权重边的写法。

多叉树的层序遍历写法是这样：

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

图结构的 BFS 遍历是类似的：

```go
// 图结构的 BFS 遍历，从节点 s 开始进行 BFS，且记录遍历步数（从起点 s 到当前节点的边的条数）
// 每个节点自行维护 State 类，记录从 s 走来的遍历步数
type State struct {
    // 当前节点 ID
    node   int
    // 从起点 s 到当前节点的遍历步数
    step int
}

func bfs(graph Graph, s int) {
    visited := make([]bool, graph.size())
    q := []*State{{s, 0}}
    visited[s] = true
    for len(q) > 0 {
        state := q[0]
        q = q[1:]
        cur := state.node
        step := state.step
        fmt.Printf("visit %d with step %d\n", cur, step)
        for _, e := range graph.neighbors(cur) {
            if visited[e.to] {
                continue
            }
            q = append(q, &State{e.to, step + 1})
            visited[e.to] = true
        }
    }
}
```

上面的代码示例中，`State.step` 变量记录了从起点 s 到当前节点的最少步数（边的条数）。

但是对于加权图来说，每条边可以有不同的权重，题目一般会让我们计算从 src 到 dest 的权重和最小的路径。在这个场景中，step 的值（边的条数）失去了意义，这个算法也无法完成任务。

我们会在之后的 图结构最短路径算法 中详细介绍如何计算加权图的最小权重路径，那时候你就会发现，只需要对这个 BFS 写法稍作修改就能得到 Dijkstra 算法，完成加权图的最短路径计算。
