---
title: 线性探查法的两种代码实现
date: 2023-09-01
author: mikez
description: 线性探查法的两种代码实现
weight: 14
tags:
  - 数据结构的基础
---

前文
[哈希表核心原理](11-哈希表核心原理.md) 中我介绍了哈希表的核心原理和几个关键概念，
[拉链法原理和实现](12-用拉链法实现哈希表.md) 中介绍了拉链法的实现，
[线性探查法的两个难点](./13-线性探查法的两个难点.md) 介绍了线性探查法实现哈希表的难点所在，并给出了两种方法解决删除元素时的空洞问题，本文会同时给出这两种方法的参考代码实现。

本文会先结合可视化面板给出简化的实现，方便大家理解增删查改的过程，最后给完整实现。

简化实现中，具体简化的地方如下：

1、我们实现的哈希表只支持 `key` 类型为 `int`，`value` 类型为 `int` 的情况，如果 `key` 不存在，就返回 -1。

2、我们实现的 hash 函数就是简单地取模，即 `hash(key) = key % table.length`。这样也方便模拟出哈希冲突的情况，比如当 `table.length = 10` 时，`hash(1)` 和 `hash(11)` 的值都是 1。

3、底层的 `table` 数组的大小在创建哈希表时就固定，假设 `table` 数组不会被装满，不考虑负载因子和动态扩缩容的问题。

## 简化版代码

### 搬移数据的线性探查法

这种方法在 `remove` 操作时，会将删除的元素后面的元素重新插入哈希表，以此保持元素的连续性。

这里直接给出简化版的代码实现，你可以先看看，后面将通过可视化面板展示增删查改的过程：

```go
import "fmt"

// 定义节点结构体
type Node struct {
	key int
	val int
}

// 定义哈希表结构体
type ExampleLinearProbingHashMap1 struct {
	table []*Node
}

// 初始化哈希表
func NewExampleLinearProbingHashMap1(cap int) *ExampleLinearProbingHashMap1 {
	table := make([]*Node, cap)

	return &ExampleLinearProbingHashMap1{
		table: table,
	}
}

// 增/改
func (e *ExampleLinearProbingHashMap1) Put(key int, value int) {
	index := e.findKeyIndex(key)
	e.table[index] = &Node{
		key: key,
		val: value,
	}
}

// 查，找不到就返回 -1
func (e *ExampleLinearProbingHashMap1) Get(key int) int {
	index := e.findKeyIndex(key)
	if e.table[index] == nil {
		return -1
	}
	return e.table[index].val
}

// 删
func (e *ExampleLinearProbingHashMap1) Remove(key int) {
	index := e.findKeyIndex(key)
	if e.table[index] == nil {
		return
	}

	e.table[index] = nil
	index = (index + 1) % len(e.table)

	for e.table[index] != nil {
		tmp := e.table[index]
		e.table[index] = nil
		// 这个操作是关键，利用 put 方法，将键值对重新插入
		// 这样就能把它们移动到正确的 table 索引位置
		e.Put(tmp.key, tmp.val)
		index = (index + 1) % len(e.table)
	}
}

// 线性探测法查找 key 在 table 中的索引
// 如果找不到，返回的就是下一个为 null 的索引，可用于插入
func (e *ExampleLinearProbingHashMap1) findKeyIndex(key int) int {
	index := e.hash(key)
	for e.table[index] != nil {
		if e.table[index].key == key {
			return index
		}
		index = (index + 1) % len(e.table)
	}
	return index
}

// 散列函数
func (e *ExampleLinearProbingHashMap1) hash(key int) int {
	return key % len(e.table)
}

func main() {
	map1 := NewExampleLinearProbingHashMap1(10)
	map1.Put(1, 1)
	map1.Put(2, 2)
	map1.Put(10, 10)
	map1.Put(20, 20)
	map1.Put(30, 30)
	map1.Put(3, 3)
	fmt.Println(map1.Get(1)) // 1
	fmt.Println(map1.Get(2)) // 2
	fmt.Println(map1.Get(20)) // 20

	map1.Put(1, 100)
	fmt.Println(map1.Get(1)) // 100

	map1.Remove(20)
	fmt.Println(map1.Get(20)) // -1
	fmt.Println(map1.Get(30)) // 30
}
```

### 特殊占位符的线性探查法

这种方法通过一个 DELETED 特殊值作为占位符，标记被删除的元素。

这个方法与上面的方法最大的区别在于 findKeyIndex 方法的实现，同时需要对 DELETED 特殊处理。直接看代码吧，后面会通过可视化面板展示增删查改的过程：

```go
// 用线性探查法解决哈希冲突的简化实现（特殊占位符版）

struct KVNode {
    key int
    val int
}

type ExampleLinearProbingHashMap2 struct {
    // 真正存储键值对的数组
    table []*KVNode
    // 用于标记被删元素的占位符
    DELETED *KVNode
}

// 哈希函数，将键映射到 table 的索引
func (e *ExampleLinearProbingHashMap2) hash(key int) int {
    return key % len(e.table)
}

// 线性探测法查找 key 在 table 中的索引，如果找不到，返回 -1
func (e *ExampleLinearProbingHashMap2) findKeyIndex(key int) int {
    // 因为删除元素时只是标记为 DELETED，并不是真的删除，所以 table 可能会被填满，导致死循环
    // step 用来记录查找的步数，防止死循环
    for i, step := e.hash(key), 0; e.table[i] != nil; i = (i+1) % len(e.table) {
        if step++; step > len(e.table) {
            return -1
        }
        // 遇到占位符直接跳过
        if e.table[i] == e.DELETED {
            continue
        }
        if e.table[i].key == key {
            return i
        }
    }

    return -1
}

// 增/改
func (e *ExampleLinearProbingHashMap2) put(key int, val int) {
    index := e.findKeyIndex(key)
    // key 已存在，修改对应的 val，如果 key 不存在，新建节点并插入表中
    if index != -1 && e.table[index] != nil {
        e.table[index].val = val
        return
    }

    node := &KVNode{key, val}
    index = e.hash(key)
    for e.table[index] != nil && e.table[index] != e.DELETED {
        index = (index+1) % len(e.table)
    }
    e.table[index] = node
}

// 删
func (e *ExampleLinearProbingHashMap2) remove(key int) {
    index := e.findKeyIndex(key)
    // key 不存在，不需要 remove
    if index == -1 {
        return
    }
    // 直接用占位符表示删除
    e.table[index] = e.DELETED
}

// 查，返回 key 对应的 val，如果 key 不存在，则返回 -1
func (e *ExampleLinearProbingHashMap2) get(key int) int {
    index := e.findKeyIndex(key)
    if index == -1 {
        return -1
    }
    return e.table[index].val
}

func main() {
    map := &ExampleLinearProbingHashMap2{
        table: make([]*KVNode, 10),
        DELETED: &KVNode{-2, -2},
    }
    map.put(1, 1)
    map.put(2, 2)
    map.put(10, 10)
    map.put(20, 20)
    map.put(30, 30)
    map.put(3, 3)
    fmt.Println(map.get(1)) // 1
    fmt.Println(map.get(2)) // 2
    fmt.Println(map.get(20)) // 20

    map.put(1, 100)
    fmt.Println(map.get(1)) // 100

    map.remove(20)
    fmt.Println(map.get(20)) // -1
    fmt.Println(map.get(30)) // 30
}
```

### 特殊值标记版本

```go
package main

import (
	"errors"
	"fmt"
)

// KVNode 表示键值对的节点
type KVNode struct {
	key interface{}
	val interface{}
}

// MyLinearProbingHashMap2 是线性探查哈希表的结构体
type MyLinearProbingHashMap2 struct {
	table []*KVNode
	size  int
}

// 被删除的 KVNode 的占位符
var dummy = &KVNode{nil, nil}

// 默认的初始化容量
const initCap = 4

// 构造函数，初始化容量
func NewMyLinearProbingHashMap2(cap int) *MyLinearProbingHashMap2 {
	if cap <= 0 {
		cap = initCap
	}
	return &MyLinearProbingHashMap2{
		table: make([]*KVNode, cap),
		size:  0,
	}
}

// **** 增/改 ****

// 添加 key -> val 键值对
// 如果键 key 已存在，则将值修改为 val
func (m *MyLinearProbingHashMap2) Put(key, val interface{}) error {
	if key == nil {
		return errors.New("key is null")
	}

	// 负载因子默认设为 0.75，超过则扩容
	if m.size >= len(m.table)*3/4 {
		m.resize(len(m.table) * 2)
	}

	index := m.getKeyIndex(key)
	if index != -1 {
		// key 已存在，修改对应的 val
		m.table[index].val = val
		return nil
	}

	// key 不存在
	x := &KVNode{key, val}
	// 在 table 中找一个空位或者占位符，插入
	index = m.hash(key)
	for m.table[index] != nil && m.table[index] != dummy {
		index = (index + 1) % len(m.table)
	}
	m.table[index] = x
	m.size++
	return nil
}

// **** 删 ****

// 删除 key 和对应的 val，并返回 val
// 若 key 不存在，则返回 nil
func (m *MyLinearProbingHashMap2) Remove(key interface{}) error {
	if key == nil {
		return errors.New("key is null")
	}

	// 缩容
	if m.size < len(m.table)/8 {
		m.resize(len(m.table) / 2)
	}

	index := m.getKeyIndex(key)
	if index == -1 {
		// key 不存在，不需要 remove
		return nil
	}

	// 开始 remove
	// 直接用占位符表示删除
	m.table[index] = dummy
	m.size--
	return nil
}

// **** 查 ****

// 返回 key 对应的 val
// 如果 key 不存在，则返回 nil
func (m *MyLinearProbingHashMap2) Get(key interface{}) interface{} {
	if key == nil {
		return nil
	}

	index := m.getKeyIndex(key)
	if index == -1 {
		return nil
	}

	return m.table[index].val
}

// 检查是否包含指定的 key
func (m *MyLinearProbingHashMap2) ContainsKey(key interface{}) bool {
	return m.getKeyIndex(key) != -1
}

// 返回所有键的列表
func (m *MyLinearProbingHashMap2) Keys() []interface{} {
	keys := []interface{}{}
	for _, entry := range m.table {
		if entry != nil && entry != dummy {
			keys = append(keys, entry.key)
		}
	}
	return keys
}

// 返回哈希表中键值对的数量
func (m *MyLinearProbingHashMap2) Size() int {
	return m.size
}

// 对 key 进行线性探查，返回一个索引
// 根据 table[i] 是否为 nil 判断是否找到对应的 key
func (m *MyLinearProbingHashMap2) getKeyIndex(key interface{}) int {
	step := 0
	for i := m.hash(key); m.table[i] != nil; i = (i + 1) % len(m.table) {
		step++
		// 防止死循环
		if step > len(m.table) {
			// 这里可以触发一次 resize，把标记为删除的占位符清理掉
			m.resize(len(m.table))
			return -1
		}
		entry := m.table[i]
		// 遇到占位符直接跳过
		if entry == dummy {
			continue
		}
		if entry.key == key {
			return i
		}
	}
	return -1
}

// 哈希函数，将键映射到 table 的索引
// [0, len(table) - 1]
func (m *MyLinearProbingHashMap2) hash(key interface{}) int {
	return int(fmt.Sprintf("%v", key)[0]) % len(m.table)
}

// 扩容或缩容函数
func (m *MyLinearProbingHashMap2) resize(cap int) {
	newMap := NewMyLinearProbingHashMap2(cap)
	for _, entry := range m.table {
		if entry != nil && entry != dummy {
			newMap.Put(entry.key, entry.val)
		}
	}
	m.table = newMap.table
}

func main() {
	mapInstance := NewMyLinearProbingHashMap2(4)
	mapInstance.Put(1, 1)
	mapInstance.Put(2, 2)
	mapInstance.Put(10, 10)
	mapInstance.Put(20, 20)
	mapInstance.Put(30, 30)
	mapInstance.Put(3, 3)

	fmt.Println(mapInstance.Get(1))  // 1
	fmt.Println(mapInstance.Get(2))  // 2
	fmt.Println(mapInstance.Get(20)) // 20

	mapInstance.Put(1, 100)
	fmt.Println(mapInstance.Get(1)) // 100

	mapInstance.Remove(20)
	fmt.Println(mapInstance.Get(20)) // nil
	fmt.Println(mapInstance.Get(30)) // 30
}
```

### 完整版代码

有了上面的铺垫，我们现在来看比较完善的 Java 代码实现，主要新增了以下几个功能：

1、使用了泛型，可以存储任意类型的 `key` 和 `value`。

2、底层的 `table` 数组会根据负载因子动态扩缩容。

3、使用了
哈希表基础 中提到的 `hash` 函数，利用 `key` 的 `hashCode()` 方法和 `table.length` 来计算哈希值。

4、实现了 `keys()` 方法，可以返回哈希表中所有的 `key`。

下面我会分别给出 rehash 版本和特殊值标记版本的实现，具体细节可以参考代码注释。

#### rehash 版本

```go
package main

import (
	"fmt"
	"hash/fnv"
)

type KVNode[K comparable, V any] struct {
	key K
	val V
}

// 线性探查哈希表
type MyLinearProbingHashMap1[K comparable, V any] struct {
	table []*KVNode[K, V]
	size  int
}

// 默认的初始化容量
const INIT_CAP = 4

func NewMyLinearProbingHashMap1[K comparable, V any]() *MyLinearProbingHashMap1[K, V] {
	return NewMyLinearProbingHashMap1WithCapacity[K, V](INIT_CAP)
}

func NewMyLinearProbingHashMap1WithCapacity[K comparable, V any](initCapacity int) *MyLinearProbingHashMap1[K, V] {
	return &MyLinearProbingHashMap1[K, V]{
		table: make([]*KVNode[K, V], initCapacity),
	}
}

// **** 增/改 ****
func (m *MyLinearProbingHashMap1[K, V]) Put(key K, val V) error {
	// 负载因子默认设为 0.75，超过则扩容
	if m.size >= len(m.table)*3/4 {
		m.resize(len(m.table) * 2)
	}

	index := m.getKeyIndex(key)
	// key 已存在，修改对应的 val
	if m.table[index] != nil && m.table[index].key == key {
		m.table[index].val = val
		return nil
	}

	// key 不存在，在空位插入
	m.table[index] = &KVNode[K, V]{key, val}
	m.size++
	return nil
}

// **** 删 ****
func (m *MyLinearProbingHashMap1[K, V]) Remove(key K) error {
	// 缩容，当负载因子小于 0.125 时，缩容
	if m.size <= len(m.table)/8 {
		m.resize(len(m.table) / 2)
	}

	index := m.getKeyIndex(key)

	if m.table[index] == nil {
		// key 不存在，不需要 remove
		return nil
	}

	// 开始 remove
	m.table[index] = nil
	m.size--
	// 保持元素连续性，进行 rehash
	index = (index + 1) % len(m.table)
	for m.table[index] != nil {
		entry := m.table[index]
		m.table[index] = nil
		m.size--
		m.Put(entry.key, entry.val)
		index = (index + 1) % len(m.table)
	}
	return nil
}

// **** 查 ****
func (m *MyLinearProbingHashMap1[K, V]) Get(key K) (V, error) {
	index := m.getKeyIndex(key)
	if m.table[index] == nil {
		return *new(V), nil
	}
	return m.table[index].val, nil
}

func (m *MyLinearProbingHashMap1[K, V]) ContainsKey(key K) bool {
	index := m.getKeyIndex(key)
	return m.table[index] != nil
}

// 返回所有 key（顺序不固定）
func (m *MyLinearProbingHashMap1[K, V]) Keys() []K {
	keys := []K{}
	for _, entry := range m.table {
		if entry != nil {
			keys = append(keys, entry.key)
		}
	}
	return keys
}

// **** 其他工具函数 ****
func (m *MyLinearProbingHashMap1[K, V]) Size() int {
	return m.size
}

// 哈希函数，将键映射到 table 的索引
func (m *MyLinearProbingHashMap1[K, V]) hash(key K) int {
	h := fnv.New32a()
	h.Write([]byte(fmt.Sprintf("%v", key)))
	return int(h.Sum32()) & 0x7fffffff % len(m.table)
}

// 对 key 进行线性探查，返回一个索引
// 如果 key 不存在，返回的就是下一个为 null 的索引，可用于插入
func (m *MyLinearProbingHashMap1[K, V]) getKeyIndex(key K) int {
	index := m.hash(key)
	for m.table[index] != nil {
		if m.table[index].key == key {
			return index
		}
		index = (index + 1) % len(m.table)
	}
	return index
}

func (m *MyLinearProbingHashMap1[K, V]) resize(newCap int) {
	newMap := NewMyLinearProbingHashMap1WithCapacity[K, V](newCap)
	for _, entry := range m.table {
		if entry != nil {
			newMap.Put(entry.key, entry.val)
		}
	}
	m.table = newMap.table
}

func main() {
	mapInstance := NewMyLinearProbingHashMap1[int, int]()
	mapInstance.Put(1, 1)
	mapInstance.Put(2, 2)
	mapInstance.Put(10, 10)
	mapInstance.Put(20, 20)
	mapInstance.Put(30, 30)
	mapInstance.Put(3, 3)

	val, _ := mapInstance.Get(1)
	fmt.Println(val) // 1
	val, _ = mapInstance.Get(2)
	fmt.Println(val) // 2
	val, _ = mapInstance.Get(20)
	fmt.Println(val) // 20

	mapInstance.Put(1, 100)
	val, _ = mapInstance.Get(1)
	fmt.Println(val) // 100

	mapInstance.Remove(20)
	fmt.Println(mapInstance.ContainsKey(20)) // false
	val, _ = mapInstance.Get(30)
	fmt.Println(val) // 30
}
```
