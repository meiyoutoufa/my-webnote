---
title: 环形数组技巧及实现
date: 2023-09-01
author: mikez
description: 环形数组技巧及实现
weight: 5
tags:
  - 数据结构的基础
---

**一句话总结**

环形数组技巧利用求模（余数）运算，将普通数组变成逻辑上的环形数组，可以让我们用 O(1) 的时间在数组头部增删元素。

## 环形数组原理

数组可能是环形的么？不可能。数组就是一块线性连续的内存空间，怎么可能有环的概念？

但是，我们可以在「逻辑上」把数组变成环形的，比如下面这段代码：

```go
// 长度为 5 的数组
var arr = [1, 2, 3, 4, 5];
var i = 0;
// 模拟环形数组，这个循环永远不会结束
while (i < arr.length) {
    console.log(arr[i]);
    i = (i + 1) % arr.length;
}
```

这段代码的关键在于求模运算 `%`，也就是求余数。当 i 到达数组末尾元素时，`i + 1` 和 `arr.length` 取余数又会变成 0，即会回到数组头部，这样就在逻辑上形成了一个环形数组，永远遍历不完。

这就是环形数组技巧。这个技巧如何帮助我们在 O(1) 的时间在数组头部增删元素呢？

是这样，假设我们现在有一个长度为 6 的数组，现在其中只装了 3 个元素，如下（未装元素的位置用 \_ 标识）：

```java
[1, 2, 3, _, _, _]
```

现在我们要在数组头部删除元素 1，那么我们可以把数组变成这样：

```java
[_, 2, 3, _, _, _]
```

即，我们仅仅把元素 `1` 的位置标记为空，但并不做数据搬移。

此时，如果我们要在数组头部增加元素 `4` 和元素 `5`，我们可以把数组变成这样：

```java
[4, 2, 3, _, _, 5]
```

你可以看到，当头部没有位置添加新元素时，它转了一圈，把新元素加到尾部了。

**核心原理**

```text
上面只是让大家对环形数组有一个直观地印象，环形数组的关键在于，它维护了两个指针 start 和 end，start 指向第一个有效元素的索引，end 指向最后一个有效元素的下一个位置索引。

这样，当我们在数组头部添加或删除元素时，只需要移动 start 索引，而在数组尾部添加或删除元素时，只需要移动 end 索引。

当 start, end 移动超出数组边界（< 0 或 >= arr.length）时，我们可以通过求模运算 % 让它们转一圈到数组头部或尾部继续工作，这样就实现了环形数组的效果。
```

## 动手环节

纸上得来终觉浅，绝知此事要躬行。

## 代码实现

**关键点、注意开闭区间**

```text
在我的代码中，环形数组的区间被定义为左闭右开的，即 [start, end) 区间包含数组元素。所以其他的方法都是以左闭右开区间为基础实现的。

那么肯定就会有读者问，为啥要左闭右开，我就是想两端都开，或者两端都闭，不行么？

在 滑动窗口算法核心框架 中定义滑动窗口的边界时也会有类似的问题，这里我也解释一下。

理论上，你可以随意设计区间的开闭，但一般设计为左闭右开区间是最方便处理的。

因为这样初始化 start = end = 0 时，区间 [0, 0) 中没有元素，但只要让 end 向右移动（扩大）一位，区间 [0, 1) 就包含一个元素 0 了。

如果你设置为两端都开的区间，那么让 end 向右移动一位后开区间 (0, 1) 仍然没有元素；如果你设置为两端都闭的区间，那么初始区间 [0, 0] 就已经包含了一个元素。这两种情况都会给边界处理带来不必要的麻烦，如果你非要使用的话，需要在代码中做一些特殊处理。
```

最后，请看代码实现：

```go
class CycleArray {
    constructor(size = 1) {
        this.size = size;
        this.arr = new Array(size);
        // start 指向第一个有效元素的索引，闭区间
        this.start = 0;
        // 切记 end 是一个开区间，
        // 即 end 指向最后一个有效元素的下一个位置索引
        this.end = 0;
        this.count = 0;
    }

    resize(newSize) {
        // 创建新的数组
        var newArr = new Array(newSize);
        // 将旧数组的元素复制到新数组中
        for (var i = 0; i < this.count; i++) {
            newArr[i] = this.arr[(this.start + i) % this.size];
        }
        this.arr = newArr;
        // 重置 start 和 end 指针
        this.start = 0;
        this.end = this.count;
        this.size = newSize;
    }

    // 在数组头部添加元素，时间复杂度 O(1)
    addFirst(val) {
        // 当数组满时，扩容为原来的两倍
        if (this.isFull()) {
            this.resize(this.size * 2);
        }
        // 因为 start 是闭区间，所以先左移，再赋值
        this.start = (this.start - 1 + this.size) % this.size;
        this.arr[this.start] = val;
        this.count++;
    }

    // 删除数组头部元素，时间复杂度 O(1)
    removeFirst() {
        if (this.isEmpty()) {
            throw new Error("Array is empty");
        }
        // 因为 start 是闭区间，所以先赋值，再右移
        this.arr[this.start] = null;
        this.start = (this.start + 1) % this.size;
        this.count--;
        // 如果数组元素数量减少到原大小的四分之一，则减小数组大小为一半
        if (this.count > 0 && this.count == this.size / 4) {
            this.resize(this.size / 2);
        }
    }

    // 在数组尾部添加元素，时间复杂度 O(1)
    addLast(val) {
        if (this.isFull()) {
            this.resize(this.size * 2);
        }
        // 因为 end 是开区间，所以是先赋值，再右移
        this.arr[this.end] = val;
        this.end = (this.end + 1) % this.size;
        this.count++;
    }

    // 删除数组尾部元素，时间复杂度 O(1)
    removeLast() {
        if (this.isEmpty()) {
            throw new Error("Array is empty");
        }
        // 因为 end 是开区间，所以先左移，再赋值
        this.end = (this.end - 1 + this.size) % this.size;
        this.arr[this.end] = null;
        this.count--;
        // 缩容
        if (this.count > 0 && this.count == this.size / 4) {
            this.resize(this.size / 2);
        }
    }

    // 获取数组头部元素，时间复杂度 O(1)
    getFirst() {
        if (this.isEmpty()) {
            throw new Error("Array is empty");
        }
        return this.arr[this.start];
    }

    // 获取数组尾部元素，时间复杂度 O(1)
    getLast() {
        if (this.isEmpty()) {
            throw new Error("Array is empty");
        }
        // end 是开区间，指向的是下一个元素的位置，所以要减 1
        return this.arr[(this.end - 1 + this.size) % this.size];
    }

    isFull() {
        return this.count === this.size;
    }

    size() {
        return this.count;
    }

    isEmpty() {
        return this.count === 0;
    }
}
```

## 思考题

数组增删头部元素的效率真的只能是 O(N) 么？

我们都说，在数组增删头部元素的时间复杂度是 O(N)，因为需要搬移元素。但是，如果我们使用环形数组，其实是可以实现在 O(1) 的时间复杂度内增删头部元素的。

当然，上面实现的这个环形数组只提供了 `addFirst`, `removeFirst`, `addLast`, `removeLast` 这几个方法，并没有提供
我们之前实现的动态数组 的某些方法，比如删除指定索引的元素，获取指定索引的元素，在指定索引插入元素等等。

但是你可以思考一下，难道环形数组实现不了这些方法么？环形数组实现这些方法，时间复杂度相比普通数组，有退化吗？

好像没有吧。

环形数组也可以删除指定索引的元素，也要做数据搬移，和普通数组一样，复杂度是 O(N)；

环形数组也可以获取指定索引的元素（随机访问），只不过不是直接访问对应索引，而是要通过 start 计算出真实索引，但计算和访问的时间复杂度依然是 O(1)；

环形数组也可以在指定索引插入元素，当然也要做数据搬移，和普通数组一样，复杂度是 O(N)。

你可以思考一下是不是这样。如果是这样，为什么编程语言的标准库中提供的动态数组容器底层并没有用环形数组技巧。
