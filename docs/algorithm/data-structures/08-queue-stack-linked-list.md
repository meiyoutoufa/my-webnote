---
title: 用链表实现队列和栈
date: 2023-09-01
author: mikez
description: 用链表实现队列和栈
weight: 8
tags:
  - 数据结构的基础
---

## 用链表实现栈

一些读者应该已经知道该怎么用链表作为底层数据结构实现队列和栈了。因为实在是太简单了，直接调用双链表的 API 就可以了。

注意我这里是直接用的 Java 标准库的 LinkedList，如果你用之前我们实现的 MyLinkedList，也是一样的。

```go
import "container/list"

// 用链表作为底层数据结构实现栈
type MyLinkedStack struct {
    list *list.List
}

func NewMyLinkedStack() *MyLinkedStack {
    return &MyLinkedStack{list: list.New()}
}

// 向栈顶加入元素，时间复杂度 O(1)
func (s *MyLinkedStack) Push(e interface{}) {
    s.list.PushBack(e)
}

// 从栈顶弹出元素，时间复杂度 O(1)
func (s *MyLinkedStack) Pop() interface{} {
    element := s.list.Back()
    if element != nil {
        s.list.Remove(element)
        return element.Value
    }
    return nil
}

// 查看栈顶元素，时间复杂度 O(1)
func (s *MyLinkedStack) Peek() interface{} {
    element := s.list.Back()
    if element != nil {
        return element.Value
    }
    return nil
}

// 返回栈中的元素个数，时间复杂度 O(1)
func (s *MyLinkedStack) Size() int {
    return s.list.Len()
}

// test
func main() {
    stack := NewMyLinkedStack()
    stack.Push(1)
    stack.Push(2)
    stack.Push(3)
    fmt.Println(stack.Pop()) // 3
    fmt.Println(stack.Pop()) // 2
    fmt.Println(stack.Peek()) // 1
}
```

```text
提示
上面这段代码相当于是把双链表的尾部作为栈顶，在双链表尾部增删元素的时间复杂度都是 O(1)，符合要求。
当然，你也可以把双链表的头部作为栈顶，因为双链表头部增删元素的时间复杂度也是 O(1)，所以这样实现也是一样的。只要做几个修改 addLast -> addFirst，removeLast -> removeFirst，getLast -> getFirst 就行了。
```

## 用链表实现队列

同理，用链表实现队列也是一样的，也直接调用双链表的 API 就可以了：

```go
import (
	"container/list"
	"fmt"
)

// 用链表作为底层数据结构实现队列
type MyLinkedQueue struct {
	list *list.List
}

// 构造函数
func NewMyLinkedQueue() *MyLinkedQueue {
	return &MyLinkedQueue{list: list.New()}
}

// 向队尾插入元素，时间复杂度 O(1)
func (q *MyLinkedQueue) Push(e interface{}) {
	q.list.PushBack(e)
}

// 从队头删除元素，时间复杂度 O(1)
func (q *MyLinkedQueue) Pop() interface{} {
	front := q.list.Front()
	if front != nil {
		return q.list.Remove(front)
	}
	return nil
}

// 查看队头元素，时间复杂度 O(1)
func (q *MyLinkedQueue) Peek() interface{} {
	front := q.list.Front()
	if front != nil {
		return front.Value
	}
	return nil
}

// 返回队列中的元素个数，时间复杂度 O(1)
func (q *MyLinkedQueue) Size() int {
	return q.list.Len()
}


func main() {
	queue := NewMyLinkedQueue()
	queue.Push(1)
	queue.Push(2)
	queue.Push(3)
	fmt.Println(queue.Peek()) // 1
	fmt.Println(queue.Pop())  // 1
	fmt.Println(queue.Pop())  // 2
	fmt.Println(queue.Peek()) // 3
}
```

```text
提示
上面这段代码相当于是把双链表的尾部作为队尾，把双链表的头部作为队头，在双链表的头尾增删元素的复杂度都是 O(1)，符合队列 API 的要求。
当然，你也可以反过来，把双链表的头部作为队尾，双链表的尾部作为队头。类似栈的实现，只要改一改 list 的调用方法就行了。
```
