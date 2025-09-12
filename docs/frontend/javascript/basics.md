---
sidebar_position: 1
---

# JavaScript 基础

JavaScript 是网页的交互逻辑语言。

## 变量和数据类型

### 变量声明

```javascript
// ES6+ 推荐使用 let 和 const
let name = "张三";        // 可变变量
const age = 25;          // 常量
var oldStyle = "旧语法";  // 不推荐使用
```

### 数据类型

```javascript
// 基本数据类型
let str = "字符串";       // String
let num = 42;            // Number
let bool = true;         // Boolean
let nothing = null;      // Null
let undef = undefined;   // Undefined

// 引用数据类型
let arr = [1, 2, 3];     // Array
let obj = {name: "张三"}; // Object
```

## 函数

### 函数声明

```javascript
// 传统函数声明
function greet(name) {
    return "你好, " + name + "!";
}

// 箭头函数（ES6+）
const greet2 = (name) => {
    return `你好, ${name}!`;
};

// 简化的箭头函数
const greet3 = name => `你好, ${name}!`;
```

## DOM 操作

```javascript
// 获取元素
const element = document.getElementById('myId');
const elements = document.querySelectorAll('.myClass');

// 修改内容
element.textContent = '新内容';
element.innerHTML = '<strong>HTML内容</strong>';

// 添加事件监听
element.addEventListener('click', function() {
    alert('按钮被点击了!');
});
```

## 实践项目

创建一个简单的计算器：
- 基本的加减乘除功能
- 清除和重置功能
- 响应式设计