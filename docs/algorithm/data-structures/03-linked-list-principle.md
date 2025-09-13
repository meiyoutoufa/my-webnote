---
title: 链表（链式存储）基本原理
date: 2023-09-01
author: mikez
description: 链表（链式存储）基本原理
weight: 3
tags:
  - 数据结构的基础
---

刷过力扣的读者肯定对单链表非常熟悉，力扣上的单链表节点定义如下：

```go

type ListNode struct {
    Val  int
    Next *ListNode
}
```

这仅仅是一个最简单的`单链表节点`，方便力扣出算法题来考你。在实际的编程语言中，我们使用的链表节点会稍微复杂一点，类似这样：

```go
type Node[T any] struct {
    val  T
    next *Node[T]
    prev *Node[T]
}

func NewNode[T any](prev *Node[T], element T, next *Node[T]) *Node[T] {
    return &Node[T]{
        val:  element,
        next: next,
        prev: prev,
    }
}
```

主要区别有两个：

1、编程语言标准库一般都会提供泛型，即你可以指定 val 字段为任意类型，而力扣的单链表节点的 val 字段只有 int 类型。

2、编程语言标准库一般使用的都是双链表而非单链表。单链表节点只有一个 next 指针，指向下一个节点；而双链表节点有两个指针，`prev` 指向前一个节点，`next` 指向下一个节点。

有了 `prev` 前驱指针，链表支持双向遍历，但由于要多维护一个指针，增删查改时会稍微复杂一些，后面带大家实现双链表时会具体介绍。

### 为什么需要链表

前面介绍了 [数组（顺序存储）的底层原理](./01-array-sequential-storage.md)，说白了就是一块连续的内存空间，有了这块内存空间的首地址，就能直接通过索引计算出任意位置的元素地址。

链表不一样，一条链表并不需要一整块连续的内存空间存储元素。链表的元素可以分散在内存空间的天涯海角，通过每个节点上的 `next`, `prev` 指针，将零散的内存块串联起来形成一个链式结构。

这样做的好处很明显，首先就是可以提高内存的利用效率，链表的节点不需要挨在一起，给点内存 new 出来一个节点就能用，操作系统会觉得这娃好养活。

另外一个好处，它的节点要用的时候就能接上，不用的时候拆掉就行了，从来不需要考虑扩缩容和数据搬移的问题，理论上讲，链表是没有容量限制的（除非把所有内存都占满，这不太可能）。

当然，不可能只有好处没有局限性。数组最大的优势是支持通过索引快速访问元素，而链表就不支持。

这个不难理解吧，因为元素并不是紧挨着的，所以如果你想要访问第 3 个链表元素，你就只能从头结点开始往顺着 next 指针往后找，直到找到第 3 个节点才行。

上面是对链表这种数据结构的基本介绍，接下来我们就结合代码实现单/双链表的几个基本操作。

### 单链表的基本操作

我先写一个工具函数，用于创建一条单链表，方便后面的讲解：

```go
type ListNode struct {
    Val int
    Next *ListNode
}

// 输入一个数组，转换为一条单链表
func createLinkedList(arr []int) *ListNode {
    if arr == nil || len(arr) == 0 {
        return nil
    }
    head := &ListNode{Val: arr[0]}
    cur := head
    for i := 1; i < len(arr); i++ {
        cur.Next = &ListNode{Val: arr[i]}
        cur = cur.Next
    }
    return head
}
```

#### 查/改

**单链表的遍历/查找/修改**

比方说，我想访问单链表的每一个节点，并打印其值，可以这样写：

```go
// 创建一条单链表
head := createLinkedList([]int{1, 2, 3, 4, 5})

// 遍历单链表
for p := head; p != nil; p = p.Next {
fmt.Println(p.Val)
}
```

类似的，如果是要通过索引访问或修改链表中的某个节点，也只能用 for 循环从头结点开始往后找，直到找到索引对应的节点，然后进行访问或修改。

#### 增

**在单链表头部插入新元素**

我们会持有单链表的头结点，所以只需要将插入的节点接到头结点之前，并将新插入的节点作为头结点即可。

```go
// 创建一条单链表
head := createLinkedList([]int{1, 2, 3, 4, 5})

// 在单链表头部插入一个新节点 0
newNode := &ListNode{Val: 0}
newNode.Next = head
head = newNode

// 现在链表变成了 0 -> 1 -> 2 -> 3 -> 4 -> 5
```

**在单链表尾部插入新元素**

直接看代码吧，很简单：

```go
// 创建一条单链表
head := createLinkedList([]int{1, 2, 3, 4, 5})

// 在单链表尾部插入一个新节点 6
p := head
// 先走到链表的最后一个节点
for p.Next != nil {
    p = p.Next
}
// 现在 p 就是链表的最后一个节点
// 在 p 后面插入新节点
p.Next = &ListNode{Val: 6}

// 现在链表变成了 1 -> 2 -> 3 -> 4 -> 5 -> 6
```

当然，如果我们持有对链表尾节点的引用，那么在尾部插入新节点的操作就会变得非常简单，不用每次从头去遍历了。这个优化会在后面具体实现双链表时介绍。

**在单链表中间插入新元素**

这个操作稍微有点复杂，我们还是要先找到要插入位置的前驱节点，然后操作前驱节点把新节点插入进去：

```go
// 创建一条单链表
head := createLinkedList([]int{1, 2, 3, 4, 5})

// 在第 3 个节点后面插入一个新节点 66
// 先要找到前驱节点，即第 3 个节点
p := head
for i := 0; i < 2; i++ {
    p = p.next
}
// 此时 p 指向第 3 个节点
// 组装新节点的后驱指针
newNode := &ListNode{Val: 66}
newNode.next = p.next

// 插入新节点
p.next = newNode

// 现在链表变成了 1 -> 2 -> 3 -> 66 -> 4 -> 5
```

#### 删

**在单链表中删除一个节点**

删除一个节点，首先要找到要被删除节点的前驱节点，然后把这个前驱节点的 next 指针指向被删除节点的下一个节点。这样就能把被删除节点从链表中摘除了。

```go
// 创建一条单链表
head := createLinkedList([]int{1, 2, 3, 4, 5})

// 删除第 4 个节点，要操作前驱节点
p := head
for i := 0; i < 2; i++ {
    p = p.next
}

// 此时 p 指向第 3 个节点，即要删除节点的前驱节点
// 把第 4 个节点从链表中摘除
p.next = p.next.next

// 现在链表变成了 1 -> 2 -> 3 -> 5
```

**在单链表尾部删除元素**

这个操作比较简单，找到倒数第二个节点，然后把它的 next 指针置为 null 就行了：

```go
// 创建一条单链表
head := createLinkedList([]int{1, 2, 3, 4, 5})

// 删除尾节点
p := head
// 找到倒数第二个节点
for p.next.next != nil {
    p = p.next
}

// 此时 p 指向倒数第二个节点
// 把尾节点从链表中摘除
p.next = nil

// 现在链表变成了 1 -> 2 -> 3 -> 4
```

**在单链表头部删除元素**

这个操作比较简单，直接把 head 移动到下一个节点就行了，直接看代码吧：

```go
// 创建一条单链表
head := createLinkedList([]int{1, 2, 3, 4, 5})

// 删除头结点
head = head.Next

// 现在链表变成了 2 -> 3 -> 4 -> 5
```

不过可能有读者疑惑，之前那个旧的头结点 `1` 的 next 指针依然指向着节点 `2`，这样会不会造成内存泄漏？

不会的，这个节点 `1` 指向其他的节点是没关系的，只要保证没有其他引用指向这个节点 `1`，它就能被垃圾回收器回收掉。

当然，如果你非要显式把节点 `1` 的 next 指针置为 null，这是个很好的习惯，在其他场景中可能可以避免指针错乱的潜在问题。

在下面这个可视化面板中，我显式地把待删除节点的 next 指针置为 null 了：

```text
是不是觉得复杂？

链表的增删查改操作确实比数组复杂。这是因为链表的节点不是紧挨着的，所以要增删一个节点，必须先找到它的前驱和后驱节点进行协同，然后才能通过指针操作把它插入或删除。

上面给出的代码还仅仅是最简单的例子，你会发现在头部、尾部、中间增删元素的代码都不一样。如果要实现一个真正可用的链表，你还要考虑到很多边界情况，比如链表可能为空、前后驱节点可能为空等，这些情况都得保证不出错。

而且，上面只是介绍了「单链表」，而我们下一章要实现的是「双链表」，双链表要同时维护前驱和后驱指针，指针操作会更复杂一些。

是不是已经不敢想了？不要怕，其实没你想的那么难，几个原因：

1、其实搞来搞去就那几个操作，等会儿带你动手实现链表 API 的时候，你亲自写一写，就会了。

2、复杂操作我都配了可视化面板，你可以结合面板中的代码和动画进行理解。

3、最重要的，我们会使用「虚拟头结点」技巧，把头、尾、中部的操作统一起来，同时还能避免处理头尾指针为空情况的边界情况。

虚拟节点技巧在 单链表经典算法技巧 中也会经常运用，这里仅仅简单提一下，具体实现会在后面讲到。
```

### 双链表的基本操作

我先写一个工具函数，用于创建一条双链表，方便后面的讲解：

```go
type DoublyListNode struct {
    Val        int
    Prev, Next *DoublyListNode
}

func NewDoublyListNode(x int) *DoublyListNode {
    return &DoublyListNode{Val: x}
}

func CreateDoublyLinkedList(arr []int) *DoublyListNode {
    if arr == nil || len(arr) == 0 {
        return nil
    }
    head := NewDoublyListNode(arr[0])
    cur := head
    // for 循环迭代创建双链表
    for i := 1; i < len(arr); i++ {
        newNode := NewDoublyListNode(arr[i])
        cur.Next = newNode
        newNode.Prev = cur
        cur = cur.Next
    }
    return head
}
```

#### 查/改

**双链表的遍历/查找/修改**

对于双链表的遍历和查找，我们可以从头节点或尾节点开始，根据需要向前或向后遍历：

```go
// 创建一条双链表
head := createDoublyLinkedList([]int{1, 2, 3, 4, 5})
var tail *DoublyListNode

// 从头节点向后遍历双链表
for p := head; p != nil; p = p.Next {
    fmt.Println(p.Val)
    tail = p
}

// 从尾节点向前遍历双链表
for p := tail; p != nil; p = p.Prev {
    fmt.Println(p.Val)
}

```

访问或修改节点时，可以根据索引是靠近头部还是尾部，选择合适的方向遍历，这样可以一定程度上提高效率。

#### 增

**在双链表头部插入新元素**

在双链表头部插入元素，需要调整新节点和原头节点的指针：

```go
// 创建一条双链表
head := createDoublyLinkedList([]int{1, 2, 3, 4, 5})

// 在双链表头部插入新节点 0
newHead := &DoublyListNode{val: 0}
newHead.next = head
head.prev = newHead
head = newHead
// 现在链表变成了 0 -> 1 -> 2 -> 3 -> 4 -> 5
```

**在双链表尾部插入新元素**

在双链表尾部插入元素时，如果我们持有尾节点的引用，这个操作会非常简单：

```go
// 创建一条双链表
head := createDoublyLinkedList([]int{1, 2, 3, 4, 5})

tail := head
// 先走到链表的最后一个节点
for tail.next != nil {
    tail = tail.next
}

// 在双链表尾部插入新节点 6
newNode := &DoublyListNode{value: 6}
tail.next = newNode
newNode.prev = tail
// 更新尾节点引用
tail = newNode

// 现在链表变成了 1 -> 2 -> 3 -> 4 -> 5 -> 6
```

**在双链表中间插入新元素**

在双链表的指定位置插入新元素，需要调整前驱节点和后继节点的指针。

比如下面的例子，把元素 66 插入到索引 3（第 4 个节点）的位置：

```go
// 创建一条双链表
head := createDoublyLinkedList([]int{1, 2, 3, 4, 5})

// 想要插入到索引 3（第 4 个节点）
// 需要操作索引 2（第 3 个节点）的指针
p := head
for i := 0; i < 2; i++ {
    p = p.next
}

// 组装新节点
newNode := &DoublyListNode{val: 66}
newNode.next = p.next
newNode.prev = p

// 插入新节点
p.next.prev = newNode
p.next = newNode

// 现在链表变成了 1 -> 2 -> 3 -> 66 -> 4 -> 5
```

#### 删

**在双链表中删除一个节点**

在双链表中删除节点时，需要调整前驱节点和后继节点的指针来摘除目标节点：

```go
// 创建一个双链表
head := createDoublyLinkedList([]int{1, 2, 3, 4, 5})

// 删除第 4 个节点
// 先找到第 3 个节点
p := head
for i := 0; i < 2; i++ {
    p = p.Next
}

// 现在 p 指向第 3 个节点，我们将它后面那个节点摘除出去
toDelete := p.Next

// 把 toDelete 从链表中摘除
p.Next = toDelete.Next
if toDelete.Next != nil {
    toDelete.Next.Prev = p
}

// 把 toDelete 的前后指针都置为 nil 是个好习惯（可选）
toDelete.Next = nil
toDelete.Prev = nil

// 现在链表变成了 1 -> 2 -> 3 -> 5
```

**在双链表头部删除元素**

在双链表头部删除元素需要调整头节点的指针：

```go
// 创建一条双链表
head := createDoublyLinkedList([]int{1, 2, 3, 4, 5})

// 删除头结点
toDelete := head
head = head.next
head.prev = nil

// 清理已删除节点的指针
toDelete.next = nil

// 现在链表变成了 2 -> 3 -> 4 -> 5
```

**在双链表尾部删除元素**

在单链表中，由于缺乏前驱指针，所以删除尾节点时需要遍历到倒数第二个节点，操作它的 next 指针，才能把尾节点摘除出去。

但在双链表中，由于每个节点都存储了前驱节点的指针，所以我们可以直接操作尾节点，把它自己从链表中摘除：

```go
// 创建一条双链表
head := createDoublyLinkedList([]int{1, 2, 3, 4, 5})

// 删除尾节点
p := head
// 找到尾结点
for p.next != nil {
    p = p.next
}

// 现在 p 指向尾节点
// 把尾节点从链表中摘除
p.prev.next = nil

// 把被删结点的指针都断开是个好习惯（可选）
p.prev = nil

// 现在链表变成了 1 -> 2 -> 3 -> 4
```

接下来
在下一篇文章中，我们分别用单链表和双链表实现一个拥有增删查改等基本操作的 MyLinkedList，并且会使用「虚拟头结点」技巧简化代码逻辑，避免处理头尾指针为空情况的边界情况。
