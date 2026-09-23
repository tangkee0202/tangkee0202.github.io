---
title: "写给 JavaScript 开发者的 Rust：值得迈出的一步"
description: "如果你来自 JS/TS 世界，又对 Rust 感到畏惧，这篇指南会用熟悉的概念和示例带你认识 Rust。"
pubDatetime: 2026-02-12T10:00:00Z
tags:
  - rust
  - javascript
  - systems
  - webassembly
draft: false
---

Rust 很早就进入了 Web 开发者的视野，但普及速度一直不快。到了 2026 年，情况已经改变：Biome、Oxc、Rolldown 和 SWC 编译器等 JavaScript 生态关键工具都由 Rust 驱动，WebAssembly 也让它在前端领域变得越来越重要。现在正是学习它的好时机。

## Table of contents

## 最大的思维转变：Ownership

在 JavaScript 中，Garbage Collector 负责管理内存；在 Rust 中，这项责任通过 **Ownership System** 交给编译器。

```rust file=ownership.rs
// 在 JS 中可以这样做
// let a = [1, 2, 3];
// let b = a; // a 仍然有效

// 在 Rust 中：
fn main() {
    let a = vec![1, 2, 3];
    let b = a;           // a 的 Ownership 被移动给 b // [!code highlight]
    println!("{:?}", a); // ✗ 错误：a 已被移动
    println!("{:?}", b); // ✓
}
```

解决方法是通过引用进行 **Borrowing**：

```rust file=borrowing.rs
fn main() {
    let a = vec![1, 2, 3];
    let b = &a;           // Immutable Borrowing // [!code ++]
    println!("{:?}", a); // ✓ a 仍然有效
    println!("{:?}", b); // ✓
}

fn print_vec(v: &Vec<i32>) { // 接收引用，而不是取得 Ownership // [!code highlight]
    for n in v {
        print!("{} ", n);
    }
}
```

## 类型：从 `any` 到高度安全的类型系统

| JavaScript/TypeScript  | Rust 对应类型                  |
| ---------------------- | ------------------------------ |
| `number`               | `i32`、`u32`、`f64` 等         |
| `string`               | `String`（堆）/ `&str`（切片） |
| `T \| null`            | `Option<T>`                    |
| `T \| Error`           | `Result<T, E>`                 |
| `any[]`                | `Vec<T>`                       |
| `{ [key: string]: T }` | `HashMap<String, T>`           |

```rust file=types.rs
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None   // 不使用 null，也能表达“没有结果”
    } else {
        Some(a / b)
    }
}

fn main() {
    match divide(10.0, 0.0) {
        Some(result) => println!("结果：{result}"),
        None => println!("不能除以零"),
    }
}
```

## 错误处理：`Result` 是 Rust 世界的 `Promise`

JavaScript 通常使用 `try/catch` 或 Promise 链处理错误；在 Rust 中，惯用方式是 `Result<T, E>`：

```rust file=errors.rs
use std::fs;
use std::io;

// 过去：不使用 ? 运算符
fn read_config_verbose() -> Result<String, io::Error> {
    let content = match fs::read_to_string("config.toml") { // [!code --]
        Ok(c) => c,                                         // [!code --]
        Err(e) => return Err(e),                            // [!code --]
    };                                                      // [!code --]
    Ok(content.to_uppercase())
}

// 使用 ? 运算符，可自动向上传递错误
fn read_config() -> Result<String, io::Error> {          // [!code ++]
    let content = fs::read_to_string("config.toml")?;   // [!code ++]
    Ok(content.to_uppercase())                            // [!code ++]
}
```

## 闭包与高阶函数

语法虽然不同，但概念与 JavaScript 非常接近：

```rust file=closures.rs
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    // map + filter + collect，类似 JS 的 Array.map + filter
    let double_evens: Vec<i32> = numbers
        .iter()
        .filter(|&&x| x % 2 == 0) // [!code highlight]
        .map(|&x| x * 2)          // [!code highlight]
        .collect();

    println!("{:?}", double_evens); // [4, 8]
}
```

## Rust → WebAssembly：连接前端的桥梁

```rust file=lib.rs
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
pub fn fibonacci(n: u32) -> u32 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}
```

```bash
# 编译为 WASM
wasm-pack build --target web
```

```javascript file=main.js
import init, { fibonacci } from "./pkg/my_project.js";

await init();
console.log(fibonacci(40)); // 通常比纯 JS 版本快得多
```

## 从哪里开始

1. **[The Rust Book](https://doc.rust-lang.org/book/)**——系统学习 Rust 的官方文档。
2. **Rustlings**——在终端中完成的交互式练习。
3. **[Rust by Example](https://doc.rust-lang.org/rust-by-example/)**——通过真实示例学习。
4. 使用 **`wasm-pack`** 构建一个小功能，并在现有 Web 项目中调用它。

> Rust 的学习曲线确实存在，但编译器也是最好的老师之一：错误信息详细、准确，而且通常会直接给出解决方向。
