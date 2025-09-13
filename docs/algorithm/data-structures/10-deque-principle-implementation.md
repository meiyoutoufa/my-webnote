---
title: 双端队列（Deque）原理及实现
date: 2023-09-01
author: mikez
description: 双端队列（Deque）原理及实现
weight: 10
tags:
  - 数据结构的基础
---

## 基本原理

如果你理解了前面讲解的内容，这个双端队列其实没啥可讲的了。所谓双端队列，主要是对比标准队列（FIFO 先进先出队列）多了一些操作罢了：

```go
// MyDeque represents a deque (double-ended queue) data structure
type MyDeque[E any] struct {
}

// 从队头插入元素，时间复杂度 O(1)
func (d *MyDeque[E]) AddFirst(e E) {
}

// 从队尾插入元素，时间复杂度 O(1)
func (d *MyDeque[E]) AddLast(e E) {
}

// 从队头删除元素，时间复杂度 O(1)
func (d *MyDeque[E]) RemoveFirst() E {
}

// 从队尾删除元素，时间复杂度 O(1)
func (d *MyDeque[E]) RemoveLast() E {
}

// 查看队头元素，时间复杂度 O(1)
func (d *MyDeque[E]) PeekFirst() E {
}

// 查看队尾元素，时间复杂度 O(1)
func (d *MyDeque[E]) PeekLast() E {
}
```

标准队列 只能在队尾插入元素，队头删除元素，而双端队列的队头和队尾都可以插入或删除元素。

普通队列就好比排队买票，先来的先买，后来的后买；而双端队列就好比一个过街天桥，两端都可以随意进出。当然，双端队列的元素就不再满足「先进先出」了，因为它比较灵活嘛。

在做算法题的场景中，双端队列用的不算很多。感觉只有 Python 用到的多一些，因为 Python 标准库没有提供内置的栈和队列，一般会用双端队列来模拟标准队列。

## 用链表实现双端队列

很简单吧，直接复用我们之前实现的 MyLinkedList 类，或者使用编程语言标准库提供的双链表结构就行了。因为双链表本就支持 O(1) 时间复杂度在链表的头尾增删元素：

```go
package main

import (
	"container/list"
	"fmt"
)

type MyListDeque struct {
	list *list.List
}

func NewMyListDeque() *MyListDeque {
	return &MyListDeque{list: list.New()}
}

// 从队头插入元素，时间复杂度 O(1)
func (d *MyListDeque) AddFirst(e interface{}) {
	d.list.PushFront(e)
}

// 从队尾插入元素，时间复杂度 O(1)
func (d *MyListDeque) AddLast(e interface{}) {
	d.list.PushBack(e)
}

// 从队头删除元素，时间复杂度 O(1)
func (d *MyListDeque) RemoveFirst() interface{} {
	if elem := d.list.Front(); elem != nil {
		return d.list.Remove(elem)
	}
	return nil
}

// 从队尾删除元素，时间复杂度 O(1)
func (d *MyListDeque) RemoveLast() interface{} {
	if elem := d.list.Back(); elem != nil {
		return d.list.Remove(elem)
	}
	return nil
}

// 查看队头元素，时间复杂度 O(1)
func (d *MyListDeque) PeekFirst() interface{} {
	if elem := d.list.Front(); elem != nil {
		return elem.Value
	}
	return nil
}

// 查看队尾元素，时间复杂度 O(1)
func (d *MyListDeque) PeekLast() interface{} {
	if elem := d.list.Back(); elem != nil {
		return elem.Value
	}
	return nil
}

func main() {
	deque := NewMyListDeque()
	deque.AddFirst(1)
	deque.AddFirst(2)
	deque.AddLast(3)
	deque.AddLast(4)

	fmt.Println(deque.RemoveFirst()) // 2
	fmt.Println(deque.RemoveLast())  // 4
	fmt.Println(deque.PeekFirst())   // 1
	fmt.Println(deque.PeekLast())    // 3
}
```

## 用数组实现双端队列

也很简单吧，直接复用我们在 [环形数组技巧](./05-circular-array-implementation.md) 中实现的 `CycleArray` 提供的方法就行了。环形数组头尾增删元素的复杂度都是 O(1)：

```go
// MyArrayDeque is a generic deque implemented using a CycleArray
type MyArrayDeque[E any] struct {
	arr CycleArray[E]
}

// NewMyArrayDeque creates a new MyArrayDeque
func NewMyArrayDeque[E any]() *MyArrayDeque[E] {
	return &MyArrayDeque[E]{arr: CycleArray[E]{}}
}

// 从队头插入元素，时间复杂度 O(1)
func (d *MyArrayDeque[E]) AddFirst(e E) {
	d.arr.AddFirst(e)
}

// 从队尾插入元素，时间复杂度 O(1)
func (d *MyArrayDeque[E]) AddLast(e E) {
	d.arr.AddLast(e)
}

// 从队头删除元素，时间复杂度 O(1)
func (d *MyArrayDeque[E]) RemoveFirst() E {
	return d.arr.RemoveFirst()
}

// 从队尾删除元素，时间复杂度 O(1)
func (d *MyArrayDeque[E]) RemoveLast() E {
	return d.arr.RemoveLast()
}

// 查看队头元素，时间复杂度 O(1)
func (d *MyArrayDeque[E]) PeekFirst() E {
	return d.arr.GetFirst()
}

// 查看队尾元素，时间复杂度 O(1)
func (d *MyArrayDeque[E]) PeekLast() E {
	return d.arr.GetLast()
}
```
