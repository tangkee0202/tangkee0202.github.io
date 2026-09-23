---
title: "2026 年现代 CSS：Container Queries、:has() 与 Anchor Positioning"
description: "如今 CSS 已经能够独立处理许多复杂布局。本文介绍三项深刻改变界面开发方式的新特性。"
pubDatetime: 2026-02-03T10:00:00Z
tags:
  - css
  - frontend
  - design
  - web
draft: false
---

很多年来，“如何用 CSS 实现某个效果？”的答案往往是“使用 JavaScript”。到了 2026 年，这个答案在多数场景中已经不再成立。尤其是下面三个特性，重新绘制了浏览器中的设计版图。

## Table of contents

## Container Queries：响应容器，而不是整个屏幕

Media Queries 响应的是 **Viewport** 宽度，但同一个组件可能因为父级布局不同，被放在窄栏或宽栏中。**Container Queries** 正是为了解决这个问题。

```css file=card.css
/* 声明容器 */
.card-wrapper {
  container-type: inline-size; /* [!code highlight] */
  container-name: card;
}

/* 组件根据所在容器响应 */
@container card (min-width: 400px) {
  /* [!code highlight] */
  .card {
    display: grid;
    grid-template-columns: 200px 1fr;
  }

  .card__image {
    grid-row: 1 / 3;
  }
}

@container card (max-width: 399px) {
  .card {
    display: flex;
    flex-direction: column;
  }
}
```

```html file=card.html
<!-- 同一个组件可以适应不同上下文 -->
<aside class="card-wrapper" style="width: 300px">
  <article class="card">...</article>
  <!-- 垂直布局 -->
</aside>

<main class="card-wrapper" style="width: 700px">
  <article class="card">...</article>
  <!-- 水平布局 -->
</main>
```

### Container Query Units

Container Queries 还提供了相对于容器的尺寸单位：

```css file=typography.css
.card__title {
  font-size: clamp(1rem, 4cqi, 2rem); /* cqi = 容器行内方向尺寸 */
}
```

## `:has()` 伪类：期待已久的父级选择器

`:has()` 可以根据元素的**后代**选择该元素，也就是 CSS 多年来缺少的“父级选择器”。

```css file=styles.css
/* 表单中存在未通过验证的必填字段 */
form:has(input:required:invalid) .submit-btn {
  opacity: 0.5;
  pointer-events: none;
}

/* 包含图片的卡片使用不同布局 */
.card:has(img) {
  /* [!code highlight] */
  display: grid;
  grid-template-columns: 150px 1fr;
}

.card:not(:has(img)) {
  padding: 1.5rem;
}

/* 导航菜单打开时，禁止 body 滚动 */
body:has(.nav-menu[aria-expanded="true"]) {
  /* [!code highlight] */
  overflow: hidden;
}
```

> 从 2023 年开始，所有现代浏览器都已支持 `:has()`，生产环境中通常不再需要 polyfill。

## Anchor Positioning：不用 JavaScript 实现 Tooltip 和 Popover

过去，把 Tooltip 定位到触发按钮旁边，需要用 JavaScript 计算位置。Anchor Positioning 让这项工作可以直接由 CSS 完成：

```css file=tooltip.css
/* 声明锚点 */
.btn-trigger {
  anchor-name: --my-button; /* [!code highlight] */
}

/* 相对于锚点定位提示框 */
.tooltip {
  position: absolute;
  position-anchor: --my-button; /* [!code highlight] */
  bottom: calc(anchor(top) + 8px); /* [!code highlight] */
  left: anchor(center); /* [!code highlight] */
  transform: translateX(-50%);

  /* 空间不足时自动翻转 */
  position-try-fallbacks: flip-block; /* [!code ++] */
}
```

```html file=tooltip.html
<button class="btn-trigger" popovertarget="tip">查看提示</button>
<div id="tip" class="tooltip" popover>
  这个提示框无需 JavaScript 就能自动定位。
</div>
```

### `position-try-fallbacks`：声明式碰撞处理

```css file=tooltip.css
.tooltip {
  position-try-fallbacks:
    flip-block,
    /* 下方空间不足时尝试上方 */ flip-inline,
    /* 右侧空间不足时尝试左侧 */ flip-start; /* 同时组合两种方式 */
}
```

## 浏览器支持情况

| 特性               | Chrome | Firefox | Safari  |
| ------------------ | ------ | ------- | ------- |
| Container Queries  | 105+ ✓ | 110+ ✓  | 16+ ✓   |
| `:has()`           | 105+ ✓ | 121+ ✓  | 15.4+ ✓ |
| Anchor Positioning | 125+ ✓ | 131+ ✓  | 18+ ✓   |

在 2026 年的浏览器分布下，大多数项目都可以在生产环境使用这三项特性。只有当目标用户仍在使用非常旧的浏览器时，才需要考虑降级方案或 polyfill。

## 今天的 CSS 更加声明式，也更有表达力

CSS 真正的静默革命不只是 Grid 或 Flexbox，而是思维方式的改变：**浏览器负责推理约束，我们只需声明想要的结果**。Container Queries、`:has()` 和 Anchor Positioning，正是这一范式的集中体现。
