+++
title = "Zola 快速入门指南"
date = "2026-02-06T19:00:00+08:00"

[taxonomies]
tags = ["zola", "静态", "指南"]

[extra]
summary = "本文介绍如何快速开始使用 Zola 静态站点生成器，包括安装、初始化项目、构建和预览网站。"
author = "博主"
+++

Zola 是一个快速、简单的静态站点生成器，使用 Rust 编写。本文将带你快速入门 Zola。

## 什么是 Zola？

Zola 是一个单二进制文件的静态站点生成器，具有以下特点：

- **快速**：构建速度极快
- **简单**：易于安装和使用
- **灵活**：支持自定义模板
- **安全**：无运行时依赖

## 安装 Zola

### Windows

下载预编译的二进制文件，或使用 scoop 安装：

```bash
scoop install zola
```

### macOS

使用 Homebrew 安装：

```bash
brew install zola
```

### Linux

下载预编译的二进制文件或使用包管理器安装。

## 初始化项目

创建一个新的 Zola 项目：

```bash
zola init my-blog
cd my-blog
```

## 项目结构

Zola 项目的基本结构如下：

```
my-blog/
├── config.toml    # 配置文件
├── content/       # 内容目录
├── templates/     # 模板目录
├── static/        # 静态文件
└── sass/          # Sass 样式文件
```

## 创建第一篇文章

在 `content/` 目录下创建文章文件：

```bash
content/
└── posts/
    └── first-post.md
```

文章文件使用 Markdown 格式，开头是 Front Matter：

```toml
+++
title = "我的第一篇文章"
date = 2024-01-01
+++

这里是文章内容...
```

## 构建和预览

构建网站：

```bash
zola build
```

启动本地开发服务器预览：

```bash
zola serve
```

默认在 `http://127.0.0.1:1111` 访问。

## 下一步

- 创建自定义模板
- 配置主题
- 添加分类和标签
- 部署到 GitHub Pages 或其他托管平台

希望这篇指南帮助你快速上手 Zola！