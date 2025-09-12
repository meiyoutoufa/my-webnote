---
title: 用拉链法实现哈希表
date: 2023-09-01
author: mikez
description: 用拉链法实现哈希表
weight: 12
tags:
  - 数据结构的基础
---

前文 [哈希表核心原理](./11-哈希表核心原理.md) 中我介绍了哈希表的核心原理和几个关键概念，其中提到了解决哈希冲突的方法主要有两种，分别是拉链法和开放寻址法（也常叫做线性探查法）：
![hash-collision](/images/algorithm/hash-collision.jpeg)

本文就来具体介绍一下拉链法的实现原理和代码。

## 拉链法的简化版实现

[哈希表核心原理](./11-哈希表核心原理.md) 已经介绍过哈希函数和 `ke`y 的类型的关系，其中 `hash` 函数的作用是在 O(1) 的时间把 key 转化成数组的索引，而 key 可以是任意不可变的类型。

但是这里为了方便诸位理解，我先做如下简化：

1、我们实现的哈希表只支持 `key` 类型为 `int`，`value` 类型为 `int` 的情况，如果 `key` 不存在，就返回 `-1`。

2、我们实现的 hash 函数就是简单地取模，即 `hash(key) = key % table.length`。这样也方便模拟出哈希冲突的情况，比如当 `table.length = 10` 时，`hash(1)` 和 `hash(11)` 的值都是 1。

3、底层的 `table` 数组的大小在创建哈希表时就固定，不考虑负载因子和动态扩缩容的问题。

这些简化能够帮助我们聚焦增删查改的核心逻辑，并且可以借助
可视化面板 辅助大家学习理解。

### 简化版代码

这里直接给出简化版的代码实现，你可以先看看，后面将通过可视化面板展示增删查改的过程：

```go
import "container/list"

// 用拉链法解决哈希冲突的简化实现
type KVNode struct {
    key   int
    value int
}

// 底层 table 数组中的每个元素是一个链表
type ExampleChainingHashMap struct {
    table []*list.List
}

// 注意这里必须存储同时存储 key 和 value
// 因为要通过 key 找到对应的 value
func NewExampleChainingHashMap(capacity int) ExampleChainingHashMap {
    return ExampleChainingHashMap{
        table: make([]*list.List, capacity),
    }
}

func (h *ExampleChainingHashMap) hash(key int) int {
    return key % len(h.table)
}

// 查
func (h *ExampleChainingHashMap) Get(key int) (int, bool) {
    index := h.hash(key)
    if h.table[index] == nil {
        // 链表为空，说明 key 不存在
        return -1, false
    }

    for e := h.table[index].Front(); e != nil; e = e.Next() {
        node := e.Value.(KVNode)
        if node.key == key {
            return node.value, true
        }
    }

    // 链表中没有目标 key
    return -1, false
}

// 增/改
func (h *ExampleChainingHashMap) Put(key int, value int) {
    index := h.hash(key)
    if h.table[index] == nil {
        // 链表为空，新建一个链表，插入 key-value
        h.table[index] = list.New()
        h.table[index].PushBack(KVNode{key, value})
        return
    }

    for e := h.table[index].Front(); e != nil; e = e.Next() {
        node := e.Value.(KVNode)
        if node.key == key {
            // key 已经存在，更新 value
            node.value = value
            return
        }
    }

    // 链表中没有目标 key，添加新节点
    // 这里使用 addFirst 添加到链表头部或者 addLast 添加到链表尾部都可以
    // 因为 Java LinkedList 的底层实现是双链表，头尾操作都是 O(1) 的
    h.table[index].PushBack(KVNode{key, value})
}

// 删
func (h *ExampleChainingHashMap) Remove(key int) {
    index := h.hash(key)
    if h.table[index] == nil {
        return
    }

    for e := h.table[index].Front(); e != nil; e = e.Next() {
        node := e.Value.(KVNode)
        if node.key == key {
            // 如果 key 存在，则删除
            h.table[index].Remove(e)
            return
        }
    }
}
```

### 完整代码实现

有了上面的铺垫，我们现在来看一个比较完善的 Java 代码实现，主要新增了以下几个功能：

1、使用了泛型，可以存储任意类型的 `key` 和 `value`。

2、底层的 `table` 数组会根据负载因子动态扩缩容。

3、使用了
[哈希表基础](./11-哈希表核心原理.md) 中提到的 `hash` 函数，利用 `key` 的 `hashCode()` 方法和 `table.length` 来计算哈希值。

4、实现了 `keys()` 方法，可以返回哈希表中所有的 `key`。

为了方便，我直接使用 Java 标准库的 `LinkedList` 作为链表，这样就不用自己手动处理增删链表节点的操作了。当然如果你愿意的话，可以参照前文
[单/双链表代码实现](04-链表代码实现.md) 自己编写操作 `KVNode` 的逻辑。

具体需要注意的细节我都写在了代码注释中，直接看代码吧：

```go
package main

import (
	"container/list"
	"errors"
	"fmt"
)

type MyChainingHashMap[K comparable, V any] struct {
	// 哈希表的底层数组，每个数组元素是一个链表，链表中每个节点是 KVNode 存储键值对
	table []*list.List

	// 哈希表中存入的键值对个数
	size int
}

// 拉链法使用的单链表节点，存储 key-value 对
type KVNode[K comparable, V any] struct {
	key   K
	value V
}

// 底层数组的初始容量
const INIT_CAP = 4

// 创建一个新的 MyChainingHashMap，使用默认的初始容量
func NewMyChainingHashMap[K comparable, V any]() *MyChainingHashMap[K, V] {
	return NewMyChainingHashMapWithCapacity[K, V](INIT_CAP)
}

// 使用指定的初始容量创建 MyChainingHashMap
func NewMyChainingHashMapWithCapacity[K comparable, V any](initCapacity int) *MyChainingHashMap[K, V] {
	if initCapacity < 1 {
		initCapacity = 1
	}
	m := &MyChainingHashMap[K, V]{
		table: make([]*list.List, initCapacity),
	}
	for i := range m.table {
		m.table[i] = list.New()
	}
	return m
}

// 添加 key -> val 键值对
func (m *MyChainingHashMap[K, V]) Put(key K, val V) error {
	if key == *new(K) {
		// 相当于 nil 检查
		return errors.New("key is null")
	}
	index := m.hash(key)
	l := m.table[index]
	// 如果 key 之前存在，则修改对应的 val
	for e := l.Front(); e != nil; e = e.Next() {
		node := e.Value.(KVNode[K, V])
		if node.key == key {
			e.Value = KVNode[K, V]{key, val}
			return nil
		}
	}
	// 如果 key 之前不存在，则插入，size 增加
	l.PushBack(KVNode[K, V]{key, val})
	m.size++

	// 如果元素数量超过了负载因子，进行扩容
	if m.size >= len(m.table)*3/4 {
		m.resize(len(m.table) * 2)
	}
	return nil
}

// 删除 key 和对应的 val
func (m *MyChainingHashMap[K, V]) Remove(key K) error {
	if key == *new(K) {
		return errors.New("key is null")
	}
	index := m.hash(key)
	l := m.table[index]
	// 如果 key 存在，则删除，size 减少
	for e := l.Front(); e != nil; e = e.Next() {
		node := e.Value.(KVNode[K, V])
		if node.key == key {
			l.Remove(e)
			m.size--

			// 缩容，当负载因子小于 0.125 时，缩容
			if m.size <= len(m.table)/8 {
				m.resize(len(m.table) / 2)
			}
			return nil
		}
	}
	return nil
}

// 返回 key 对应的 val，如果 key 不存在，则返回 nil
func (m *MyChainingHashMap[K, V]) Get(key K) *V {
	if key == *new(K) {
		return nil
	}
	index := m.hash(key)
	l := m.table[index]
	for e := l.Front(); e != nil; e = e.Next() {
		node := e.Value.(KVNode[K, V])
		if node.key == key {
			return &node.value
		}
	}
	return nil
}

// 返回所有 key
func (m *MyChainingHashMap[K, V]) Keys() []K {
	var keys []K
	for _, l := range m.table {
		for e := l.Front(); e != nil; e = e.Next() {
			node := e.Value.(KVNode[K, V])
			keys = append(keys, node.key)
		}
	}
	return keys
}

// 返回当前哈希表中的键值对个数
func (m *MyChainingHashMap[K, V]) Size() int {
	return m.size
}

// 哈希函数，将键映射到 table 的索引
func (m *MyChainingHashMap[K, V]) hash(key K) int {
	return int(fmt.Sprintf("%v", key)[0]) % len(m.table)
}

// 扩展或缩小哈希表的容量
func (m *MyChainingHashMap[K, V]) resize(newCap int) {
	if newCap < 1 {
		newCap = 1
	}
	newMap := NewMyChainingHashMapWithCapacity[K, V](newCap)
	for _, l := range m.table {
		for e := l.Front(); e != nil; e = e.Next() {
			node := e.Value.(KVNode[K, V])
			newMap.Put(node.key, node.value)
		}
	}
	m.table = newMap.table
}

// 测试代码
func main() {
	map1 := NewMyChainingHashMap[int, int]()
	map1.Put(1, 1)
	map1.Put(2, 2)
	map1.Put(3, 3)
	fmt.Println(*map1.Get(1)) // 1
	fmt.Println(*map1.Get(2)) // 2

	map1.Put(1, 100)
	fmt.Println(*map1.Get(1)) // 100

	map1.Remove(2)
	fmt.Println(map1.Get(2)) // nil

	fmt.Println(map1.Keys()) // [1, 3]

	map1.Remove(1)
	map1.Remove(3)
	fmt.Println(map1.Get(1)) // nil
}
```
