---
title: 双指针技巧秒杀七道数组题目
date: 2023-09-01
author: mikez
description: 双指针技巧秒杀七道数组题目
weight: 3
tags:
  - 核心刷题框架
---

在处理数组和链表相关问题时，双指针技巧是经常用到的，双指针技巧主要分为两类：**左右指针**和**快慢指针**。

所谓左右指针，就是两个指针相向而行或者相背而行；而所谓快慢指针，就是两个指针同向而行，一快一慢。

对于单链表来说，大部分技巧都属于快慢指针，
单链表的六大解题套路 都涵盖了，比如链表环判断，倒数第 K 个链表节点等问题，它们都是通过一个 fast 快指针和一个 slow 慢指针配合完成任务。

在数组中并没有真正意义上的指针，但我们可以把索引当做数组中的指针，这样也可以在数组中施展双指针技巧，本文主要讲数组相关的双指针算法。

## 一、快慢指针技巧

### 原地修改

**数组问题中比较常见的快慢指针技巧，是让你原地修改数组。**

比如说看下力扣第 26 题[「删除有序数组中的重复项」](https://leetcode.cn/problems/remove-duplicates-from-sorted-array)，让你在有序数组去重：

简单解释一下什么是原地修改：

如果不是原地修改的话，我们直接 new 一个 int[] 数组，把去重之后的元素放进这个新数组中，然后返回这个新数组即可。

但是现在题目让你原地删除，不允许 new 新数组，只能在原数组上操作，然后返回一个长度，这样就可以通过返回的长度和原始数组得到我们去重后的元素有哪些了。

由于数组已经排序，所以重复的元素一定连在一起，找出它们并不难。但如果毎找到一个重复元素就立即原地删除它，由于数组中删除元素涉及数据搬移，整个时间复杂度是会达到 O(n²)。

高效解决这道题就要用到快慢指针技巧：

我们让慢指针 `slow` 走在后面，快指针 `fast` 走在前面探路，找到一个不重复的元素就赋值给 `slow` 并让 `slow` 前进一步。

这样，就保证了 `nums[0..slow]` 都是无重复的元素，当 fast 指针遍历完整个数组 `nums` 后，`nums[0..slow]` 就是整个数组去重之后的结果。

看代码：

```go
func removeDuplicates(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    slow, fast := 0, 0
    for fast < len(nums) {
        if nums[fast] != nums[slow] {
            slow++
            // 维护 nums[0..slow] 无重复
            nums[slow] = nums[fast]
        }
        fast++
    }
    // 数组长度为索引 + 1
    return slow + 1
}
```

再简单扩展一下，看看力扣第 83 题[「删除排序链表中的重复元素」](https://leetcode.cn/problems/remove-duplicates-from-sorted-list)，如果给你一个有序的单链表，如何去重呢？

其实和数组去重是一模一样的，唯一的区别是把数组赋值操作变成操作指针而已，你对照着之前的代码来看：

```go
func deleteDuplicates(head *ListNode) *ListNode {
    if head == nil {
        return nil
    }
    slow := head
    fast := head
    for fast != nil {
        if fast.Val != slow.Val {
            // nums[slow] = nums[fast];
            slow.Next = fast
            // slow++;
            slow = slow.Next
        }
        // fast++
        fast = fast.Next
    }
    // 断开与后面重复元素的连接
    slow.Next = nil
    return head
}

```

**除了让你在有序数组/链表中去重，题目还可能让你对数组中的某些元素进行「原地删除」。**

比如力扣第 27 题[「移除元素」](https://leetcode.cn/problems/remove-element/)，看下题目：

题目要求我们把 `nums` 中所有值为 `val` 的元素原地删除，依然需要使用快慢指针技巧：

如果 `fast` 遇到值为 `val` 的元素，则直接跳过，否则就赋值给 `slow` 指针，并让 `slow` 前进一步。

这和前面说到的数组去重问题解法思路是完全一样的，直接看代码：

```go
func removeElement(nums []int, val int) int {
    slow := 0
    for fast := 0; fast < len(nums); fast++ {
        if nums[fast] != val {
            nums[slow] = nums[fast]
            slow++
        }
    }
    return slow
}
```

注意这里和有序数组去重的解法有一个细节差异，我们这里是先给 `nums[slow]` 赋值然后再给 `slow++`，这样可以保证 `nums[0..slow-1]` 是不包含值为 `val` 的元素的，最后的结果数组长度就是 `slow`。

实现了这个 `removeElement` 函数，接下来看看力扣第 283 题[「移动零」](https://leetcode.cn/problems/move-zeroes/)：

给你输入一个数组 `nums`，请你原地修改，将数组中的所有值为 0 的元素移到数组末尾：

比如说给你输入 `nums = [0,1,4,0,2]`，你的算法没有返回值，但是会把 `nums` 数组原地修改成 `[1,4,2,0,0]`。

结合之前说到的几个题目，你是否有已经有了答案呢？

稍微修改上一题中的 `removeElement` 函数就可以完成这道题，或者直接复用 `removeElement` 函数也可以。

题目让我们将所有 0 移到最后，其实就相当于移除 nums 中的所有 0，然后再把后面的元素都赋值为 0：

```go
func moveZeroes(nums []int) {
    // 去除 nums 中的所有 0
    // 返回去除 0 之后的数组长度
    p := removeElement(nums, 0)
    // 将 p 之后的所有元素赋值为 0
    for ; p < len(nums); p++ {
        nums[p] = 0
    }
}

// 双指针技巧，复用 [27. 移除元素] 的解法。
func removeElement(nums []int, val int) int {
    fast, slow := 0, 0
    for fast < len(nums) {
        if nums[fast] != val {
            nums[slow] = nums[fast]
            slow++
        }
        fast++
    }
    return slow
}
```

到这里，原地修改数组的这些题目就已经差不多了。

### 滑动窗口

数组中另一大类快慢指针的题目就是「滑动窗口算法」。我在另一篇文章 滑动窗口算法核心框架详解 给出了滑动窗口的代码框架：

```java
// 滑动窗口算法框架伪码
int left = 0, right = 0;

while (right < nums.size()) {
    // 增大窗口
    window.addLast(nums[right]);
    right++;

    while (window needs shrink) {
        // 缩小窗口
        window.removeFirst(nums[left]);
        left++;
    }
}
```

具体的题目本文就不重复了，这里只强调滑动窗口算法的快慢指针特性：

`left` 指针在后，`right` 指针在前，两个指针中间的部分就是「窗口」，算法通过扩大和缩小「窗口」来解决某些问题。

## 二、左右指针的常用算法

### 二分查找

我在另一篇文章 二分查找框架详解 中有详细探讨二分搜索代码的细节问题，这里只写最简单的二分算法，旨在突出它的双指针特性：

```go
func binarySearch(nums []int, target int) int {
	// 一左一右两个指针相向而行
	left, right := 0, len(nums)-1
	for left <= right {
		mid := (right + left) / 2
		if nums[mid] == target {
			return mid
		} else if nums[mid] < target {
			left = mid + 1
		} else if nums[mid] > target {
			right = mid - 1
		}
	}
	return -1
}
```

### n 数之和

看下力扣第 167 题[「两数之和 II」](https://leetcode.cn/problems/two-sum-ii-input-array-is-sorted)：

只要数组有序，就应该想到双指针技巧。这道题的解法有点类似二分查找，通过调节 `left` 和 `right` 就可以调整 `sum` 的大小：

```go
func twoSum(nums []int, target int) []int {
    // 一左一右两个指针相向而行
    left, right := 0, len(nums) - 1
    for left < right {
        sum := nums[left] + nums[right]
        if sum == target {
            // 题目要求的索引是从 1 开始的
            return []int{left + 1, right + 1}
        } else if sum < target {
            // 让 sum 大一点
            left++
        } else if sum > target {
            // 让 sum 小一点
            right--
        }
    }
    return []int{-1, -1}
}
```

我在另一篇文章 一个函数秒杀所有 nSum 问题 中也运用类似的左右指针技巧给出了 nSum 问题的一种通用思路，本质上利用的也是双指针技巧。

### 反转数组

一般编程语言都会提供 `reverse` 函数，其实这个函数的原理非常简单，力扣第 344 题[「反转字符串」](https://leetcode.cn/problems/reverse-string/)就是类似的需求，让你反转一个 `char[]` 类型的字符数组，我们直接看代码吧：

```go

func reverseString(s []rune) {
    // 一左一右两个指针相向而行
    left, right := 0, len(s)-1
    for left < right {
        // 交换 s[left] 和 s[right]
        temp := s[left]
        s[left] = s[right]
        s[right] = temp
        left++
        right--
    }
}
```

关于数组翻转的更多进阶问题，可以参见
二维数组的花式遍历。

### 回文串判断

回文串就是正着读和反着读都一样的字符串。比如说字符串 `aba` 和 `abba` 都是回文串，因为它们对称，反过来还是和本身一样；反之，字符串 `abac` 就不是回文串。

现在你应该能感觉到回文串问题和左右指针肯定有密切的联系，比如让你判断一个字符串是不是回文串，你可以写出下面这段代码：

```go
func isPalindrome(s string) bool {
    // 一左一右两个指针相向而行
    left, right := 0, len(s) - 1
    for left < right {
        if s[left] != s[right] {
            return false
        }
        left++
        right--
    }
    return true
}
```

那接下来我提升一点难度，给你一个字符串，让你用双指针技巧从中找出最长的回文串，你会做吗？

这就是力扣第 5 题[「最长回文子串」](https://leetcode.cn/problems/longest-palindromic-substring/)：

找回文串的难点在于，回文串的的长度可能是奇数也可能是偶数，解决该问题的核心是从中心向两端扩散的双指针技巧。

如果回文串的长度为奇数，则它有一个中心字符；如果回文串的长度为偶数，则可以认为它有两个中心字符。所以我们可以先实现这样一个函数：

```go

// 在 s 中寻找以 s[l] 和 s[r] 为中心的最长回文串
func palindrome(s string, l int, r int) string {
    // 防止索引越界
    for l >= 0 && r < len(s) && s[l] == s[r] {
        // 双指针，向两边展开
        l--
        r++
    }
    // 此时 s[l+1..r-1] 就是最长回文串
    return s[l+1 : r]
}
```

这样，如果输入相同的 `l` 和 `r`，就相当于寻找长度为奇数的回文串，如果输入相邻的 `l` 和 `r`，则相当于寻找长度为偶数的回文串。

那么回到最长回文串的问题，解法的大致思路就是：

```python
for 0 <= i < len(s):
    找到以 s[i] 为中心的回文串
    找到以 s[i] 和 s[i+1] 为中心的回文串
    更新答案
```

翻译成代码，就可以解决最长回文子串这个问题：

```go
func longestPalindrome(s string) string {
    res := ""
    for i := 0; i < len(s); i++ {
        // 以 s[i] 为中心的最长回文子串
        s1 := palindrome(s, i, i)
        // 以 s[i] 和 s[i+1] 为中心的最长回文子串
        s2 := palindrome(s, i, i+1)
        // res = longest(res, s1, s2)
        if len(res) > len(s1) {
            res = res
        } else {
            res = s1
        }
        if len(res) > len(s2) {
            res = res
        } else {
            res = s2
        }
    }
    return res
}

func palindrome(s string, l, r int) string {
    // 防止索引越界
    for l >= 0 && r < len(s) && s[l] == s[r] {
        // 向两边展开
        l--
        r++
    }
    // 此时 s[l+1..r-1] 就是最长回文串
    return s[l+1:r]
}
```

你应该能发现最长回文子串使用的左右指针和之前题目的左右指针有一些不同：之前的左右指针都是从两端向中间相向而行，而回文子串问题则是让左右指针从中心向两端扩展。不过这种情况也就回文串这类问题会遇到，所以我也把它归为左右指针了。

到这里，数组相关的双指针技巧就全部讲完了，这些技巧的更多扩展延伸见
更多数组双指针经典高频题。
