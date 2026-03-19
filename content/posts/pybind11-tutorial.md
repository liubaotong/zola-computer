+++
title = "使用 PyBind11 开发 Python 扩展：完整教程"
date = "2026-02-14T19:40:00+08:00"

[taxonomies]
tags = ["python", "c++", "pybind11", "扩展开发"]
categories = ["编程"]

[extra]
summary = "本文详细介绍了如何使用 PyBind11 将 C++ 代码暴露给 Python，包括基本绑定、类定义、异常处理等核心内容。"
author = "博主"
+++

PyBind11 是一个轻量级的头文件库，用于将 C++ 代码暴露给 Python。它使用现代 C++ 特性（C++11 及以上）来简化 Python 扩展的开发过程，非常适合需要高性能计算的 C++ 项目。

## 为什么选择 PyBind11？

1. **轻量级**：只有头文件，无需复杂依赖
2. **现代 C++**：支持 C++11/14/17 特性
3. **类型安全**：自动类型转换，减少错误
4. **易于学习**：简洁的 API 设计，学习曲线平缓
5. **广泛应用**：OpenCV、LLVM 等著名项目使用

## 环境准备

安装 PyBind11：

```bash
# 使用 pip 安装
pip install pybind11

# 或使用包管理器
# vcpkg install pybind11
# conda install pybind11
```

## 项目结构

```
pybind_example/
├── CMakeLists.txt        # CMake 配置
├── setup.py              # Python 构建配置
├── src/
│   ├── main.cpp         # C++ 源码
│   └── module.cpp       # 模块定义
└── test_module.py       # 测试脚本
```

## 基础实现

### 1. 简单函数

创建 `module.cpp` 文件：

```cpp
#include <pybind11/pybind11.h>

namespace py = pybind11;

// 简单函数
int add(int a, int b) {
    return a + b;
}

// 斐波那契数列
int fibonacci(int n) {
    if (n == 0) return 0;
    int a = 0, b = 1;
    for (int i = 1; i < n; ++i) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}

// 模块定义
PYBIND11_MODULE(pybind_example, m) {
    m.doc() = "PyBind11 示例模块";
    
    // 注册函数
    m.def("add", &add, "两个整数相加");
    m.def("fibonacci", &fibonacci, "计算斐波那契数列");
    
    // 添加模块常量
    m.attr("version") = "1.0.0";
    m.attr("author") = "C++ Developer";
}
```

### 2. 函数参数处理

```cpp
#include <pybind11/pybind11.h>
#include <pybind11/stl.h>  // 支持 STL 容器
#include <string>
#include <vector>

namespace py = pybind11;

// 默认参数
std::string greet(const std::string& name = "World") {
    return "Hello, " + name + "!";
}

// 带可选参数
int multiply(int a, int b = 1, int c = 1) {
    return a * b * c;
}

// 引用参数
void increment(int& value) {
    value += 1;
}

// 支持向量
double sum_vector(const std::vector<double>& vec) {
    double total = 0.0;
    for (double v : vec) {
        total += v;
    }
    return total;
}

// 可变参数模板
template<typename... Args>
int sum_all(Args... args) {
    return (args + ...);
}
```

### 3. 类定义

```cpp
class Counter {
private:
    int count_;
    std::string name_;
    
public:
    // 构造函数
    Counter(const std::string& name, int initial_value = 0)
        : count_(initial_value), name_(name) {}
    
    // 实例方法
    int increment(int amount = 1) {
        count_ += amount;
        return count_;
    }
    
    int decrement(int amount = 1) {
        count_ -= amount;
        return count_;
    }
    
    // 属性访问
    int get_count() const { return count_; }
    void set_count(int value) { count_ = value; }
    
    std::string get_name() const { return name_; }
    
    // 字符串表示
    std::string __str__() const {
        return "Counter(name='" + name_ + "', count=" + 
               std::to_string(count_) + ")";
    }
    
    // 运算符重载
    Counter operator+(const Counter& other) const {
        return Counter(name_ + "+" + other.name_, 
                       count_ + other.count_);
    }
    
    bool operator==(const Counter& other) const {
        return count_ == other.count_ && name_ == other.name_;
    }
};

// 绑定类
PYBIND11_MODULE(pybind_example, m) {
    py::class_<Counter>(m, "Counter")
        .def(py::init<const std::string&, int>(), 
             py::arg("name") = "default", 
             py::arg("initial_value") = 0)
        .def("increment", &Counter::increment,
             py::arg("amount") = 1)
        .def("decrement", &Counter::decrement,
             py::arg("amount") = 1)
        .def_property("count", &Counter::get_count, 
                      &Counter::set_count)
        .def_property_readonly("name", &Counter::get_name)
        .def("__str__", &Counter::__str__)
        .def("__repr__", &Counter::__str__)
        .def(py::self + py::self)
        .def(py::self == py::self);
}
```

### 4. 属性绑定

```cpp
class Point {
public:
    double x, y;
    
    Point(double x = 0, double y = 0) : x(x), y(y) {}
    
    double distance(const Point& other) const {
        double dx = x - other.x;
        double dy = y - other.y;
        return std::sqrt(dx * dx + dy * dy);
    }
};

PYBIND11_MODULE(pybind_example, m) {
    py::class_<Point>(m, "Point")
        .def(py::init<double, double>())
        .def_readwrite("x", &Point::x)  // 读写属性
        .def_readwrite("y", &Point::y)
        .def("distance", &Point::distance)
        .def("__str__", [](const Point& p) {
            return "Point(" + std::to_string(p.x) + ", " + 
                   std::to_string(p.y) + ")";
        });
}
```

### 5. 静态方法和类方法

```cpp
class MathUtils {
public:
    // 静态方法
    static int square(int x) {
        return x * x;
    }
    
    // 类方法（工厂方法）
    static Point create_point(double x, double y) {
        return Point(x, y);
    }
};

PYBIND11_MODULE(pybind_example, m) {
    py::class_<MathUtils>(m, "MathUtils")
        .def_static("square", &MathUtils::square)
        .def_static("create_point", &MathUtils::create_point);
}
```

### 6. 构造函数重载

```cpp
class Rectangle {
private:
    double width_, height_;
    
public:
    Rectangle() : width_(0), height_(0) {}
    Rectangle(double size) : width_(size), height_(size) {}
    Rectangle(double width, double height) 
        : width_(width), height_(height) {}
    
    double area() const { return width_ * height_; }
    double perimeter() const { return 2 * (width_ + height_); }
};

PYBIND11_MODULE(pybind_example, m) {
    py::class_<Rectangle>(m, "Rectangle")
        .def(py::init<>())  // 默认构造函数
        .def(py::init<double>())  // 正方形
        .def(py::init<double, double>())  // 矩形
        .def("area", &Rectangle::area)
        .def("perimeter", &Rectangle::perimeter);
}
```

### 7. 智能指针

```cpp
#include <memory>

class DataHolder {
private:
    std::shared_ptr<std::string> data_;
    
public:
    DataHolder(const std::string& data) 
        : data_(std::make_shared<std::string>(data)) {}
    
    std::string get_data() const { return *data_; }
    void set_data(const std::string& data) { *data_ = data; }
};

PYBIND11_MODULE(pybind_example, m) {
    py::class_<DataHolder, std::shared_ptr<DataHolder>>(m, "DataHolder")
        .def(py::init<const std::string&>())
        .def_property("data", &DataHolder::get_data, 
                      &DataHolder::set_data);
}
```

### 8. 枚举类型

```cpp
enum class Status {
    Pending,
    Running,
    Completed,
    Failed
};

PYBIND11_MODULE(pybind_example, m) {
    py::enum_<Status>(m, "Status")
        .value("Pending", Status::Pending)
        .value("Running", Status::Running)
        .value("Completed", Status::Completed)
        .value("Failed", Status::Failed)
        .export_values();
}
```

### 9. 异常处理

```cpp
#include <stdexcept>

class CustomError : public std::runtime_error {
public:
    CustomError(int code, const std::string& message)
        : std::runtime_error(message), code_(code) {}
    
    int get_code() const { return code_; }
    
private:
    int code_;
};

int risky_operation(int code) {
    if (code < 0) {
        throw CustomError(code, "Invalid code");
    }
    return code * 10;
}

PYBIND11_MODULE(pybind_example, m) {
    // 注册自定义异常
    py::register_exception<CustomError>(m, "CustomError");
    
    m.def("risky_operation", &risky_operation,
          "可能抛出异常的操作");
}
```

### 10. 迭代器

```cpp
#include <pybind11/pybind11.h>
#include <pybind11/operators.h>

class FibonacciIterator {
private:
    int current_, next_, max_iterations_, iteration_;
    
public:
    FibonacciIterator(int max_iterations)
        : current_(0), next_(1), 
          max_iterations_(max_iterations), iteration_(0) {}
    
    FibonacciIterator& __iter__() { return *this; }
    
    int __next__() {
        if (iteration_ >= max_iterations_) {
            throw py::stop_iteration();
        }
        int result = current_;
        int temp = current_;
        current_ = next_;
        next_ = temp + next_;
        iteration_ += 1;
        return result;
    }
};

PYBIND11_MODULE(pybind_example, m) {
    py::class_<FibonacciIterator>(m, "FibonacciIterator")
        .def(py::init<int>())
        .def("__iter__", &FibonacciIterator::__iter__)
        .def("__next__", &FibonacciIterator::__next__);
}
```

### 11. 回调函数

```cpp
#include <pybind11/functional.h>

int apply_callback(const std::function<int(int)>& callback, int value) {
    return callback(value * 2);
}

std::vector<int> map_with_callback(
    const std::vector<int>& list,
    const std::function<int(int)>& callback) {
    std::vector<int> result;
    result.reserve(list.size());
    for (int item : list) {
        result.push_back(callback(item));
    }
    return result;
}

PYBIND11_MODULE(pybind_example, m) {
    m.def("apply_callback", &apply_callback);
    m.def("map_with_callback", &map_with_callback);
}
```

### 12. NumPy 集成

```cpp
#define PY_ARRAY_UNIQUE_SYMBOL pybind_example_ARRAY_API
#include <pybind11/numpy.h>

py::array_t<double> process_numpy(py::array_t<double> input) {
    // 获取数组信息
    auto buf = input.request();
    double* ptr = static_cast<double*>(buf.ptr);
    size_t size = buf.size;
    
    // 创建输出数组
    auto result = py::array_t<double>(size);
    auto result_buf = result.request();
    double* result_ptr = static_cast<double*>(result_buf.ptr);
    
    // 处理数据
    for (size_t i = 0; i < size; ++i) {
        result_ptr[i] = ptr[i] * 2.0;
    }
    
    return result;
}
```

## 构建配置

### CMakeLists.txt

```cmake
cmake_minimum_required(VERSION 3.4)
project(pybind_example)

# 查找 Python 和 PyBind11
find_package(Python3 COMPONENTS Interpreter Development REQUIRED)
find_package(pybind11 REQUIRED)

# 创建 Python 模块
pybind11_add_module(pybind_example src/module.cpp)

# 链接目标
target_link_libraries(pybind_example PRIVATE pybind11::module)

# 安装位置
install(TARGETS pybind_example DESTINATION .)
```

### setup.py（替代方案）

```python
from setuptools import setup, Extension
from pybind11 import get_include

ext_modules = [
    Extension(
        'pybind_example',
        ['src/module.cpp'],
        include_dirs=[get_include()],
        language='c++',
        extra_compile_args=['-std=c++11', '-O3'],
    ),
]

setup(
    name='pybind_example',
    ext_modules=ext_modules,
    zip_safe=False,
)
```

### pyproject.toml

```toml
[build-system]
requires = ["setuptools", "wheel", "pybind11"]
build-backend = "setuptools.build_meta"

[project]
name = "pybind_example"
requires-python = ">=3.8"

[tool.setuptools]
py-modules = []

[tool.setuptools.cmake]
minimum-version = "3.4"

[tool.cibuildwheel]
test-command = "python -m pytest {project}/tests"
```

## 构建和安装

```bash
# 使用 setuptools
pip install .

# 使用 CMake
mkdir build && cd build
cmake ..
cmake --build .
python -m pip install .
```

## 性能优化技巧

### 1. 使用 py::call_guard

```cpp
m.def("fast_operation", &fast_operation,
      py::call_guard<py::gil_scoped_release>());
```

### 2. 避免不必要的类型转换

```cpp
// 避免：频繁转换
double process(const py::object& obj) {
    double value = obj.cast<double>();
    return value * 2;
}

// 推荐：直接使用 C++ 类型
double process(double value) {
    return value * 2;
}
```

### 3. 批量处理

```cpp
py::array_t<double> batch_process(const py::array_t<double>& arr) {
    auto buf = arr.request();
    double* ptr = static_cast<double*>(buf.ptr);
    
    // 批量处理逻辑
    for (size_t i = 0; i < buf.size; ++i) {
        ptr[i] = std::sqrt(ptr[i]);
    }
    
    return arr;
}
```

## 与 Cython 和 PyO3 对比

| 特性 | PyBind11 | Cython | PyO3 (Rust) |
|------|----------|--------|--------------|
| 语言 | C++ | Python 超集 | Rust |
| 学习曲线 | 中等 | 低 | 中等 |
| 性能 | 极高 | 高 | 极高 |
| 内存安全 | 手动管理 | 手动管理 | 编译时保证 |
| 现代特性 | C++11/14/17 | Python+C | Rust 特性 |
| 构建复杂度 | 中等 | 简单 | 简单 |
| 生态 | 活跃发展 | 成熟稳定 | 快速成长 |

## 最佳实践

1. **保持接口简单**：尽量使用简单的数据类型
2. **错误处理**：使用异常而不是错误码
3. **文档注释**：为每个函数和类添加文档
4. **性能测试**：使用 `timeit` 测试性能
5. **兼容性检查**：测试不同 Python 版本

## 实际应用场景

### 高性能计算

```cpp
py::array_t<double> matrix_multiply(
    const py::array_t<double>& A,
    const py::array_t<double>& B) {
    
    auto buf_A = A.request();
    auto buf_B = B.request();
    
    double* A_ptr = static_cast<double*>(buf_A.ptr);
    double* B_ptr = static<double*>(buf_B.ptr);
    
    // 矩阵乘法实现
    // ...
    
    return result;
}
```

### 图像处理

```cpp
py::array_t<uint8_t> process_image(
    const py::array_t<uint8_t>& image,
    int width,
    int height) {
    
    // 图像处理逻辑
    // ...
    
    return processed_image;
}
```

### 机器学习推理

```cpp
py::array_t<float> predict(
    const py::array_t<float>& input,
    const std::string& model_path) {
    
    // 加载模型和推理
    // ...
    
    return prediction;
}
```

## 总结

PyBind11 是连接 C++ 和 Python 的桥梁，特别适合：
- 已有 C++ 代码库的项目
- 需要极致性能的应用
- 复杂的数值计算和数据处理

它的轻量级设计和现代 C++ 特性使其成为高性能 Python 扩展的理想选择。