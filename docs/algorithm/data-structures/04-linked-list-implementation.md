---
title: 链表代码实现
date: 2023-09-01
author: mikez
description: 链表代码实现
weight: 4
tags:
  - 数据结构的基础
---

### 几个关键点

下面我会分别用双链表和单链给出一个简单的 MyLinkedList 代码实现，包含了基本的增删查改功能。这里给出几个关键点，等会你看代码的时候可以着重注意一下。

#### 关键点一、同时持有头尾节点的引用

在力扣做题时，一般题目给我们传入的就是单链表的头指针。但是在实际开发中，用的都是双链表，而双链表一般会同时持有头尾节点的引用。

因为在软件开发中，在容器尾部添加元素是个非常高频的操作，双链表持有尾部节点的引用，就可以在 O(1) 的时间复杂度内完成尾部添加元素的操作。

对于单链表来说，持有尾部节点的引用也有优化效果。比如你要在单链表尾部添加元素，如果没有尾部节点的引用，你就需要遍历整个链表找到尾部节点，时间复杂度是 O(n)；如果有尾部节点的引用，就可以在
O(1) 的时间复杂度内完成尾部添加元素的操作。

细心的读者可能会说，即便如此，如果删除一次单链表的尾结点，那么之前尾结点的引用就失效了，还是需要遍历一遍链表找到尾结点。

是的，但你再仔细想想，删除单链表尾结点的时候，是不是也得遍历到倒数第二个节点（尾结点的前驱），才能通过指针操作把尾结点删掉？那么这个时候，你不就可以顺便把尾结点的引用给更新了吗？

#### 关键点二、虚拟头尾节点

在上一篇文章 链表基础 中我提到过「虚拟头尾节点」技巧，它的原理很简单，就是在创建双链表时就创建一个虚拟头节点和一个虚拟尾节点，无论双链表是否为空，这两个节点都存在。这样就不会出现空指针的问题，可以避免很多边界情况的处理。

举例来说，假设虚拟头尾节点分别是 `dummyHead` 和 `dummyTail`，那么一条空的双链表长这样：

```text
dummyHead <-> dummyTail
```

如果你添加 `1,2,3` 几个元素，那么链表长这样：

```text
dummyHead <-> 1 <-> 2 <-> 3 <-> dummyTail
```

你以前要把在头部插入元素、在尾部插入元素和在中间插入元素几种情况分开讨论，现在有了头尾虚拟节点，无论链表是否为空，都只需要考虑在中间插入元素的情况就可以了，这样代码会简洁很多。

当然，虚拟头结点会多占用一点内存空间，但是比起给你解决的麻烦，这点空间消耗是划算的。

对于单链表，虚拟头结点有一定的简化作用，但虚拟尾节点没有太大作用。

```text
虚拟节点是内部实现，对外不可见

虚拟节点是你内部实现数据结构的技巧，对外是不可见的。比如按照索引获取元素的 get(index) 方法，都是从真实节点开始计算索引，而不是从虚拟节点开始计算。
```

#### 关键点三、内存泄露？

在前文 动态数组实现 中，我提到了删除元素时，要注意内存泄露的问题。那么在链表中，删除元素会不会也有内存泄露的问题呢？

尤其是这样的写法，你觉得有没有问题：

```java
// 假设单链表头结点 head = 1 -> 2 -> 3 -> 4 -> 5

// 删除单链表头结点
head = head.next;

// 此时 head = 2 -> 3 -> 4 -> 5
```

细心的读者可能认为这样写会有内存泄露的问题，因为原来的那个头结点 `1` 的 `next` 指针没有断开，依然指向着节点 `2`。

但实际上这样写是 OK 的，因为 Java 的垃圾回收的判断机制是看这个对象是否被别人引用，而并不会 care 这个对象是否还引用着别人。

那个节点 `1` 的 `next` 指针确实还指向着节点 `2`，但是并没有别的指针引用节点 `1` 了，所以节点 `1` 最终会被垃圾回收器回收释放。所以说这个场景和数组中删除元素的场景是不一样的，你可以再仔细思考一下。

不过呢，删除节点时，最好还是把被删除节点的指针都置为 null，这是个好习惯，不会有什么代价，还可能避免一些潜在的问题。所以在下面的实现中，无论是否有必要，我都会把被删除节点上的指针置为 null。

```text
如何验证你的实现？

你可以借助力扣第 707 题「设计链表」来验证自己的实现是否正确。注意 707 题要求的增删查改 API 名字和本文给出的不一样，所以需要修改一下才能通过。
```

### 双链表代码实现

```go
package main

import (
	"errors"
	"fmt"
)

type Node struct {
	val  interface{}
	next *Node
	prev *Node
}

type MyLinkedList struct {
	head *Node
	tail *Node
	size int
}

// 虚拟头尾节点
func NewMyLinkedList() *MyLinkedList {
	head := &Node{}
	tail := &Node{}
	head.next = tail
	tail.prev = head
	return &MyLinkedList{head: head, tail: tail, size: 0}
}

// ***** 增 *****

func (list *MyLinkedList) AddLast(e interface{}) {
	x := &Node{val: e}
	temp := list.tail.prev
	// temp <-> x
	temp.next = x
	x.prev = temp

	x.next = list.tail
	list.tail.prev = x
	// temp <-> x <-> tail
	list.size++
}

func (list *MyLinkedList) AddFirst(e interface{}) {
	x := &Node{val: e}
	temp := list.head.next
	// head <-> temp
	temp.prev = x
	x.next = temp

	list.head.next = x
	x.prev = list.head
	// head <-> x <-> temp
	list.size++
}

func (list *MyLinkedList) Add(index int, element interface{}) error {
	if err := list.checkPositionIndex(index); err != nil {
		return err
	}
	if index == list.size {
		list.AddLast(element)
		return nil
	}

	// 找到 index 对应的 Node
	p := list.getNode(index)
	temp := p.prev
	// temp <-> p

	// 新要插入的 Node
	x := &Node{val: element}

	p.prev = x
	temp.next = x

	x.prev = temp
	x.next = p

	// temp <-> x <-> p

	list.size++
	return nil
}

// ***** 删 *****

func (list *MyLinkedList) RemoveFirst() (interface{}, error) {
	if list.size < 1 {
		return nil, errors.New("no elements to remove")
	}
	// 虚拟节点的存在是我们不用考虑空指针的问题
	x := list.head.next
	temp := x.next
	// head <-> x <-> temp
	list.head.next = temp
	temp.prev = list.head

	// head <-> temp

	list.size--
	return x.val, nil
}

func (list *MyLinkedList) RemoveLast() (interface{}, error) {
	if list.size < 1 {
		return nil, errors.New("no elements to remove")
	}
	x := list.tail.prev
	temp := x.prev
	// temp <-> x <-> tail

	list.tail.prev = temp
	temp.next = list.tail

	// temp <-> tail

	list.size--
	return x.val, nil
}

func (list *MyLinkedList) Remove(index int) (interface{}, error) {
	if err := list.checkElementIndex(index); err != nil {
		return nil, err
	}
	// 找到 index 对应的 Node
	x := list.getNode(index)
	prev := x.prev
	next := x.next
	// prev <-> x <-> next
	prev.next = next
	next.prev = prev

	list.size--

	return x.val, nil
}

// ***** 查 *****

func (list *MyLinkedList) Get(index int) (interface{}, error) {
	if err := list.checkElementIndex(index); err != nil {
		return nil, err
	}
	// 找到 index 对应的 Node
	p := list.getNode(index)

	return p.val, nil
}

func (list *MyLinkedList) GetFirst() (interface{}, error) {
	if list.size < 1 {
		return nil, errors.New("no elements in the list")
	}

	return list.head.next.val, nil
}

func (list *MyLinkedList) GetLast() (interface{}, error) {
	if list.size < 1 {
		return nil, errors.New("no elements in the list")
	}

	return list.tail.prev.val, nil
}

// ***** 改 *****

func (list *MyLinkedList) Set(index int, val interface{}) (interface{}, error) {
	if err := list.checkElementIndex(index); err != nil {
		return nil, err
	}
	// 找到 index 对应的 Node
	p := list.getNode(index)

	oldVal := p.val
	p.val = val

	return oldVal, nil
}

// ***** 其他工具函数 *****

func (list *MyLinkedList) Size() int {
	return list.size
}

func (list *MyLinkedList) IsEmpty() bool {
	return list.size == 0
}

func (list *MyLinkedList) getNode(index int) *Node {
	p := list.head.next
	// TODO: 可以优化，通过 index 判断从 head 还是 tail 开始遍历
	for i := 0; i < index; i++ {
		p = p.next
	}
	return p
}

func (list *MyLinkedList) isElementIndex(index int) bool {
	return index >= 0 && index < list.size
}

func (list *MyLinkedList) isPositionIndex(index int) bool {
	return index >= 0 && index <= list.size
}

// 检查 index 索引位置是否可以存在元素
func (list *MyLinkedList) checkElementIndex(index int) error {
	if !list.isElementIndex(index) {
		return fmt.Errorf("Index: %d, Size: %d", index, list.size)
	}
	return nil
}

// 检查 index 索引位置是否可以添加元素
func (list *MyLinkedList) checkPositionIndex(index int) error {
	if !list.isPositionIndex(index) {
		return fmt.Errorf("Index: %d, Size: %d", index, list.size)
	}
	return nil
}

func (list *MyLinkedList) Display() {
	fmt.Printf("size = %d\n", list.size)
	p := list.head.next
	for p != list.tail {
		fmt.Printf("%v <-> ", p.val)
		p = p.next
	}
	fmt.Println("null\n")
}

func main() {
	list := NewMyLinkedList()
	list.AddLast(1)
	list.AddLast(2)
	list.AddLast(3)
	list.AddFirst(0)
	list.Add(2, 100)

	list.Display()
	// size = 5
	// 0 <-> 1 <-> 100 <-> 2 <-> 3 <-> null
}
```

### 单链表代码实现

```go
package main

import (
	"errors"
	"fmt"
)

// Node 节点结构
type Node[E any] struct {
	val  E
	next *Node[E]
}

// MyLinkedList2 链表结构
type MyLinkedList2[E any] struct {
	head *Node[E]
	// 实际的尾部节点引用
	tail  *Node[E]
	size_ int
}

// NewMyLinkedList2 创建一个新的链表
func NewMyLinkedList2[E any]() *MyLinkedList2[E] {
	head := &Node[E]{}
	return &MyLinkedList2[E]{head: head, tail: head, size_: 0}
}

// addFirst 在头部添加元素
func (list *MyLinkedList2[E]) AddFirst(e E) {
	newNode := &Node[E]{val: e}
	newNode.next = list.head.next
	list.head.next = newNode
	if list.size_ == 0 {
		list.tail = newNode
	}
	list.size_++
}

// addLast 在尾部添加元素
func (list *MyLinkedList2[E]) AddLast(e E) {
	newNode := &Node[E]{val: e}
	list.tail.next = newNode
	list.tail = newNode
	list.size_++
}

// add 在指定索引处添加元素
func (list *MyLinkedList2[E]) Add(index int, element E) error {
	if index < 0 || index > list.size_ {
		return errors.New("index out of bounds")
	}

	if index == list.size_ {
		list.AddLast(element)
		return nil
	}

	prev := list.head
	for i := 0; i < index; i++ {
		prev = prev.next
	}
	newNode := &Node[E]{val: element}
	newNode.next = prev.next
	prev.next = newNode
	list.size_++
	return nil
}

// removeFirst 移除头部元素
func (list *MyLinkedList2[E]) RemoveFirst() (E, error) {
	if list.IsEmpty() {
		return *new(E), errors.New("no elements to remove")
	}
	first := list.head.next
	list.head.next = first.next
	if list.size_ == 1 {
		list.tail = list.head
	}
	list.size_--
	return first.val, nil
}

// removeLast 移除尾部元素
func (list *MyLinkedList2[E]) RemoveLast() (E, error) {
	if list.IsEmpty() {
		return *new(E), errors.New("no elements to remove")
	}

	prev := list.head
	for prev.next != list.tail {
		prev = prev.next
	}
	val := list.tail.val
	prev.next = nil
	list.tail = prev
	list.size_--
	return val, nil
}

// remove 在指定索引处移除元素
func (list *MyLinkedList2[E]) Remove(index int) (E, error) {
	if index < 0 || index >= list.size_ {
		return *new(E), errors.New("index out of bounds")
	}

	prev := list.head
	for i := 0; i < index; i++ {
		prev = prev.next
	}

	nodeToRemove := prev.next
	prev.next = nodeToRemove.next
	// 删除的是最后一个元素
	if index == list.size_-1 {
		list.tail = prev
	}
	list.size_--
	return nodeToRemove.val, nil
}

// GetFirst 获取头部元素
func (list *MyLinkedList2[E]) GetFirst() (E, error) {
	if list.IsEmpty() {
		return *new(E), errors.New("no elements in the list")
	}
	return list.head.next.val, nil
}

// GetLast 获取尾部元素
func (list *MyLinkedList2[E]) GetLast() (E, error) {
	if list.IsEmpty() {
		return *new(E), errors.New("no elements in the list")
	}
	return list.tail.val, nil
}

// Get 获取指定索引的元素
func (list *MyLinkedList2[E]) Get(index int) (E, error) {
	if index < 0 || index >= list.size_ {
		return *new(E), errors.New("index out of bounds")
	}
	return list.getNode(index).val, nil
}

// Set 更新指定索引的元素
func (list *MyLinkedList2[E]) Set(index int, element E) (E, error) {
	if index < 0 || index >= list.size_ {
		return *new(E), errors.New("index out of bounds")
	}
	node := list.getNode(index)
	oldVal := node.val
	node.val = element
	return oldVal, nil
}

// Size 获取链表大小
func (list *MyLinkedList2[E]) Size() int {
	return list.size_
}

// IsEmpty 检查链表是否为空
func (list *MyLinkedList2[E]) IsEmpty() bool {
	return list.size_ == 0
}

// getNode 返回指定索引的节点
func (list *MyLinkedList2[E]) getNode(index int) *Node[E] {
	p := list.head.next
	for i := 0; i < index; i++ {
		p = p.next
	}
	return p
}

func main() {
	list := NewMyLinkedList2[int]()
	list.addFirst(1)
	list.addFirst(2)
	list.addLast(3)
	list.addLast(4)
	list.add(2, 5)

	if val, err := list.removeFirst(); err == nil {
		fmt.Println(val) // 2
	}
	if val, err := list.removeLast(); err == nil {
		fmt.Println(val) // 4
	}
	if val, err := list.remove(1); err == nil {
		fmt.Println(val) // 5
	}

	if val, err := list.getFirst(); err == nil {
		fmt.Println(val) // 1
	}
	if val, err := list.getLast(); err == nil {
		fmt.Println(val) // 3
	}
	if val, err := list.get(1); err == nil {
		fmt.Println(val) // 3
	}
}
```

### 【游戏】实现贪吃蛇

按照要求实现 move 方法，即可完成一个可运行的贪吃蛇游戏。

游戏引擎会每个一段时间调用一次 move 方法让蛇前进一个单元格，move 方法传入蛇身的所有单元格坐标（存储在一条双链表 LinkedList 中，链表头部是蛇头，尾部是蛇尾）、食物的坐标以及当前的方向，你需要原地修改这个 LinkedList 对象，根据当前方向更新蛇身坐标，完成移动。

#### 讲解

你仔细想想，蛇前进时，其实只有头结点和尾结点在移动，中间的蛇身节点根本没有移动。

所以，我们只需要根据当前方向，创建一个新的头结点插入链表头部，然后再删除链表尾部的节点，即可完成蛇的前进；当新的头结点遇到食物时，我们不删除尾结点，就相当于蛇身长长了一个单位，这样就完成了贪吃蛇的全部逻辑。

所以解法代码如下：

```go
// 游戏面板仅支持提交 JavaScript 代码
// Go 解法仅供参考，方便读者理解算法逻辑
func move(snake *[]Point, direction string, foodPosition Point) {
    currentHead := (*snake)[0]
    // 计算新头部位置
    newHeadPos := Point{X: currentHead.X, Y: currentHead.Y}
    switch direction {
    case "right":
        newHeadPos.X += 1
    case "left":
        newHeadPos.X -= 1
    case "up":
        newHeadPos.Y += 1
    case "down":
        newHeadPos.Y -= 1
    default:
        panic("Invalid direction")
    }
    // 添加新的头部节点
    *snake = append([]Point{newHeadPos}, *snake...)
    if !newHeadPos.Equals(foodPosition) {
        // 如果不是吃到食物，则删除尾部节点
        *snake = (*snake)[:len(*snake)-1]
    }
}
```

```javascript
var move = function (snake, direction, foodPosition) {
  const currentHead = snake[0];
  // 计算新头部位置
  const newHeadPos = new Point(currentHead.x, currentHead.y);
  switch (direction) {
    case "right":
      newHeadPos.x += 1;
      break;
    case "left":
      newHeadPos.x -= 1;
      break;
    case "up":
      newHeadPos.y += 1;
      break;
    case "down":
      newHeadPos.y -= 1;
      break;
    default:
      throw new Error("Invalid direction");
  }

  // 添加新的头部节点
  snake.unshift(newHeadPos);

  if (!newHeadPos.equals(foodPosition)) {
    // 如果不是吃到食物，则删除尾部节点
    snake.pop();
  }
};
```
