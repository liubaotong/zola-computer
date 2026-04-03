+++
title = "Rust 编程语言入门指南"
date = "2026-02-11T19:00:00+08:00"

[taxonomies]
tags = ["rust", "编程", "指南"]
categories = ["编程"]

[extra]
summary = "Rust 是一门系统级编程语言，注重安全性和性能。本文带你了解 Rust 的基础概念和语法。"
author = "博主"
+++

Rust 是一门现代系统级编程语言，专注于安全、并发和性能。如果你对系统编程感兴趣，Rust 是一个绝佳的选择。

## 为什么选择 Rust？

Rust 有以下独特优势：

- **内存安全**：编译时保证内存安全，无需垃圾回收
- **零成本抽象**：高级抽象不影响运行时性能
- **出色的工具链**：Cargo 包管理器和构建工具
- **活跃的社区**：丰富的库和活跃的开发者社区

## 安装 Rust

使用 rustup 安装 Rust 工具链：

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

安装后，运行：

```bash
cargo --version
rustc --version
```

## Hello World

创建第一个 Rust 程序：

```rust
fn main() {
    println!("Hello, World!");
}
```

使用 cargo 运行：

```bash
cargo run
```

## 变量和可变性

```rust
fn main() {
    let x = 5;           // 不可变变量
    let mut y = 10;      // 可变变量
    
    y = 15;              // 可以修改
    // x = 10;            // 错误：不能修改不可变变量
}
```

## 数据类型

### 基本类型

```rust
fn main() {
    // 整数
    let a: i32 = 42;
    
    // 浮点数
    let b: f64 = 3.14159;
    
    // 布尔
    let c: bool = true;
    
    // 字符
    let d: char = '🦀';
    
    // 字符串
    let e: &str = "Hello";
}
```

### 数组和向量

```rust
fn main() {
    // 固定大小数组
    let arr: [i32; 5] = [1, 2, 3, 4, 5];
    
    // 动态大小向量
    let vec: Vec<i32> = vec![1, 2, 3, 4, 5];
}
```

## 控制流

```rust
fn main() {
    // if-else
    let number = 6;
    
    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else {
        println!("number is not divisible by 4 or 3");
    }
    
    // for 循环
    for i in 1..=5 {
        println!("{}", i);
    }
    
    // while 循环
    let mut n = 0;
    while n < 5 {
        println!("{}", n);
        n += 1;
    }
}
```

## 所有权和借用

Rust 的核心概念是所有权：

```rust
fn main() {
    let s1 = String::from("Hello");
    let s2 = s1;  // s1 的所有权转移到 s2
    
    // println!("{}", s1);  // 错误：s1 不再有效
    println!("{}", s2);
}
```

使用引用借用：

```rust
fn main() {
    let s1 = String::from("Hello");
    let len = calculate_length(&s1);  // 借用 s1
    
    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

## 结构体

```rust
struct Person {
    name: String,
    age: u32,
}

impl Person {
    fn new(name: String, age: u32) -> Person {
        Person { name, age }
    }
    
    fn introduce(&self) {
        println!("Hi, I'm {} and I'm {} years old", self.name, self.age);
    }
}

fn main() {
    let person = Person::new(String::from("Alice"), 30);
    person.introduce();
}
```

## 错误处理

```rust
use std::fs::File;

fn main() {
    // 使用 match 处理 Result
    let file_result = File::open("test.txt");
    
    let file = match file_result {
        Ok(file) => file,
        Err(error) => panic!("Error opening file: {:?}", error),
    };
    
    // 使用 ? 运算符
    fn read_file() -> Result<String, std::io::Error> {
        let file = File::open("test.txt")?;
        Ok(String::from("File read successfully"))
    }
}
```

## 下一步学习

- 学习 Trait 和泛型
- 了解生命周期
- 探索异步编程（async/await）
- 使用第三方 crates

Rust 是一门强大的语言，值得投入时间学习。祝你编程愉快！🦀