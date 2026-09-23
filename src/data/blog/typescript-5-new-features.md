---
title: "TypeScript 5.x：改变代码写法的重要特性"
description: "介绍 TypeScript 5.x 中影响较大的实用特性，包括 Standard Decorators、const Type Parameters 和更精确的类型推断。"
pubDatetime: 2026-02-15T10:00:00Z
tags:
  - typescript
  - javascript
  - dev
draft: false
---

TypeScript 仍在快速演进。5.x 系列带来的不仅是性能改进，还重新定义了一些我们使用多年的编程模式。

## Table of contents

## Standard Decorators（TC39 Stage 3）

经过多年实验版本之后，TypeScript 5.0 终于采用了 TC39 的 **Standard Decorators**。语法看起来相似，但语义已经发生了明显变化。

```typescript file=decorators.ts
// 过去的类装饰器（实验性）
@sealed
class OldClass { ... }

// TypeScript 5.x Standard Decorator // [!code highlight]
function logged<T extends new (...args: unknown[]) => unknown>(
  target: T,
  _ctx: ClassDecoratorContext,
) {
  return class extends target {
    constructor(...args: unknown[]) {
      super(...args);
      console.log(`[日志] 创建了 ${target.name} 的实例`);
    }
  };
}

@logged
class UserService {
  constructor(private db: Database) {}
}
```

### 方法与访问器装饰器

```typescript file=method-decorator.ts
function measure(_target: unknown, ctx: ClassMethodDecoratorContext) {
  const name = String(ctx.name);
  return function (this: unknown, ...args: unknown[]) {
    const start = performance.now();
    const result = (this as Record<string, Function>)[name](...args); // [!code --]
    const result = Reflect.apply(
      // [!code ++]
      _target as Function,
      this,
      args // [!code ++]
    ); // [!code ++]
    console.log(`${name} 耗时 ${performance.now() - start}ms`);
    return result;
  };
}

class ReportService {
  @measure
  async generatePDF(id: string) {
    /* ... */
  }
}
```

## `const` Type Parameters

过去，为了推断精确的字面量元组，调用时经常需要添加 `as const`。现在可以直接在泛型参数中声明：

```typescript file=const-type-params.ts
// 过去：推断结果为 string[]
function head<T>(arr: T[]) {
  return arr[0];
}
head(["a", "b"]); // 类型：string

// 现在：推断出精确字面量 // [!code highlight]
function head<const T extends readonly unknown[]>(arr: T) {
  return arr[0];
}
head(["a", "b"] as const); // 类型："a"
head(["a", "b"]); // 类型："a"，无需 as const // [!code ++]
```

## 更实用的 `satisfies` 运算符

`satisfies` 在 4.9 中引入，并在 5.x 时代成为日常工作流的一部分。它可以验证某个值是否满足类型要求，同时避免把值的类型过度扩宽。

```typescript file=satisfies.ts
type Palette = {
  red: [number, number, number] | string;
  green: [number, number, number] | string;
  blue: [number, number, number] | string;
};

const palette = {
  red: [255, 0, 0],
  green: "#00ff00",
  blue: [0, 0, 255],
} satisfies Palette; // [!code highlight]

// TypeScript 仍然知道 red 是元组，而不只是 string 或数组
palette.red.at(0);
```

## `infer` 推断能力改进

```typescript file=infer-extends.ts
// 提取满足约束的返回类型
type ReturnIfString<T> = T extends () => infer R extends string
  ? R
  : never;

type A = ReturnIfString<() => "hello">; // "hello"
type B = ReturnIfString<() => number>;  // never
```

## 性能：`--incremental` 与 `--composite` 模式

TypeScript 5.x 优化了增量构建，大型项目中的速度提升可能达到 **3 倍**：

```json file=tsconfig.json
{
  "compilerOptions": {
    "composite": true,
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",
    "moduleResolution": "bundler"
  }
}
```

> **提示：**在 monorepo 中，可以把 `composite` 与项目引用（`references`）结合使用，让每个包只编译真正发生变化的部分。

## 快速总结

| 特性                | 版本               | 影响                 |
| ------------------- | ------------------ | -------------------- |
| Standard Decorators | 5.0                | 高：替代实验性实现   |
| `const` 类型参数    | 5.0                | 中：减少 `as const`  |
| `satisfies`         | 4.9，在 5.x 中普及 | 高：让类型表达更准确 |
| `infer ... extends` | 5.x                | 中：条件类型更加精确 |
| 增量构建改进        | 5.x                | 对 monorepo 影响较大 |
