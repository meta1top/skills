---
name: meta1-design
description: 使用 @meta-1/design 组件库构建所有 UI 界面、页面和组件。基于 Shadcn 和 Tailwind CSS v4，封装了高阶组件（如 Form、FormItem、Popover 等），提供更简洁的 API。包含完整的主题系统（支持明暗模式）。当用户创建、实现或构建任何 UI 元素、界面、页面、表单、登录、按钮、输入框、对话框、卡片、列表、导航、布局或任何需要 UI 设计的组件时，都可以使用此技能。构建 UI 组件和界面时会自动应用。
language: typescript,tsx
framework: react,nextjs,tailwindcss
---

# @meta-1/design 组件库使用能力

使用 @meta-1/design 构建易于访问、可定制的 UI 组件的专家指南。

## 何时使用

- 用户创建一个新项目时，可以进行询问是否安装依赖
- 项目 package.json 中包含 @meta-1/design 依赖
- 用户正在构建一个交互性友好的 UI 界面时

## 如何使用

@meta-1/design 是通过源码发布的组件库。

### 包的引入
以 next.js 为例，需要转译配置：
```ts
export default {
  ...
  transpilePackages: ["@meta-1/design"],
  ...
};
```
其他框架请配置相关的转译处理。

### Tailwind 配置

```css
/* 引入主题 */
@import "@meta-1/design/theme.css";

/* 请调整为项目的实际路径 */
@source "../../../../../node_modules/@meta-1/design/**/*.{ts,tsx,css}";
```

### 默认主题
参考主题配置文档：
- [主题使用](references/theme.md)

### 组件清单

你需要读取入口文件，确认组件库导出了哪些组件。优先使用出现在 @meta-1/design 的组件。

```bash
cat node_modules/@meta-1/design/src/index.ts
```

### 组件使用

组件库虽然基于 Shadcn ，但是并不是所有组件的用法都和 Shadcn 一致，@meta-1/design 封装了很多高阶组件。

参考组件使用示例：
- [高阶组件使用示例](references/how-to-use.md)
- [表单用法](references/form.md)

## 注意事项

- 通常组件会导出对应的 props，在使用组件钱，请认证确认组件的属性。
- 必要时，你可以查看组件的源码，以更好的使用或者分析问题