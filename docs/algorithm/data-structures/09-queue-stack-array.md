---
title: 用数组实现队列和栈
date: 2023-09-01
author: mikez
description: 用数组实现队列和栈
weight: 9
tags:
  - 数据结构的基础
---

这篇文章带大家用数组作为底层数据结构实现队列和栈。

## 用数组实现栈

先用数组实现栈，这个不难，你把动态数组的尾部作为栈顶，然后调用动态数组的 API 就行了。因为数组尾部增删元素的时间复杂度都是 O(1)，符合栈的要求。

注意我这里用的是 Java 标准库的 ArrayList，如果你想用之前我们实现的 MyArrayList，也是一样的：

```go
import "fmt"

// MyArrayStack 用数组切片作为底层数据结构实现栈
type MyArrayStack[T any] struct {
    list []T
}

// 向栈顶加入元素，时间复杂度 O(1)
func (s *MyArrayStack[T]) Push(e T) {
    s.list = append(s.list, e)
}

// 从栈顶弹出元素，时间复杂度 O(1)
func (s *MyArrayStack[T]) Pop() T {
    if len(s.list) == 0 {
        var zero T
        return zero
    }
    e := s.list[len(s.list)-1]
    s.list = s.list[:len(s.list)-1]
    return e
}

// 查看栈顶元素，时间复杂度 O(1)
func (s *MyArrayStack[T]) Peek() T {
    if len(s.list) == 0 {
        var zero T
        return zero
    }
    return s.list[len(s.list)-1]
}

// 返回栈中的元素个数，时间复杂度 O(1)
func (s *MyArrayStack[T]) Size() int {
    return len(s.list)
}
```

能否让数组的头部作为栈顶？

按照我们之前实现 MyArrayList 的逻辑，是不行的。因为数组头部增删元素的时间复杂度都是 O(n)，不符合要求。

但是我们可以改用前文 [环形数组技巧](05-circular-array-implementation.md) 中实现的 `CycleArray` 类，这个数据结构在头部增删元素的时间复杂度是 O(1)，符合栈的要求。

你直接调用 `CycleArray` 的 `addFirst` 和 `removeFirs`t 方法实现栈的 API 就行，我这里就不写了。

## 用数组实现队列

有了前文 环形数组 中实现的 CycleArray 类，用数组作为底层数据结构实现队列就不难了吧。直接复用我们实现的 `CycleArray`，就可以实现标准队列了。当然，一些编程语言也有内置的环形数组实现，你也可以自行搜索使用：

```go
type MyArrayQueue[E any] struct {
    arr *CycleArray[E]
}

func NewMyArrayQueue[E any]() *MyArrayQueue[E] {
    return &MyArrayQueue[E]{
        arr: NewCycleArray[E](),
    }
}

func (q *MyArrayQueue[E]) Push(t E) {
    q.arr.AddLast(t)
}

func (q *MyArrayQueue[E]) Pop() E {
    return q.arr.RemoveFirst()
}

func (q *MyArrayQueue[E]) Peek() E {
    return q.arr.GetFirst()
}

func (q *MyArrayQueue[E]) Size() int {
    return q.arr.Size()
}
```
