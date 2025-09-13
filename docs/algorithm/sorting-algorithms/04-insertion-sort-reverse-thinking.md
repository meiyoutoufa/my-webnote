---
title: 运用逆向思维：插入排序
date: 2023-09-01
author: mikez
description: 运用逆向思维：插入排序
weight: 4
tags:
  - 十大排序原理
---

_一句话总结_

插入排序是基于 [选择排序](02-selection-sort-problems.md) 的一种优化，将 `nums[sortedIndex]` 插入到左侧的有序数组中。对于有序度较高的数组，插入排序的效率比较高。

前文 [选择排序](02-selection-sort-problems.md)所面临的问题 中分析了选择排序遇到的几个问题，然后逐步优化写出了 [冒泡排序](./03-stable-bubble-sort.md)，使得排序算法具有稳定性，且能够在输入数组的有序度较高时提前终止，提升效率。

回顾一下，冒泡排序的关键点在于对下面这段代码的优化：

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

为了避免 while 内存在两个 for 循环，我们使用了一种类似冒泡的方式逐步交换 `nums[sortedIndex..]` 中的逆序对，将最小值换到 `nums[sortedIndex]` 的位置。

好的，先停在这一步，让我们忘记冒泡排序的优化方法，你来思考一下，是否还有其他方法能够优化上述代码，把 while 循环中的两个 for 循环优化成一个 for 循环？

## 反向思维

上面的算法思路是：在 `nums[sortedIndex..]` 中找到最小值，然后将其插入到 `nums[sortedIndex]` 的位置。

那么我们能不能反过来想，在 `nums[0..sortedIndex-1]` 这个部分有序的数组中，找到 `nums[sortedIndex]` 应该插入的位置，然后进行插入呢？

当年我思考如何对插入排序进行优化时，是想到过这个思路的，因为我想利用数组的有序性呀：既然 `nums[0..sortedIndex-1]` 这部分是已经排好序的，那么我就可以用二分搜索来寻找 nums[sortedIndex] 应该插入的位置。

这样一来，上述代码中的内层第一个 for 循环，我可以给他优化成对数级别的复杂度。

但是仔细想想，用二分搜索好像是多此一举的。因为就算我用二分搜索找到了 `nums[sortedIndex]` 应该插入的位置，我还是需要搬移元素进行插入，那还不如一边遍历一遍交换元素的方法简单高效呢：

```go
// 对选择排序进一步优化，向左侧有序数组中插入元素
// 这个算法有另一个名字，叫做插入排序
func sort(nums []int) {
    n := len(nums)
    // 维护 [0, sortedIndex) 是有序数组
    sortedIndex := 0
    for sortedIndex < n {
        // 将 nums[sortedIndex] 插入到有序数组 [0, sortedIndex) 中
        for i := sortedIndex; i > 0; i-- {
            if nums[i] < nums[i-1] {
                // swap(nums[i], nums[i - 1])
                tmp := nums[i]
                nums[i] = nums[i-1]
                nums[i-1] = tmp
            } else {
                break
            }
        }
        sortedIndex++
    }
}
```

```md
插入排序

这个算法的名字叫做插入排序，它的执行过程就像是打扑克牌时，将新抓到的牌插入到手中已经排好序的牌中。

插入排序的空间复杂度是 O(1)，是原地排序算法。时间复杂度是 O(n²)，具体的操作次数和选择排序类似，是一个等差数列求和，大约是 n²/2 次。

插入排序是一种稳定排序，因为只有在 `nums[i] < nums[i - 1]` 的情况下才会交换元素，所以相同元素的相对位置不会发生改变
```

## 初始有序度越高，效率越高

显然，插入排序的效率和输入数组的有序度有很大关系，可以举极端例子来理解：

如果输入数组已经有序，或者仅有个别元素逆序，那么插入排序的内层 for 循环几乎不需要执行元素交换，所以时间复杂度接近 O(n)。

如果输入的数组是完全逆序的，那么插入排序的效率就会很低，内层 for 循环每次都要对 nums[0..sortedIndex-1] 的所有元素进行交换，算法的总时间复杂度就接近 O(n²)。

如果对比插入排序和冒泡排序，**插入排序的综合性能应该要高于冒泡排序**。

直观地说，插入排序的内层 for 循环，只需要对 sortedIndex 左侧 nums[0..sortedIndex-1] 这部分有序数组进行遍历和元素交换，大部分非极端情况下，可能不需要遍历完 nums[0..sortedIndex-1] 的所有元素；而冒泡排序的内层 for 循环，每次都需要遍历 sortedIndex 右侧 nums[sortedIndex..] 的所有元素。

所以冒泡排序的操作数大约是 n²/2，而插入排序的操作数会小于 n²/2。

你可以把插入排序的代码拿去力扣第 912 题「排序数组」提交，它最终依然会超时，但可以说明算法代码的逻辑是正确的。之后的文章我们继续探讨如何对排序算法进行优化。
