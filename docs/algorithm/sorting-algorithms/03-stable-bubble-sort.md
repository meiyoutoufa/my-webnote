---
title: 拥有稳定性：冒泡排序
date: 2023-09-01
author: mikez
description: 拥有稳定性：冒泡排序
weight: 3
tags:
  - 十大排序原理
---

_一句话总结_

冒泡算法是对 [选择排序](02-selection-sort-problems.md) 的一种优化，通过交换 nums[sortedIndex] 右侧的逆序对完成排序，是一种稳定排序算法。

前文讲解了 [选择排序](02-selection-sort-problems.md) 这种最简单直接的排序算法，其中分析了选择排序的几个待优化的问题：

1、选择排序算法是个不稳定排序算法，因为每次都要交换最小元素和当前元素的位置，这样可能会改变相同元素的相对位置。

2、选择排序的时间复杂度和初始数据的有序度完全没有关系，即便输入的是一个已经有序的数组，选择排序的时间复杂度依然是 O(n²)。

3、选择排序的时间复杂度是 O(n²)，具体的操作次数大概是 n²/2 次，常规的优化思路无法降低时间复杂度。

那么本文就围绕着选择排序的种种缺陷，看看能不能想办法帮它解决一下。

## 重获排序稳定性

前文分析过选择排序失去稳定性的原因，即每次都要交换最小元素（`nums[minIndex]`）和当前元素（`nums[sortedIndex]`），这样可能会改变相同元素的相对位置。

你仔细思考这个交换过程，其实它的目标是把 `nums[minIndex]` 放到到 `nums[sortedIndex]`，至于 `nums[sortedIndex]` 这个位置的元素应该去哪里，它并不关心。之所以它用交换操作，只是因为交换操作最简单，不需要涉及数据搬移。

在交换过程中，把 `nums[minIndex]` 放到到 `nums[sortedIndex]` 的操作是不影响相同元素的相对顺序的：

```text
[2, 2', 2'', 1, 1']
 ^           ^
[1, 2', 2'', _, 1']
 ^           ^
sortedIndex  minIndex
```

真正破坏稳定性的，是让 `nums[sortedIndex]` 去 `nums[minIndex]` 的位置这一步：

```text
[1, 2', 2'', 2, 1']
 ^           ^
```

可以看到 `2, 2', 2''` 这三个元素的相对顺序被打乱了。

所以优化的方向就在这里，你不要图省事儿直接把 `nums[sortedIndex]` 交换到 `nums[minIndex]`，而是模仿 在数组中部插入元素的操作，将 `nums[sortedIndex..minIndex]` 的元素整体向后移动一位，把 `nums[sortedIndex + 1]` 的位置空出来让 `nums[sortedIndex]` 这个元素去那里待着。

```text
[2, 2', 2'', 1, 1']
 ^           ^
[1, 2', 2'', _, 1']
 ^           ^
[1, _, 2', 2'', 1']
 ^           ^
[1, 2, 2', 2'', 1']
 ^           ^
sortedIndex  minIndex
```

可以看到，这次 `2, 2', 2''` 和 `1, 1'` 的相对顺序都没有发生改变，选择排序就变成了稳定排序了。

具体代码如下，只需要把 选择排序 代码中交换元素的部分换一下即可：

```go
// 对选择排序进行第一波优化，获得了稳定性
func sort(nums []int) {
    n := len(nums)
    sortedIndex := 0
    for sortedIndex < n {
        // 在未排序部分中找到最小值 nums[minIndex]
        minIndex := sortedIndex
        for i := sortedIndex + 1; i < n; i++ {
            if nums[i] < nums[minIndex] {
                minIndex = i
            }
        }

        // 优化：将 nums[minIndex] 插入到 nums[sortedIndex] 的位置
        // 将 nums[sortedIndex..minIndex] 的元素整体向后移动一位
        minVal := nums[minIndex]
        // 数组搬移数据的操作
        for i := minIndex; i > sortedIndex; i-- {
            nums[i] = nums[i - 1]
        }
        nums[sortedIndex] = minVal

        sortedIndex++
    }
}
```

你可以拿着这个算法去力扣第 912 题[排序数组](https://leetcode.cn/problems/sort-an-array)提交一下，虽然最后会超时无法通过，但是可以证明这个算法的正确性是没有问题的。

这个算法对比标准的选择排序，虽然拥有了稳定性，但是执行效率会下降，虽然从 Big O 表示法的角度来看，两层嵌套循环的时间复杂度还是 O(n²)，但毕竟又加了一个 for 循环，实际执行次数肯定会大于标准选择排序的 n²/2 次。

下面我们再来看看，能不能进一步优化，避免这个额外的 for 循环。

## 优化时间复杂度

仔细观察上面的算法代码，while 循环内部主要做了两件事：

1、第一个 for 循环寻找 `nums[sortedIndex..]` 中的最小值。

2、第二个 for 循环将这个最小值插入到 `nums[sortedIndex]` 的位置。

那么我们能否将这两个步骤合在一起呢？具体来说，你在寻找 `nums[sortedIndex..]` 中的最小值的时候能不能做些力所能及的事情，能不能做到找到最小值后，它就已经被放在正确的位置上，不需要再进行数据搬移了？

答案是可以的，看我操作：

```go
// 对选择排序进行第二波优化，获得稳定性的同时避免额外的 for 循环
// 这个算法有另一个名字，叫做冒泡排序
func sort(nums []int) {
    n := len(nums)
    sortedIndex := 0
    for sortedIndex < n {
        // 寻找 nums[sortedIndex..] 中的最小值
        // 同时将这个最小值逐步移动到 nums[sortedIndex] 的位置
        for i := n - 1; i > sortedIndex; i-- {
            if nums[i] < nums[i-1] {
                // swap(nums[i], nums[i - 1])
                tmp := nums[i]
                nums[i] = nums[i-1]
                nums[i-1] = tmp
            }
        }
        sortedIndex++
    }
}
```

这个优化就比较巧妙了，倒序遍历 nums[sortedIndex..]，如果发现逆序对儿，就交换顺序，这样最小值就会逐步移动到 nums[sortedIndex] 的位置。

而且由于我们只交换相邻的逆序对儿，不会去碰值相同的元素，所以这个算法是稳定排序。

这个算法的时间复杂度依然是 O(n²)，实际执行次数和选择排序类似，也是一个等差数列求和，大约是 n²/2 次。

```md
冒泡排序

这个算法的名字叫做冒泡排序，因为它的执行过程就像从数组尾部向头部冒出水泡，每次都会将最小值顶到正确的位置。
```

## 提前终止算法

上面说到选择排序的一个问题是，其时间复杂度和初始数据的有序度完全没有关系，即便输入的数组已经有序，选择排序依然会执行 O(n²) 次操作。

在上面的一些列优化之后，就可以解决这个问题了，具体看代码：

```go
// 进一步优化，数组有序时提前终止算法

func sort(nums []int) {
    n := len(nums)
    sortedIndex := 0
    for sortedIndex < n {
        // 加一个布尔变量，记录是否进行过交换操作
        swapped := false
        for i := n - 1; i > sortedIndex; i-- {
            if nums[i] < nums[i-1] {
                // swap(nums[i], nums[i - 1])
                tmp := nums[i]
                nums[i] = nums[i-1]
                nums[i-1] = tmp
                swapped = true
            }
        }
        // 如果一次交换操作都没有进行，说明数组已经有序，可以提前终止算法
        if !swapped {
            break
        }
        sortedIndex++
    }
}
```

好了，以上就是针对选择排序的一系列优化，最终使它拥有了排序稳定性，并支持在数组有序时提前终止算法。唯一的遗憾是，时间复杂度依然是 O(n²)，并没有降低。

下面我们继续探讨，看看还有什么方法能够改进选择排序。
