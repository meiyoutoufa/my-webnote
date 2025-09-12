---
sidebar_position: 1
---

# HTML & CSS 基础

## HTML 基础

HTML（HyperText Markup Language）是网页的骨架。

### 基本结构

```html title="index.html"
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>页面标题</title>
</head>
<body>
    <h1>这是一个标题</h1>
    <p>这是一个段落。</p>
</body>
</html>
```

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

<Tabs>
  <TabItem value="js" label="JavaScript">
    ```javascript
    console.log('Hello from JavaScript!');
    ```
  </TabItem>
  <TabItem value="py" label="Python">
    ```python
    print('Hello from Python!')
    ```
  </TabItem>
</Tabs>

### 常用标签

- `<h1>` - `<h6>`: 标题标签
- `<p>`: 段落标签
- `<div>`: 容器标签
- `<span>`: 行内容器
- `<a>`: 链接标签
- `<img>`: 图片标签

## CSS 基础

CSS（Cascading Style Sheets）负责网页的样式和布局。

### 基本语法

```css
选择器 {
    属性: 值;
    属性: 值;
}
```

### 常用属性

```css
.example {
    color: #333;           /* 文字颜色 */
    background-color: #f0f0f0; /* 背景颜色 */
    font-size: 16px;       /* 字体大小 */
    margin: 10px;          /* 外边距 */
    padding: 15px;         /* 内边距 */
    border: 1px solid #ccc; /* 边框 */
}
```

## 实践练习

尝试创建一个简单的个人介绍页面，包含：
- 标题
- 个人照片
- 自我介绍段落
- 联系方式链接