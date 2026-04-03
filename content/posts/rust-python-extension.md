+++
title = "使用 Rust 开发 Python 扩展：完整教程"
date = "2026-02-14T18:00:00+08:00"

[taxonomies]
tags = ["rust", "python", "pyo3", "扩展开发"]
categories = ["编程"]

[extra]
summary = "本文详细介绍了如何使用 Rust 和 PyO3 库开发高性能的 Python 扩展，涵盖从基础函数到复杂类定义的完整实现指南。"
author = "博主"
+++

Python 是一门优秀的脚本语言，但在某些场景下性能可能成为瓶颈。通过使用 Rust 开发 Python 扩展，我们可以在保持 Python 易用性的同时，获得接近原生代码的性能。

本文将详细介绍如何使用 [PyO3](https://pyo3.rs/) 库开发 Python 扩展，涵盖从简单函数到复杂类定义的所有重要概念。

## 为什么选择 Rust 开发 Python 扩展？

1. **卓越性能**：Rust 提供接近 C/C++ 的性能，但没有内存安全问题
2. **内存安全**：编译时保证内存安全，避免空指针和缓冲区溢出
3. **易于集成**：PyO3 提供了简洁的 API 来桥接 Rust 和 Python
4. **现代化工具链**：Cargo 包管理和构建系统简化了开发流程

## 环境准备

首先确保安装了必要的工具：

```bash
# 安装 Rust 工具链
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# 安装 maturin（Python 包构建工具）
pip install maturin
```

## 项目结构

一个典型的 PyO3 项目结构如下：

```
newpy/
├── Cargo.toml          # Rust 项目配置
├── pyproject.toml      # Python 项目配置
├── src/
│   └── lib.rs          # 扩展源码
└── test_newpy.py       # 测试脚本
```

## Cargo.toml 配置

```toml
[package]
name = "newpy"
version = "0.1.0"
edition = "2021"

[lib]
name = "newpy"
crate-type = ["cdylib"]

[dependencies]
pyo3 = { version = "0.28", features = ["extension-module"] }
```

## pyproject.toml 配置

```toml
[build-system]
requires = ["maturin>=1.0,<2.0"]
build-backend = "maturin"

[project]
name = "newpy"
version = "0.1.0"
description = "A comprehensive Rust Python extension demo using PyO3"
requires-python = ">=3.8"
```

## 核心实现

### 1. 基本函数

最简单的函数实现：

```rust
use pyo3::prelude::*;

#[pyfunction]
fn fibonacci(n: usize) -> PyResult<u64> {
    if n == 0 {
        return Ok(0);
    }
    let mut a: u64 = 0;
    let mut b: u64 = 1;
    for _ in 1..n {
        let temp = a + b;
        a = b;
        b = temp;
    }
    Ok(b)
}

#[pymodule]
fn newpy(_py: Python, m: &PyModule) -> PyResult<()> {
    m.add_function(wrap_pyfunction!(fibonacci, m)?)?;
    Ok(())
}
```

Python 调用：

```python
import newpy
result = newpy.fibonacci(10)  # 返回 55
```

### 2. 函数参数处理

支持多种参数形式：

```rust
// 默认参数
#[pyfunction]
#[pyo3(signature = (a, b=10))]
fn add_with_default(a: i64, b: i64) -> i64 {
    a + b
}

// 可选参数
#[pyfunction]
#[pyo3(signature = (text, repeat=None))]
fn repeat_text(text: String, repeat: Option<usize>) -> PyResult<String> {
    let times = repeat.unwrap_or(1);
    Ok(text.repeat(times))
}

// 可变参数
#[pyfunction]
fn sum_all(args: &PyTuple) -> PyResult<f64> {
    let mut total = 0.0;
    for arg in args.iter() {
        let value: f64 = arg.extract()?;
        total += value;
    }
    Ok(total)
}
```

### 3. 类定义

定义复杂的 Python 类：

```rust
#[pyclass]
struct Counter {
    count: i64,
    name: String,
}

#[pymethods]
impl Counter {
    // 构造函数
    #[new]
    #[pyo3(signature = (name, initial_value=0))]
    fn new(name: String, initial_value: i64) -> Self {
        Counter {
            count: initial_value,
            name,
        }
    }

    // 实例方法
    fn increment(&mut self, amount: i64) -> i64 {
        self.count += amount;
        self.count
    }

    // 属性 getter/setter
    #[getter]
    fn count(&self) -> i64 {
        self.count
    }

    #[setter]
    fn set_count(&mut self, value: i64) {
        self.count = value;
    }

    // 类方法
    #[classmethod]
    fn create_default(_cls: &PyType) -> Self {
        Counter {
            count: 0,
            name: "default".to_string(),
        }
    }

    // 静态方法
    #[staticmethod]
    fn description() -> &'static str {
        "Counter is a simple class to count things"
    }

    // 特殊方法
    fn __str__(&self) -> String {
        format!("Counter(name='{}', count={})", self.name, self.count)
    }

    // 运算符重载
    fn __add__(&self, other: &Counter) -> Counter {
        Counter {
            count: self.count + other.count,
            name: format!("{}+{}", self.name, other.name),
        }
    }
}
```

### 4. 自定义异常

创建自定义异常类型：

```rust
use pyo3::create_exception;

// 创建自定义异常
create_exception!(newpy, CustomError, pyo3::exceptions::PyException);

#[pyfunction]
fn risky_operation(code: i32) -> PyResult<String> {
    if code < 0 {
        return Err(CustomError::new_err(format!("Invalid code: {}", code)));
    }
    Ok(format!("Operation succeeded with code: {}", code))
}
```

### 5. 迭代器实现

实现自定义迭代器：

```rust
#[pyclass]
struct FibonacciIterator {
    current: u64,
    next: u64,
    max_iterations: usize,
    iteration: usize,
}

#[pymethods]
impl FibonacciIterator {
    #[new]
    fn new(max_iterations: usize) -> Self {
        FibonacciIterator {
            current: 0,
            next: 1,
            max_iterations,
            iteration: 0,
        }
    }

    fn __iter__(slf: PyRef<Self>) -> PyRef<Self> {
        slf
    }

    fn __next__(mut slf: PyRefMut<Self>) -> Option<u64> {
        if slf.iteration >= slf.max_iterations {
            return None;
        }
        let result = slf.current;
        let temp = slf.current;
        slf.current = slf.next;
        slf.next = temp + slf.next;
        slf.iteration += 1;
        Some(result)
    }
}
```

### 6. 与 Python 对象交互

处理 Python 内置类型：

```rust
#[pyfunction]
fn process_list(list: &PyList) -> PyResult<PyObject> {
    let mut sum: f64 = 0.0;
    for item in list.iter() {
        if let Ok(num) = item.extract::<f64>() {
            sum += num;
        }
    }
    // 返回处理结果
    Ok(sum.into())
}
```

### 7. 高性能计算

释放 GIL 进行高性能计算：

```rust
#[pyfunction]
fn fast_sum(py: Python, numbers: &PyList) -> PyResult<f64> {
    let nums: Vec<f64> = numbers.iter()
        .map(|x| x.extract::<f64>().unwrap_or(0.0))
        .collect();
    
    // 释放 GIL 进行计算，允许其他 Python 线程并行执行
    let result = py.allow_threads(|| {
        nums.iter().sum()
    });
    
    Ok(result)
}
```

### 8. 回调函数

接受 Python 回调函数：

```rust
#[pyfunction]
fn apply_callback(callback: &PyAny, value: i64) -> PyResult<i64> {
    let args = (value * 2,);
    let result = callback.call1(args)?;
    result.extract()
}
```

## 构建和测试

构建并安装扩展：

```bash
# 构建并安装扩展
maturin develop

# 运行测试
python test_newpy.py
```

发布到 PyPI：

```bash
# 构建发布版本
maturin build --release

# 发布到 PyPI
maturin publish
```

## 关键概念总结

### PyO3 核心宏

| 宏 | 用途 |
|---|---|
| `#[pyfunction]` | 定义 Python 可调用的函数 |
| `#[pyclass]` | 定义 Python 类 |
| `#[pymethods]` | 为类实现 Python 方法 |
| `#[pymodule]` | 定义 Python 模块 |

### 常用属性

| 属性 | 用途 |
|---|---|
| `#[new]` | 构造函数 `__init__` |
| `#[getter]` | 属性 getter |
| `#[setter]` | 属性 setter |
| `#[classmethod]` | 类方法 |
| `#[staticmethod]` | 静态方法 |
| `#[pyo3(signature = ...)]` | 定义函数签名和默认值 |

### 性能优化技巧

1. **释放 GIL**：在长时间计算中使用 `py.allow_threads()` 释放全局解释器锁
2. **批量操作**：尽量减少 Python/Rust 边界穿越次数
3. **预分配内存**：对于大量数据操作，预先分配足够的内存空间

## 实际应用场景

1. **数值计算**：科学计算、数据分析
2. **图像处理**：计算机视觉、图形渲染
3. **加密算法**：密码学、哈希函数
4. **网络协议**：高性能网络服务
5. **游戏开发**：物理引擎、AI 逻辑

## 结语

使用 Rust 开发 Python 扩展是一种强大的技术组合，既保留了 Python 的易用性，又获得了 Rust 的性能和安全性。通过 PyO3 库，我们可以轻松地将 Rust 代码暴露给 Python 环境。

掌握这些技术后，你就可以为 Python 生态系统贡献高性能的扩展模块，提升应用程序的整体性能。

开始你的 Rust-Python 之旅吧！🦀🐍