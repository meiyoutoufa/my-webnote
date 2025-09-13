---
title: 妙用二叉树后序位置：归并排序
date: 2023-09-01
author: mikez
description: 妙用二叉树后序位置：归并排序
weight: 8
tags:
  - 十大排序原理
---

_一句话总结_

堆排序是从
二叉堆结构 衍生出来的排序算法，复杂度为 O(NlogN)。堆排序主要分两步，第一步是在待排序数组上原地创建二叉堆（Heapify），然后进行原地排序（Sort）。

前文 [二叉堆基础](./../data-structures/25-binary-heap-principle-visualization.md) 介绍过二叉堆结构，
二叉堆实现优先级队列 利用二叉堆结构实现了一个 SimpleMinPQ 优先级队列，插入队列的元素会按照从小到大的顺序取出。

本文将介绍堆排序算法，它是基于二叉堆性质衍生出来的一种全新排序算法，非常优雅和高效。

首先，我要复述一下二叉堆实现优先级队列的几个关键原理，如果你有任何不理解的地方，务必回去复习前文，否则无法理解堆排序。

1、二叉堆（优先级队列）底层是用数组实现的，但是逻辑上是一棵完全二叉树，主要依靠 swim, sink 方法来维护堆的性质。

2、优先级队列可以分为小顶堆和大顶堆，小顶堆会将整个堆中最小的元素维护在堆顶，大顶堆会将整个堆中最大的元素维护在堆顶。

3、优先级队列插入元素时，首先把元素追加到二叉堆底部，然后调用 swim 方法把该元素上浮到合适的位置，时间复杂度是 O(logN)。

4、优先级队列删除堆顶元素时，首先把堆底的最后一个元素交换到堆顶作为新的堆顶元素，然后调用 sink 方法把这个新的堆顶元素下沉到合适的位置，时间复杂度是 O(logN)。

那么最简单的堆排序算法思路就是直接利用优先级队列，把所有元素塞到优先级队列里面，然后再取出来，不就完成排序了吗？

```go
// 直接利用优先级队列对数组从小到大排序
func sort(nums []int) {
    // 创建一个从小到大排序元素的小顶堆
    pq := NewSimpleMinPQ(len(nums))

    // 先把所有元素插入到优先级队列中
    for _, num := range nums {
        // push 操作会自动构建二叉堆，时间复杂度为 O(logN)
        pq.Push(num)
    }

    // 再把所有元素取出来，就是从小到大排序的结果
    for i := 0; i < len(nums); i++ {
        // pop 操作从堆顶弹出二叉堆堆中最小的元素，时间复杂度为 O(logN)
        nums[i] = pq.Pop()
    }
}
```

因为优先级队列的 push, pop 方法的复杂度都是 O(logN)，所以整个排序的时间复杂度是 O(NlogN)，其中 N 是输入数组的长度。

这个思路可以得到正确的排序结果，但空间复杂度是 O(N)，因为我们创建的这个优先级队列是一个额外的数据结构，它的底层使用了一个数组来存储元素。

所以，堆排序要解决的问题是，**不要使用额外的辅助空间，直接在原数组上进行** `sink`, `swim` 操作，在 O(NlogN) 的时间内完成排序。

```md
堆排序的两个关键步骤

1、原地建堆（Heapify）：直接把待排序数组原地变成一个二叉堆。

2、排序（Sort）：将元素不断地从堆中取出，最终得到有序的结果。
```

你不妨自己思考几分钟，对比一下优先级队列增删元素的过程，其实利用 swim, sink 方法原地实现这两步并不难，应该可以独立思考出来。

在具体讲解堆排序代码实现之前，我先把二叉堆的 swim, sink 方法和配套的工具函数写出来，因为后文我会带你逐步优化堆排序的代码，就不重复实现这些函数了。

这些函数就是从 [二叉堆实现优先级队列](../data-structures/26-binary-heap-priority-queue.md) 中的优先级队列实现里抠出来的，把数组作为函数参数传入，其他的逻辑完全一样：

```go
// 小顶堆的上浮操作，时间复杂度是树高 O(logN)
func minHeapSwim(heap []int, node int) {
    for node > 0 && heap[parent(node)] > heap[node] {
        swap(heap, parent(node), node)
        node = parent(node)
    }
}

// 小顶堆的下沉操作，时间复杂度是树高 O(logN)
func minHeapSink(heap []int, node, size int) {
    for left(node) < size || right(node) < size {
        // 比较自己和左右子节点，看看谁最小
        min := node
        if left(node) < size && heap[left(node)] < heap[min] {
            min = left(node)
        }
        if right(node) < size && heap[right(node)] < heap[min] {
            min = right(node)
        }
        if min == node {
            break
        }
        // 如果左右子节点中有比自己小的，就交换
        swap(heap, node, min)
        node = min
    }
}

// 大顶堆的上浮操作
func maxHeapSwim(heap []int, node int) {
    for node > 0 && heap[parent(node)] < heap[node] {
        swap(heap, parent(node), node)
        node = parent(node)
    }
}

// 大顶堆的下沉操作
func maxHeapSink(heap []int, node, size int) {
    for left(node) < size || right(node) < size {
        // 小顶堆和大顶堆的唯一区别就在这里，比较逻辑相反
        // 比较自己和左右子节点，看看谁最大
        max := node
        if left(node) < size && heap[left(node)] > heap[max] {
            max = left(node)
        }
        if right(node) < size && heap[right(node)] > heap[max] {
            max = right(node)
        }
        if max == node {
            break
        }
        swap(heap, node, max)
        node = max
    }
}

// 父节点的索引
func parent(node int) int {
    return (node - 1) / 2
}

// 左子节点的索引
func left(node int) int {
    return node*2 + 1
}

// 右子节点的索引
func right(node int) int {
    return node*2 + 2
}

// 交换数组中两个元素的位置
func swap(heap []int, i, j int) {
    heap[i], heap[j] = heap[j], heap[i]
}
```

## 简单直接的堆排序实现

只要你彻底理解了优先级队列的 push, pop 操作，那么就能比较容易得想到下面这个原地堆排序的实现：

```go
// 将输入的数组元素从小到大排序
func sort(nums []int) {
    // 第一步，原地建堆，注意这里创建的是大顶堆
    // 只要从左往右对每个元素调用 swim 方法，就可以原地建堆
    for i := 0; i < len(nums); i++ {
        maxHeapSwim(nums, i)
    }

    // 第二步，排序
    // 现在整个数组已经是一个大顶了，直接模拟删除堆顶元素的过程即可
    heapSize := len(nums)
    for heapSize > 0 {
        // 从堆顶删除元素，放到堆的后面
        swap(nums, 0, heapSize-1)
        heapSize--
        // 恢复堆的性质
        maxHeapSink(nums, 0, heapSize)
        // 现在 nums[0..heapSize) 是一个大顶堆，nums[heapSize..) 是有序元素
    }
}
```

这里面一个关键点是要用大顶堆来完成 nums 从小到大的排序，因为从堆顶删除的元素是从后往前填到 nums 数组中的，最终 nums 中的元素是从小到大排序的。

如果你用小顶堆的话，最终 nums 中的元素是从大到小排序的，还需要再翻转一下数组，没有大顶堆的效率高。

````md
类比优先级队列的操作
这个实现思路相当于完全模拟了优先级队列的 push, pop 操作，类似下面的逻辑：

```java
void sort(int[] nums) {
    // 第一步，创建大顶堆
    SimpleMaxPQ pq = new SimpleMaxPQ();
    for (int num : nums) {
        pq.push(num);
    }

    // 第二步，排序
    int heapSize = pq.size();
    while (heapSize > 0) {
        // 因为这是大顶堆，所以 pop 出来的元素是从大到小的
        nums[heapSize - 1] = pq.pop();
        heapSize--;
    }
    // 最终 nums 数组就是从小到大排序的结果
}
```

如果还是不理解，建议回去复习一下优先级队列的实现。
````

我们来分析一下上述代码的时间复杂度，假设 nums 的元素个数为 N：

第一步建堆的过程中，swim 方法的时间复杂度是 O(logN)，算法对每个元素调用一次 swim 方法，所以总时间复杂度是 O(NlogN)。
第二步排序的过程中，每次 sink 方法的时间复杂度是 O(logN)，算法对每个元素调用一次 sink 方法，所以总时间复杂度是 O(NlogN)。
综上，整个堆排序的时间复杂度是 2NlogN，用 Big O 表示就是 O(NlogN)。与
快速排序、
归并排序 是一个级别的排序算法。

上述算法直接操作原始数组，没有使用额外的辅助空间，所以空间复杂度是 O(1)。

你可以把这段代码拿去力扣第 912 题「排序数组」提交，它可以轻松通过所有测试用例。

## 利用逆向思维进行优化

能写出上面的堆排序代码已经不错了，但是这个解法其实还可以再优化，优化的点在于建堆的过程。

说实话这个优化点不太容易想到，我直接讲了，请你仔细思考理解。

**首先，请你忘记二叉堆底层的数组实现，我们只看二叉堆的逻辑结构，即完全二叉树**。

在我们的解法中，对每个元素依次调用 maxHeapSwim 方法建堆，相当于把数组中的第一个元素作为堆顶（完全二叉树的根节点），然后不断把新元素追加到堆底（完全二叉树的最底层最右侧），调用 maxHeapSwim 方法让新元素上浮到合适的位置。

那么在这个过程中，其实你是没有任何优化空间的，因为每个新元素都必须加到堆里，且必须进行上浮操作，才能维护堆的性质。

想要找到优化空间，需要利用二叉堆的一个性质：**对于一个二叉堆来说，其左右子堆（子树）也是一个二叉堆**。

就好比一棵二叉树的根节点，它的左右子树也是一棵二叉树。

如果你给我一个二叉树节点和两棵二叉树，那么我可以把这个二叉树节点作为根节点，两棵二叉树作为根节点的左右子树，这样就能构建出一棵新的二叉树。

二叉堆本质上是二叉树，所以也是类似的：

如果给我两个二叉堆，和一个二叉堆节点，那么我可以把这个节点作为堆顶（根节点），两个二叉堆作为左右子堆（子树），构建出一棵新的二叉堆（二叉树），对吧？

但是有个问题，构建出的这个新二叉堆，它的左右子堆肯定都符合堆的性质，但这个新的根节点的值可能不符合堆的性质，怎么办？

简单啊，`sink` 方法不就是专门针对这种情况的吗？只要对根节点调用一次 `sink` 方法，就能让这个新的二叉堆符合堆的性质了。

这就是堆排序算法的优化思路的核心，先看代码，注意建堆的部分：

```go
// 将输入的数组元素从小到大排序
func sort(nums []int) {
    // 第一步，原地建堆，注意这里创建的是大顶堆
    // 从最后一个非叶子节点开始，依次下沉，合并二叉堆
    n := len(nums)
    for i := n / 2 - 1; i >= 0; i-- {
        maxHeapSink(nums, i, n)
    }

    // 合并完成，现在整个数组已经是一个大顶堆

    // 第二步，排序，和刚才的代码一样
    heapSize := n
    for heapSize > 0 {
        // 从堆顶删除元素，放到堆的后面
        swap(nums, 0, heapSize-1)
        heapSize--
        // 恢复堆的性质
        maxHeapSink(nums, 0, heapSize)
        // 现在 nums[0..heapSize) 是一个大顶堆，nums[heapSize..) 是有序元素
    }
}
```

每个单独的叶子节点都是符合堆的性质的，所以上述代码从最后一个非叶子节点开始，依次调用 maxHeapSink 方法，合并所有的子堆，最终整个数组就是一个大顶堆了。

`n / 2 - 1` 是最后一个非叶子节点的索引，这个值的运算也比较精妙，需要你熟练掌握
完全二叉树的性质，我计划专门开一个章节把各种二叉树的性质及推导进行总结，所以这里先不展开，你有兴趣的话可以先自己思考一下。

显然，这个优化后的堆排序算法在建堆时效率更高，因为只需要对一半的元素调用 `sink` 方法，总的操作次数大概是 1/2∗NlogN。虽然用 Big O 表示法还是 O(NlogN)，但实际执行的操作次数肯定会比对每个元素调用 `swim` 方法要少。

二叉堆结构，用数组抽象二叉树，简单的 `swim`, `sink` 方法，都能玩出这么多花活，算法还是挺有意思的吧？

## 堆排序的稳定性

堆排序是一种不稳定的排序算法，因为二叉堆本质上是把数组结构抽象成了二叉树结构，在二叉树逻辑结构上的元素交换操作映射回数组上，无法顾及相同元素的相对位置。
