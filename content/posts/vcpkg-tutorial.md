+++
title = "vcpkg 完全指南：C++ 包管理器的安装与使用"
date = "2026-02-14T20:00:00+08:00"

[taxonomies]
tags = ["C++", "vcpkg", "包管理器", "CMake", "开发工具"]
categories = ["编程"]

[extra]
summary = "vcpkg 是由微软开发和维护的免费开源 C/C++ 包管理器，旨在简化跨平台 C++ 开发中的依赖管理。它支持 Windows、macOS 和 Linux 三大主流操作系统，能够与各种构建系统和项目结构无缝集成。"
author = "博主"
+++

## 简介

**vcpkg** 是由微软开发和维护的免费开源 C/C++ 包管理器，旨在简化跨平台 C++ 开发中的依赖管理。它支持 Windows、macOS 和 Linux 三大主流操作系统，能够与各种构建系统和项目结构无缝集成。

### 为什么选择 vcpkg？

- **跨平台支持**：一套配置，多平台运行
- **海量库支持**：超过 2000+ 个 C++ 库
- **版本管理**：精确控制依赖库的版本
- **二进制缓存**：加速重复构建
- **Visual Studio 集成**：原生支持 MSVC 工具链
- **CMake 友好**：与现代 CMake 完美配合

---

## 安装 vcpkg

### 系统要求

- **Windows**: Windows 7 或更高版本，需要 Visual Studio 2015 Update 3 或更高版本
- **macOS**: 10.12 或更高版本，需要 Xcode 命令行工具
- **Linux**: 大多数现代发行版，需要 GCC 7 或更高版本

### 快速安装

#### Windows (PowerShell)

```powershell
# 克隆仓库
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg

# 运行安装脚本
.\bootstrap-vcpkg.bat

# 添加到环境变量（可选）
.\vcpkg integrate install
```

#### macOS / Linux (Bash)

```bash
# 克隆仓库
git clone https://github.com/Microsoft/vcpkg.git
cd vcpkg

# 运行安装脚本
./bootstrap-vcpkg.sh

# 添加到 PATH（推荐）
export PATH="$PWD:$PATH"
```

### 验证安装

```bash
vcpkg --version
```

---

## 两种工作模式

vcpkg 提供两种主要的工作模式：

### 1. Classic Mode（经典模式）

全局安装包，所有项目共享同一套库。

```bash
# 安装包到全局存储
vcpkg install fmt
vcpkg install boost
vcpkg install openssl
```

### 2. Manifest Mode（清单模式）⭐ 推荐

每个项目独立的依赖管理，通过 `vcpkg.json` 文件定义依赖。

```bash
# 初始化项目
cd your-project
vcpkg new --application

# 添加依赖
vcpkg add port fmt
vcpkg add port spdlog
vcpkg add port nlohmann-json
```

生成的 `vcpkg.json` 示例：

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "dependencies": [
    "fmt",
    "spdlog",
    "nlohmann-json"
  ]
}
```

---

## 常用命令

### 包管理命令

| 命令 | 说明 |
|------|------|
| `vcpkg search <pkg>` | 搜索包 |
| `vcpkg install <pkg>` | 安装包 |
| `vcpkg remove <pkg>` | 移除包 |
| `vcpkg list` | 列出已安装的包 |
| `vcpkg update` | 更新包列表 |
| `vcpkg upgrade` | 升级所有包 |
| `vcpkg cache` | 管理缓存 |

### 清单模式命令

| 命令 | 说明 |
|------|------|
| `vcpkg new --application` | 创建新项目 |
| `vcpkg add port <pkg>` | 添加依赖 |
| `vcpkg remove port <pkg>` | 移除依赖 |
| `vcpkg install` | 根据清单安装依赖 |

### 三元组（Triplet）

指定目标平台和架构：

```bash
# Windows x64 静态库
vcpkg install fmt:x64-windows-static

# Windows x64 动态库
vcpkg install fmt:x64-windows

# Linux x64
vcpkg install fmt:x64-linux

# macOS x64
vcpkg install fmt:x64-osx

# macOS ARM64 (Apple Silicon)
vcpkg install fmt:arm64-osx
```

查看可用三元组：

```bash
vcpkg help triplet
```

---

## 与 CMake 集成

### 方法一：工具链文件（推荐）

在 CMake 配置时指定 vcpkg 工具链：

```bash
cmake -B build \
  -DCMAKE_TOOLCHAIN_FILE=[vcpkg-root]/scripts/buildsystems/vcpkg.cmake
```

### 方法二：CMakePresets.json

```json
{
  "version": 3,
  "configurePresets": [
    {
      "name": "default",
      "hidden": true,
      "generator": "Ninja",
      "binaryDir": "${sourceDir}/build",
      "cacheVariables": {
        "CMAKE_TOOLCHAIN_FILE": "$env{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
      }
    },
    {
      "name": "windows",
      "inherits": "default",
      "condition": {
        "type": "equals",
        "lhs": "${hostSystemName}",
        "rhs": "Windows"
      }
    }
  ]
}
```

### CMakeLists.txt 示例

```cmake
cmake_minimum_required(VERSION 3.20)
project(my-project LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# vcpkg 会自动找到这些包
find_package(fmt CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)

add_executable(my-app main.cpp)

target_link_libraries(my-app PRIVATE
    fmt::fmt
    spdlog::spdlog
    nlohmann_json::nlohmann_json
)
```

### 使用 PkgConfig

对于一些使用 pkg-config 的库：

```cmake
find_package(PkgConfig REQUIRED)
pkg_check_modules(unicorn REQUIRED IMPORTED_TARGET unicorn)

add_executable(main main.c)
target_link_libraries(main PRIVATE PkgConfig::unicorn)
```

---

## 高级功能

### 1. 版本覆盖（Versioning）

在 `vcpkg.json` 中指定特定版本：

```json
{
  "name": "my-project",
  "version": "1.0.0",
  "dependencies": [
    {
      "name": "fmt",
      "version>=": "9.0.0"
    },
    "spdlog"
  ],
  "overrides": [
    {
      "name": "fmt",
      "version": "9.1.0"
    }
  ]
}
```

### 2. 特性（Features）

安装包的可选功能：

```bash
# 安装 OpenCV 并启用 CUDA 支持
vcpkg install opencv[cuda,contrib]:x64-windows
```

在清单中声明：

```json
{
  "dependencies": [
    {
      "name": "opencv4",
      "features": ["cuda", "contrib"]
    }
  ]
}
```

### 3. 自定义注册表（Registries）

使用私有或第三方包注册表：

```json
{
  "registries": [
    {
      "kind": "git",
      "repository": "https://github.com/my-org/custom-registry",
      "baseline": "abc123...",
      "packages": ["my-private-lib"]
    }
  ]
}
```

### 4. 二进制缓存

配置远程二进制缓存加速团队构建：

```bash
# 配置 Azure Blob 存储缓存
vcpkg configure binarycache \
  --azure-blob-url="https://myaccount.blob.core.windows.net/vcpkg-cache" \
  --azure-blob-token="sas-token"

# 或使用 NuGet 缓存
vcpkg configure binarycache --nuget-uri="https://myfeed/nuget/v3/index.json"
```

### 5. 平台特定配置

在 CMake 中处理平台差异：

```cmake
if(WIN32)
    target_link_libraries(my-app PRIVATE wininet.lib)
elseif(APPLE)
    find_library(CoreFoundation_FRAMEWORK CoreFoundation)
    target_link_libraries(my-app PRIVATE ${CoreFoundation_FRAMEWORK})
else()
    find_library(PTHREAD_LIBRARIES pthread)
    if(PTHREAD_LIBRARIES)
        target_link_libraries(my-app PRIVATE ${PTHREAD_LIBRARIES})
    endif()
endif()
```

---

## 最佳实践

### 1. 始终使用清单模式

为每个项目创建独立的 `vcpkg.json`，避免全局依赖冲突。

### 2. 提交 vcpkg 配置文件

将以下文件提交到版本控制：
- `vcpkg.json`
- `vcpkg-configuration.json`（如果有）
- `vcpkg.lock`（锁定确切版本）

### 3. 使用 CMake Presets

统一团队构建配置：

```json
{
  "version": 6,
  "configurePresets": [
    {
      "name": "vcpkg-base",
      "hidden": true,
      "cacheVariables": {
        "CMAKE_TOOLCHAIN_FILE": "$penv{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake"
      }
    }
  ]
}
```

### 4. 设置 VCPKG_ROOT 环境变量

```bash
# Windows
setx VCPKG_ROOT "C:\path\to\vcpkg"

# macOS/Linux
export VCPKG_ROOT=/path/to/vcpkg
```

### 5. 定期清理缓存

```bash
# 清理未使用的包
vcpkg cache clean

# 清理所有缓存（谨慎使用）
vcpkg cache clean --all
```

---

## 故障排除

### 常见问题

**问题：找不到包**
```bash
# 更新端口列表
vcpkg update

# 搜索正确的包名
vcpkg search <keyword>
```

**问题：构建失败**
```bash
# 查看详细日志
vcpkg install <pkg> --debug

# 清理并重新安装
vcpkg remove <pkg>
vcpkg install <pkg> --clean-after-build
```

**问题：CMake 找不到包**
- 确保设置了 `CMAKE_TOOLCHAIN_FILE`
- 检查包名是否正确（区分大小写）
- 确认使用了 `CONFIG` 模式：`find_package(<pkg> CONFIG REQUIRED)`

**问题：版本冲突**
```json
// 使用 overrides 强制特定版本
{
  "overrides": [
    { "name": "conflicting-pkg", "version": "1.2.3" }
  ]
}
```

---

## 总结

vcpkg 是现代 C++ 开发的必备工具，它解决了 C++ 生态中长期存在的依赖管理难题。通过清单模式、版本控制和 CMake 集成，你可以：

- ✅ 简化项目依赖管理
- ✅ 实现可重现的构建
- ✅ 提高团队协作效率
- ✅ 轻松管理跨平台项目

开始使用 vcpkg，让你的 C++ 开发体验更加顺畅！

---

## 参考资源

- [vcpkg GitHub 仓库](https://github.com/microsoft/vcpkg)
- [官方文档](https://learn.microsoft.com/en-us/vcpkg/)
- [可用包列表](https://vcpkg.io/en/packages.html)
- [CMake 集成指南](https://learn.microsoft.com/en-us/vcpkg/users/buildsystems/cmake-integration)
