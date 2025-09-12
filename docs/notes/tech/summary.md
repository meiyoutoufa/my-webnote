---
sidebar_position: 1
---

# 技术总结

记录日常开发中的技术心得和总结。

## 最近学习

### Docusaurus 使用心得

**优点：**
- 基于 React，易于定制
- 支持 MDX，可以在 Markdown 中使用 React 组件
- 自动生成侧边栏和导航
- 内置搜索功能
- 支持多语言

**配置要点：**
```javascript
// docusaurus.config.ts
const config = {
  title: '网站标题',
  tagline: '网站副标题',
  // ... 其他配置
};
```

### 开发技巧

#### 1. 代码组织
- 按功能模块划分目录
- 使用有意义的文件名
- 保持代码简洁和可读性

#### 2. 调试技巧
```javascript
// 使用 console.log 调试
console.log('变量值:', variable);

// 使用 debugger 断点
function myFunction() {
    debugger; // 浏览器会在这里暂停
    // ... 代码逻辑
}
```

#### 3. 性能优化
- 避免不必要的重新渲染
- 使用适当的数据结构
- 优化图片和资源加载

## 问题记录

### 问题：Docusaurus 侧边栏文件不存在
**解决方案：**
1. 检查 `sidebars.ts` 中引用的文件路径
2. 确保对应的 `.md` 文件存在
3. 文件路径要与侧边栏配置完全匹配

### 问题：样式不生效
**解决方案：**
1. 检查 CSS 选择器是否正确
2. 确认样式文件已正确导入
3. 使用浏览器开发者工具调试

## 学习计划

- [ ] 深入学习 React Hooks
- [ ] 掌握 TypeScript 高级特性
- [ ] 学习 Node.js 后端开发
- [ ] 了解微前端架构

## 资源推荐

### 文档网站
- [MDN Web Docs](https://developer.mozilla.org/)
- [React 官方文档](https://react.dev/)
- [Docusaurus 官方文档](https://docusaurus.io/)

### 学习平台
- GitHub
- Stack Overflow
- 掘金
- 博客园