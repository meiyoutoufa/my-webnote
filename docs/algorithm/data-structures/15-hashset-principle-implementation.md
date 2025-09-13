---
title: 哈希集合的原理及代码实现
date: 2023-09-01
author: mikez
description: 哈希集合的原理及代码实现
weight: 15
tags:
  - 数据结构的基础
---

我讲解前面每种数据结构时，都会把原理和代码实现分到两篇文章里讲解，而这里讲哈希集合时，把原理和实现同时放在本文讲解，且本章节只有本文一篇文章，你有没有觉得奇怪？

哈哈，因为哈希集合没什么好讲的，它就是把前文讲的哈希表简单封装了一下：哈希表的键，其实就是哈希集合。

这么一句话就可以讲完了，不过我们还是稍微具体讲一下，照顾一下哈希集合的面子。

## 哈希集合原理

哈希集合的主要使用场景是「去重」，因为它的特性是：不会出现重复元素，可以在 O(1) 的时间增删元素，可以在 O(1) 的时间判断一个元素是否存在。

哈希集合的主要 API 如下：

```go
type HashSet struct {}

// 增，向哈希集合中添加一个元素，复杂度 O(1)
// 如果元素已存在，则什么都不会发生
func (hs *HashSet) Add(key int) {}

// 删，从哈希集合中移除一个元素，复杂度 O(1)
// 如果元素不存在，则什么都不会发生
func (hs *HashSet) Remove(key int) {}

// 查，判断元素是否存在，复杂度 O(1)
func (hs *HashSet) Contains(key int) bool {}
```

这几个 API 咋实现？其实根本用不着实现，直接把前文教给你的哈希表拿来用就行了。

你往哈希表里插入一个键值对 `put(key, value)`，时间复杂度是 O(1)；你往哈希表里删除一个键值对 remove(key)，时间复杂度是 O(1)；判断一个键是否存在哈希表中，就是判断 `get(key)` 是否得到空指针 `null`，所以时间复杂度也是 O(1)。

**综上，这几个哈希集合的 API 可以直接复用哈希表的 API 来实现。操作哈希集合，其实就是操作哈希表的键**。

那就有读者问了，哈希表里面的值呢，如何处理？答案是不处理，直接忽略掉就行了，我们可以用一个占位符来填充值，等会看代码实现你就懂了。

虽然只是简单封装了哈希表，但是大部分高级编程语言还是提供哈希集合的数据结构的，比如 Python 的 `set`，Java 的 `HashSet`，C++ 的 `unordered_set` 等等。但是像 Go 语言，标准库干脆就不提供哈希集合这种数据结构，你要用的话需要自己用哈希表 map 来模拟。

```md
因为哈希集合的元素就是哈希表里面的键，所以**哈希集合和哈希表具有相同的限制**，比如不能依赖哈希集合的元素遍历顺序、哈希集合中的元素应该是不可变的等等，具体限制和原因可以回顾前文
在前面的 [哈希表核心原理](./11-hashtable-core-principles.md) 一文中，我们介绍了哈希表的基本原理和实现思路。
```

## 哈希集合代码实现

有了上面的铺垫，代码实现应该没有任何难度了。因为太简单了，我就只给出一个 Java 实现，其他语言可以自行使用标准库提供的哈希表，或者我们前文动手实现的哈希表来封装哈希集合：

```go
package main

type MyHashSet[K comparable] struct {
	// 底层 map，用于存储集合元素，值用空结构体作为占位符
	data map[K]struct{}
}

func NewMyHashSet[K comparable]() *MyHashSet[K] {
	return &MyHashSet[K]{
		data: make(map[K]struct{}),
	}
}

func (s *MyHashSet[K]) Add(key K) {
	// 向哈希表添加一个键值对，值用空结构体作为占位符
	s.data[key] = struct{}{}
}


func (s *MyHashSet[K]) Remove(key K) {
	// 从哈希表中移除键 key
	delete(s.data, key)
}

func (s *MyHashSet[K]) Contains(key K) bool {
	// 判断哈希表中是否包含键 key
	_, exists := s.data[key]
	return exists
}

func (s *MyHashSet[K]) Size() int {
	return len(s.data)
}
```
