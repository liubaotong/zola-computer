# 我的博客

[![Zola](https://img.shields.io/badge/Powered%20by-Zola-000000?style=flat-square&logo=zola)](https://www.getzola.org/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg?style=flat-square)](LICENSE)

> 欢迎来到我的个人博客，这里分享技术文章和生活感悟

🌐 **在线访问**: https://huajiu.pages.dev

## 📋 项目简介

这是一个基于 [Zola](https://www.getzola.org/) 构建的静态博客网站。Zola 是一个使用 Rust 编写的快速静态网站生成器，支持 Markdown、Sass、代码高亮等现代 Web 开发特性。

## ✨ 特性

- 🚀 **极速构建** - 基于 Rust 的高性能静态站点生成器
- 📝 **Markdown 支持** - 使用 Markdown 编写文章，支持 YAML 前置元数据
- 🎨 **Sass/SCSS 样式** - 现代化的 CSS 预处理器支持
- 🏷️ **标签系统** - 文章分类和标签管理
- 🔍 **代码高亮** - 使用 Catppuccin Mocha 主题的语法高亮
- 📱 **响应式设计** - 适配桌面和移动设备
- 🎭 **美观的表格样式** - 表头浅蓝色渐变背景，斑马纹效果

## 🛠️ 技术栈

| 技术 | 说明 |
|------|------|
| [Zola](https://www.getzola.org/) | 静态网站生成器 |
| [Tera](https://tera.netlify.app/) | 模板引擎（类似 Jinja2）|
| Sass/SCSS | CSS 预处理器 |
| Markdown | 文章格式 |
| HTML5 | 页面结构 |

## 📁 项目结构

```
.
├── config.toml           # Zola 配置文件
├── content/              # 网站内容
│   ├── _index.md         # 首页内容
│   ├── about.md          # 关于页面
│   └── posts/            # 博客文章
│       ├── _index.md     # 文章列表页
│       ├── cpu-architecture-guide.md
│       ├── gpu-npu-tpu-guide.md
│       ├── llm-ecosystem-guide.md
│       ├── openssh-tools-guide.md
│       └── ...           # 更多文章
├── sass/                 # Sass/SCSS 样式文件
│   └── main.scss         # 主样式文件
├── static/               # 静态资源
│   └── favicon.svg       # 网站图标
├── templates/            # HTML 模板
│   ├── base.html         # 基础模板
│   ├── index.html        # 首页模板
│   ├── page.html         # 单页模板
│   ├── section.html      # 分类模板
│   └── ...               # 其他模板
└── README.md             # 本文件
```

## 🚀 快速开始

### 环境要求

- [Zola](https://www.getzola.org/documentation/getting-started/installation/) >= 0.17.0

### 安装 Zola

**macOS/Linux:**
```bash
# 使用 Homebrew
brew install zola

# 或使用官方安装脚本
curl -sL https://github.com/getzola/zola/releases/download/v0.18.0/zola-v0.18.0-x86_64-unknown-linux-gnu.tar.gz | tar xz
sudo mv zola /usr/local/bin/
```

**Windows:**
```powershell
# 使用 Scoop
scoop install zola

# 或从 GitHub Releases 下载
# https://github.com/getzola/zola/releases
```

### 本地开发

```bash
# 克隆项目
git clone https://github.com/liubaotong/zola-computer.git
cd zola-computer

# 启动开发服务器
zola serve

# 访问 http://127.0.0.1:1111
```

### 构建网站

```bash
# 构建静态网站（输出到 public/ 目录）
zola build

# 构建并指定输出目录
zola build --output-dir ./dist
```

## 📝 写作指南

### 创建新文章

在 `content/posts/` 目录下创建新的 `.md` 文件：

```markdown
+++
title = "文章标题"
date = "2026-03-17T00:00:00+08:00"

[taxonomies]
tags = ["标签1", "标签2"]

[extra]
summary = "文章摘要"
author = "博主"
+++

# 文章标题

正文内容...
```

### 文章模板

```markdown
+++
title = ""
date = "2026-03-16T00:00:00+08:00"

[taxonomies]
tags = []

[extra]
summary = ""
author = "博主"
+++

```

### Markdown 扩展

- **代码高亮**: 使用 ```language 指定语言
- **表格**: 支持 GitHub 风格的表格
- **数学公式**: 支持 LaTeX 数学公式
- **脚注**: 支持文章脚注

## 🎨 自定义主题

### 修改样式

编辑 `sass/main.scss` 文件：

```scss
// 修改主题颜色
:root {
    --primary-color: #3498db;
    --secondary-color: #2c3e50;
    // ...
}
```

### 修改模板

编辑 `templates/` 目录下的 HTML 文件：

- `base.html` - 基础布局
- `index.html` - 首页
- `page.html` - 文章页
- `section.html` - 分类页

## 📦 部署

### Cloudflare Pages

1. 连接 Git 仓库到 Cloudflare Pages
2. 构建设置：
   - 构建命令：`zola build`
   - 构建输出目录：`public`
3. 环境变量（可选）：
   - `ZOLA_VERSION`: `0.18.0`

### GitHub Pages

使用 GitHub Actions 自动部署：

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Install Zola
      run: |
        wget https://github.com/getzola/zola/releases/download/v0.18.0/zola-v0.18.0-x86_64-unknown-linux-gnu.tar.gz
        tar xzf zola-v0.18.0-x86_64-unknown-linux-gnu.tar.gz
        sudo mv zola /usr/local/bin/
    - name: Build
      run: zola build
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./public
```

### Vercel/Netlify

1. 导入项目
2. 构建设置：
   - 构建命令：`zola build`
   - 发布目录：`public`

## 📚 文章分类

### 技术教程
- [CPU 架构全景解析](https://huajiu.pages.dev/posts/cpu-architecture-guide/)
- [GPU、NPU 与 TPU 全景解析](https://huajiu.pages.dev/posts/gpu-npu-tpu-guide/)
- [大语言模型生态全景解析](https://huajiu.pages.dev/posts/llm-ecosystem-guide/)
- [OpenSSH 工具链完全指南](https://huajiu.pages.dev/posts/openssh-tools-guide/)

### 编程语言
- [Go 语言常用第三方库](https://huajiu.pages.dev/posts/go-libraries-guide/)
- [Python 常用第三方库](https://huajiu.pages.dev/posts/python-libraries-guide/)
- [Rust 常用第三方库](https://huajiu.pages.dev/posts/rust-libraries-guide/)

### 系统运维
- [Linux 防火墙配置指南](https://huajiu.pages.dev/posts/linux-firewall-guide/)
- [Linux 初始化系统指南](https://huajiu.pages.dev/posts/linux-init-systems-guide/)
- [SSH 隧道完全指南](https://huajiu.pages.dev/posts/ssh-tunnel-guide/)

### 开发工具
- [Zola 快速入门](https://huajiu.pages.dev/posts/zola-quickstart/)
- [vcpkg 使用教程](https://huajiu.pages.dev/posts/vcpkg-tutorial/)

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

本项目采用 [MIT](LICENSE) 许可证。

## 📧 联系方式

- 📧 Email: agebook257@hotmail.com
- 💬 QQ: 121335287
- 🐙 GitHub: https://github.com/liubaotong

---

<p align="center">Made with ❤️ using <a href="https://www.getzola.org/">Zola</a></p>
