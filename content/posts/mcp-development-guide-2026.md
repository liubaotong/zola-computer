+++
title = "MCP 开发完全指南 2026：Python、TypeScript、Go 和 Rust 实现详解"
date = "2026-03-18T21:00:00+08:00"

[taxonomies]
tags = ["MCP", "Model Context Protocol", "AI", "Python", "TypeScript", "Go", "Rust", "开发指南"]

[extra]
summary = "全面介绍 Model Context Protocol (MCP) 开发，包含架构设计、核心概念、Python/TypeScript/Go/Rust 四种语言完整实现、生产级部署方案，帮助你构建 AI Agent 与外部工具的标准接口。"
author = "博主"
+++

**Model Context Protocol (MCP)** 正在成为 AI Agent 与外部工具通信的事实标准。从 Anthropic 在 2024 年 11 月发布至今，MCP 已获得 OpenAI、Google、Microsoft 等巨头的支持，超过 1000 个公共 MCP 服务器投入使用。

本文将深入介绍 MCP 的架构设计、核心概念，并提供 **Python、TypeScript、Go（两种 SDK）和 Rust** 四种语言的完整实现示例，帮助你快速掌握 MCP 开发技能。

---

## 一、什么是 MCP？

### 1.1 MCP 简介

**Model Context Protocol (MCP)** 是一个开放标准，定义了 AI 应用如何与外部工具和数据源通信。它解决了 AI Agent 集成中的关键问题：**如何让任何 AI 客户端都能以标准化方式发现和调用外部工具**。

**核心价值：**
- **标准化**：统一的协议，无需为每个 AI 平台定制集成
- **安全性**：工具逻辑与 LLM 隔离，沙箱执行
- **可发现性**：AI 客户端自动发现可用工具
- **可复用**：一次开发，多处使用

### 1.2 为什么需要 MCP？

#### 传统集成方式的问题

```
┌─────────────────────────────────────────────────────────┐
│                    传统集成方式                           │
├─────────────────────────────────────────────────────────┤
│  Claude → 自定义集成 → 你的 API                          │
│  ChatGPT → 自定义集成 → 你的 API                         │
│  Cursor → 自定义集成 → 你的 API                          │
│  VS Code → 自定义集成 → 你的 API                         │
│                                                         │
│  问题：每个客户端都需要单独的集成代码                    │
└─────────────────────────────────────────────────────────┘
```

#### MCP 解决方案

```
┌─────────────────────────────────────────────────────────┐
│                     MCP 架构                             │
├─────────────────────────────────────────────────────────┤
│  Claude ──┐                                             │
│  ChatGPT ─┼──→ MCP Client → MCP Server → 你的 API       │
│  Cursor ──┤                    │                        │
│  VS Code ─┘                    ↓                        │
│                          任何 MCP 兼容客户端              │
│                          都能使用你的工具                 │
└─────────────────────────────────────────────────────────┘
```

**一句话总结**：MCP 就像 AI 领域的 USB-C 接口——一个标准接口连接所有设备。

---

## 二、MCP 架构详解

### 2.1 核心组件

```
┌─────────────────────────────────────────────────────────┐
│                    MCP 生态系统                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────┐         ┌─────────────┐               │
│  │ MCP Host    │         │ MCP Client  │               │
│  │ (Claude     │◄───────►│ (AI Agent   │               │
│  │  Desktop,   │         │  应用)       │               │
│  │  Cursor)    │         │             │               │
│  └─────────────┘         └──────┬──────┘               │
│                                 │ JSON-RPC 2.0          │
│                                 │ (stdio / HTTP)        │
│                                 ↓                       │
│                        ┌─────────────┐                  │
│                        │ MCP Server  │                  │
│                        │ (你的工具)   │                  │
│                        │             │                  │
│                        │ ┌─────────┐ │                  │
│                        │ │  Tools  │ │                  │
│                        │ │ Resources│ │                 │
│                        │ │  Prompts │ │                 │
│                        │ └─────────┘ │                  │
│                        └─────────────┘                  │
│                                 │                       │
│                                 ↓                       │
│                        ┌─────────────┐                  │
│                        │ 你的 API/    │                  │
│                        │ 数据库/服务  │                  │
│                        └─────────────┘                  │
└─────────────────────────────────────────────────────────┘
```

### 2.2 三个核心原语

MCP 服务器暴露三种能力：

#### 1. Tools（工具）
**模型控制** - LLM 可以调用的函数

```
用途：执行操作、产生副作用
类似：REST API 的 POST/PUT 端点
示例：查询天气、执行计算、发送邮件
```

#### 2. Resources（资源）
**应用控制** - 只读数据源

```
用途：提供上下文信息
类似：REST API 的 GET 端点
示例：文件内容、数据库记录、配置信息
```

#### 3. Prompts（提示）
**用户控制** - 可复用的交互模板

```
用途：编码最佳实践
类似：预定义的指令模板
示例：代码审查模板、邮件写作模板
```

### 2.3 通信协议

MCP 基于 **JSON-RPC 2.0** 协议：

```json
// 客户端请求
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {}
}

// 服务器响应
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "tools": [
      {
        "name": "get_weather",
        "description": "获取城市天气",
        "inputSchema": {
          "type": "object",
          "properties": {
            "city": {"type": "string"}
          },
          "required": ["city"]
        }
      }
    ]
  }
}
```

### 2.4 传输层

| 传输方式 | 用途 | 优点 | 缺点 |
|---------|------|------|------|
| **stdio** | 本地进程 | 简单、安全 | 仅限本地 |
| **Streamable HTTP** | 远程服务 | 灵活、可扩展 | 需要认证 |
| **SSE** | 远程服务 | 实时推送 | 已弃用 |

---

## 三、Python 实现

### 3.1 环境准备

```bash
# 创建虚拟环境
python -m venv mcp-env
source mcp-env/bin/activate  # Windows: mcp-env\Scripts\activate

# 安装官方 MCP SDK
pip install mcp

# 或使用 FastMCP（更简洁）
pip install fastmcp

# 安装依赖
pip install pydantic httpx python-dotenv
```

### 3.2 使用 FastMCP 构建天气服务器

```python
"""
MCP 天气服务器 - Python 实现
使用 FastMCP 简化开发
"""

from typing import Literal
from fastmcp import FastMCP
import httpx

# 创建 MCP 服务器
mcp = FastMCP(
    name="weather-server",
    version="1.0.0",
    dependencies=["httpx"]
)

# 模拟天气数据库
WEATHER_DATABASE = {
    "beijing": {"temperature": 25, "condition": "晴朗", "humidity": 45},
    "shanghai": {"temperature": 28, "condition": "多云", "humidity": 60},
    "guangzhou": {"temperature": 32, "condition": "小雨", "humidity": 75},
    "shenzhen": {"temperature": 30, "condition": "雷阵雨", "humidity": 80},
}

# ============== Tools（工具）=============

@mcp.tool()
async def get_weather(city: str) -> str:
    """
    获取指定城市的当前天气信息。
    
    Args:
        city: 城市名称（如 'beijing', 'shanghai'）
        
    Returns:
        天气信息字符串
    """
    city_lower = city.lower()
    
    if city_lower not in WEATHER_DATABASE:
        available = ", ".join(WEATHER_DATABASE.keys())
        return f'城市 "{city}" 未找到。可用城市：{available}'
    
    data = WEATHER_DATABASE[city_lower]
    return (
        f"天气信息：\n"
        f"城市：{city}\n"
        f"温度：{data['temperature']}°C\n"
        f"状况：{data['condition']}\n"
        f"湿度：{data['humidity']}%"
    )

@mcp.tool()
async def convert_temperature(
    value: float,
    from_unit: Literal["celsius", "fahrenheit", "kelvin"],
    to_unit: Literal["celsius", "fahrenheit", "kelvin"]
) -> float:
    """
    在不同温度单位之间转换。
    
    Args:
        value: 温度值
        from_unit: 源单位 (celsius/fahrenheit/kelvin)
        to_unit: 目标单位 (celsius/fahrenheit/kelvin)
        
    Returns:
        转换后的温度值
    """
    # 先转换为摄氏度
    if from_unit == "celsius":
        celsius = value
    elif from_unit == "fahrenheit":
        celsius = (value - 32) * 5/9
    elif from_unit == "kelvin":
        celsius = value - 273.15
    
    # 再从摄氏度转换为目标单位
    if to_unit == "celsius":
        result = celsius
    elif to_unit == "fahrenheit":
        result = celsius * 9/5 + 32
    elif to_unit == "kelvin":
        result = celsius + 273.15
    
    return round(result, 2)

# ============== Resources（资源）=============

@mcp.resource("weather://supported-cities")
def get_supported_cities() -> str:
    """获取支持的城市列表"""
    cities = list(WEATHER_DATABASE.keys())
    return f"支持的城市：{', '.join(cities)}"

@mcp.resource("weather://server-info")
def get_server_info() -> dict:
    """获取服务器信息"""
    return {
        "name": "weather-server",
        "version": "1.0.0",
        "capabilities": ["tools", "resources"],
        "total_cities": len(WEATHER_DATABASE)
    }

# ============== 运行服务器 =============

if __name__ == "__main__":
    # 方式 1: stdio 传输（本地，Claude Desktop 使用）
    mcp.run()
    
    # 方式 2: HTTP 传输（远程服务）
    # mcp.run(transport="streamable-http", host="0.0.0.0", port=8000)
```

---

## 四、TypeScript 实现

### 4.1 环境准备

```bash
# 创建项目
mkdir mcp-weather-server-ts && cd mcp-weather-server-ts
npm init -y

# 安装官方 MCP SDK
npm install @modelcontextprotocol/sdk zod

# 安装开发依赖
npm install -D typescript @types/node

# 初始化 TypeScript
npx tsc --init
```

### 4.2 使用官方 SDK 构建天气服务器

```typescript
/**
 * MCP 天气服务器 - TypeScript 实现
 * 使用官方 @modelcontextprotocol/sdk
 */

import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import { z } from "zod";

// 创建 MCP 服务器
const server = new McpServer({
  name: "weather-server",
  version: "1.0.0",
});

// 模拟天气数据库
const WEATHER_DATABASE: Record<string, WeatherData> = {
  beijing: { temperature: 25, condition: "晴朗", humidity: 45 },
  shanghai: { temperature: 28, condition: "多云", humidity: 60 },
  guangzhou: { temperature: 32, condition: "小雨", humidity: 75 },
  shenzhen: { temperature: 30, condition: "雷阵雨", humidity: 80 },
};

interface WeatherData {
  temperature: number;
  condition: string;
  humidity: number;
}

// ============== Tools（工具）=============

// 工具 1: 获取天气
server.tool(
  "get_weather",
  "获取指定城市的当前天气信息",
  {
    city: z.string().describe("城市名称（如 'beijing', 'shanghai'）"),
  },
  async ({ city }) => {
    const cityLower = city.toLowerCase();
    
    if (!(cityLower in WEATHER_DATABASE)) {
      const available = Object.keys(WEATHER_DATABASE).join(", ");
      return {
        content: [
          {
            type: "text",
            text: `城市 "${city}" 未找到。可用城市：${available}`,
          },
        ],
      };
    }
    
    const data = WEATHER_DATABASE[cityLower];
    return {
      content: [
        {
          type: "text",
          text: `天气信息：\n城市：${city}\n温度：${data.temperature}°C\n状况：${data.condition}\n湿度：${data.humidity}%`,
        },
      ],
    };
  }
);

// 工具 2: 温度转换
server.tool(
  "convert_temperature",
  "在不同温度单位之间转换",
  {
    value: z.number().describe("温度值"),
    from_unit: z
      .enum(["celsius", "fahrenheit", "kelvin"])
      .describe("源单位"),
    to_unit: z
      .enum(["celsius", "fahrenheit", "kelvin"])
      .describe("目标单位"),
  },
  async ({ value, from_unit, to_unit }) => {
    // 转换逻辑
    let celsius: number;
    if (from_unit === "celsius") {
      celsius = value;
    } else if (from_unit === "fahrenheit") {
      celsius = (value - 32) * 5 / 9;
    } else {
      celsius = value - 273.15;
    }
    
    let result: number;
    if (to_unit === "celsius") {
      result = celsius;
    } else if (to_unit === "fahrenheit") {
      result = celsius * 9 / 5 + 32;
    } else {
      result = celsius + 273.15;
    }
    
    return {
      content: [
        {
          type: "text",
          text: `${value} ${from_unit} = ${result.toFixed(2)} ${to_unit}`,
        },
      ],
    };
  }
);

// ============== 运行服务器 ==============

async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  
  console.error("🌤️  天气 MCP 服务器运行在 stdio");
  console.error(`支持 ${Object.keys(WEATHER_DATABASE).length} 个城市`);
}

main().catch((error) => {
  console.error("服务器错误:", error);
  process.exit(1);
});
```

---

## 五、Go 实现（两种方式）

Go 语言有两种主流的 MCP SDK 实现：

1. **官方 SDK** (`github.com/modelcontextprotocol/go-sdk`) - Anthropic 官方维护
2. **mark3labs SDK** (`github.com/mark3labs/mcp-go`) - 社区驱动，更简洁

### 5.1 环境准备

```bash
# 创建项目
mkdir mcp-weather-server-go && cd mcp-weather-server-go
go mod init mcp-weather-server-go

# 安装官方 SDK
go get github.com/modelcontextprotocol/go-sdk@latest

# 或安装 mark3labs SDK（更简洁）
go get github.com/mark3labs/mcp-go@latest
```

### 5.2 使用官方 SDK 实现

```go
// MCP 天气服务器 - Go 实现
// 使用官方 Go SDK

package main

import (
	"context"
	"encoding/json"
	"fmt"
	"os"
	"strings"

	"github.com/modelcontextprotocol/go-sdk/mcp"
)

// 天气数据结构
type WeatherData struct {
	Temperature float64 `json:"temperature"`
	Condition   string  `json:"condition"`
	Humidity    float64 `json:"humidity"`
}

// 模拟天气数据库
var weatherDatabase = map[string]WeatherData{
	"beijing":   {Temperature: 25, Condition: "晴朗", Humidity: 45},
	"shanghai":  {Temperature: 28, Condition: "多云", Humidity: 60},
	"guangzhou": {Temperature: 32, Condition: "小雨", Humidity: 75},
	"shenzhen":  {Temperature: 30, Condition: "雷阵雨", Humidity: 80},
}

func main() {
	// 创建 MCP 服务器
	server := mcp.NewServer(&mcp.ServerOptions{
		Name:    "weather-server",
		Version: "1.0.0",
	})

	// 注册工具
	server.AddTool("get_weather",
		"获取指定城市的当前天气信息",
		map[string]interface{}{
			"type": "object",
			"properties": map[string]interface{}{
				"city": map[string]interface{}{
					"type":        "string",
					"description": "城市名称（如 'beijing', 'shanghai'）",
				},
			},
			"required": []string{"city"},
		},
		getWeatherHandler,
	)

	// 启动服务器
	fmt.Fprintln(os.Stderr, "🌤️  天气 MCP 服务器启动 (官方 SDK)")
	fmt.Fprintf(os.Stderr, "支持 %d 个城市\n", len(weatherDatabase))

	if err := server.RunStdio(); err != nil {
		fmt.Fprintf(os.Stderr, "服务器错误：%v\n", err)
		os.Exit(1)
	}
}

// getWeatherHandler 获取天气处理函数
func getWeatherHandler(ctx context.Context, args map[string]interface{}) (*mcp.ToolResult, error) {
	city, ok := args["city"].(string)
	if !ok {
		return &mcp.ToolResult{
			Content: []mcp.Content{
				mcp.TextContent{Text: "参数错误：city 必须是字符串"},
			},
		}, nil
	}

	cityLower := strings.ToLower(city)
	data, exists := weatherDatabase[cityLower]
	if !exists {
		return &mcp.ToolResult{
			Content: []mcp.Content{
				mcp.TextContent{Text: fmt.Sprintf(`城市 "%s" 未找到`, city)},
			},
		}, nil
	}

	result := fmt.Sprintf(
		"天气信息：\n城市：%s\n温度：%.1f°C\n状况：%s\n湿度：%.0f%%",
		city, data.Temperature, data.Condition, data.Humidity,
	)

	return &mcp.ToolResult{
		Content: []mcp.Content{
			mcp.TextContent{Text: result},
		},
	}, nil
}
```

### 5.3 使用 mark3labs/mcp-go 实现（推荐）

```go
// MCP 天气服务器 - Go 实现
// 使用 mark3labs/mcp-go SDK（更简洁的 API）

package main

import (
	"context"
	"encoding/json"
	"fmt"
	"os"
	"strings"

	"github.com/mark3labs/mcp-go/mcp"
	"github.com/mark3labs/mcp-go/server"
)

// 天气数据结构
type WeatherData struct {
	Temperature float64 `json:"temperature"`
	Condition   string  `json:"condition"`
	Humidity    float64 `json:"humidity"`
}

// 模拟天气数据库
var weatherDatabase = map[string]WeatherData{
	"beijing":   {Temperature: 25, Condition: "晴朗", Humidity: 45},
	"shanghai":  {Temperature: 28, Condition: "多云", Humidity: 60},
	"guangzhou": {Temperature: 32, Condition: "小雨", Humidity: 75},
	"shenzhen":  {Temperature: 30, Condition: "雷阵雨", Humidity: 80},
}

func main() {
	// 创建 MCP 服务器
	s := server.NewMCPServer(
		"weather-server",
		"1.0.0",
		server.WithResourceCapabilities(true, true),
		server.WithToolCapabilities(true),
	)

	// 注册工具 - 使用链式调用
	s.AddTool(
		mcp.NewTool(
			"get_weather",
			mcp.WithDescription("获取指定城市的当前天气信息"),
			mcp.WithString("city",
				mcp.Description("城市名称（如 'beijing', 'shanghai'）"),
				mcp.Required(),
			),
		),
		getWeatherHandler,
	)

	// 注册工具 - 温度转换
	s.AddTool(
		mcp.NewTool("convert_temperature",
			mcp.WithDescription("在不同温度单位之间转换"),
			mcp.WithNumber("value",
				mcp.Description("温度值"),
				mcp.Required(),
			),
			mcp.WithString("from_unit",
				mcp.Description("源单位"),
				mcp.Required(),
				mcp.Enum("celsius", "fahrenheit", "kelvin"),
			),
			mcp.WithString("to_unit",
				mcp.Description("目标单位"),
				mcp.Required(),
				mcp.Enum("celsius", "fahrenheit", "kelvin"),
			),
		),
		convertTemperatureHandler,
	)

	// 注册资源
	s.AddResource(
		mcp.NewResource(
			"weather://supported-cities",
			"支持的城市列表",
			mcp.WithMIMEType("application/json"),
		),
		getSupportedCitiesHandler,
	)

	// 启动服务器
	fmt.Fprintln(os.Stderr, "🌤️  天气 MCP 服务器启动 (mark3labs SDK)")
	fmt.Fprintf(os.Stderr, "支持 %d 个城市\n", len(weatherDatabase))

	if err := server.ServeStdio(s); err != nil {
		fmt.Fprintf(os.Stderr, "服务器错误：%v\n", err)
		os.Exit(1)
	}
}

// ============== 工具处理函数 ==============

// getWeatherHandler 获取天气处理函数
func getWeatherHandler(ctx context.Context, request mcp.CallToolRequest) (*mcp.CallToolResult, error) {
	city, ok := request.Arguments["city"].(string)
	if !ok {
		return mcp.NewToolResultText("参数错误：city 必须是字符串"), nil
	}

	cityLower := strings.ToLower(city)
	data, exists := weatherDatabase[cityLower]
	if !exists {
		available := make([]string, 0, len(weatherDatabase))
		for k := range weatherDatabase {
			available = append(available, k)
		}
		return mcp.NewToolResultText(
			fmt.Sprintf(`城市 "%s" 未找到。可用城市：%s`, city, strings.Join(available, ", ")),
		), nil
	}

	result := fmt.Sprintf(
		"天气信息：\n城市：%s\n温度：%.1f°C\n状况：%s\n湿度：%.0f%%",
		city, data.Temperature, data.Condition, data.Humidity,
	)

	return mcp.NewToolResultText(result), nil
}

// convertTemperatureHandler 温度转换处理函数
func convertTemperatureHandler(ctx context.Context, request mcp.CallToolRequest) (*mcp.CallToolResult, error) {
	value, ok := request.Arguments["value"].(float64)
	if !ok {
		return mcp.NewToolResultText("参数错误：value 必须是数字"), nil
	}

	fromUnit, ok := request.Arguments["from_unit"].(string)
	if !ok {
		return mcp.NewToolResultText("参数错误：from_unit 必须是字符串"), nil
	}

	toUnit, ok := request.Arguments["to_unit"].(string)
	if !ok {
		return mcp.NewToolResultText("参数错误：to_unit 必须是字符串"), nil
	}

	// 转换逻辑
	var celsius float64
	switch fromUnit {
	case "celsius":
		celsius = value
	case "fahrenheit":
		celsius = (value - 32) * 5 / 9
	case "kelvin":
		celsius = value - 273.15
	}

	var result float64
	switch toUnit {
	case "celsius":
		result = celsius
	case "fahrenheit":
		result = celsius*9/5 + 32
	case "kelvin":
		result = celsius + 273.15
	}

	return mcp.NewToolResultText(
		fmt.Sprintf("%.2f %s = %.2f %s", value, fromUnit, result, toUnit),
	), nil
}

// ============== 资源处理函数 ==============

// getSupportedCitiesHandler 获取支持的城市列表
func getSupportedCitiesHandler(ctx context.Context, request mcp.ReadResourceRequest) (*mcp.ReadResourceResult, error) {
	cities := make([]string, 0, len(weatherDatabase))
	for k := range weatherDatabase {
		cities = append(cities, k)
	}

	data := map[string]interface{}{"cities": cities}
	jsonData, _ := json.MarshalIndent(data, "", "  ")

	return &mcp.ReadResourceResult{
		Contents: []mcp.ResourceContent{
			{
				URI:      request.Params.URI,
				MIMEType: "application/json",
				Text:     string(jsonData),
			},
		},
	}, nil
}
```

### 5.4 两种 Go SDK 对比

| 特性 | 官方 SDK | mark3labs SDK |
|------|---------|--------------|
| **API 简洁度** | 较繁琐 | ✅ 简洁直观 |
| **工具注册** | 需要手动处理 | ✅ 链式调用 |
| **类型安全** | ✅ 完整 | ✅ 完整 |
| **文档** | ✅ 官方 | ⚠️ 社区 |
| **更新频率** | 中 | ✅ 高 |
| **社区支持** | ⚠️ 一般 | ✅ 活跃 |
| **示例代码** | 较少 | ✅ 丰富 |

**推荐使用场景：**
- ✅ **新项目**推荐使用 **mark3labs/mcp-go**（更简洁）
- ✅ **需要官方支持**使用 **modelcontextprotocol/go-sdk**

---

## 六、Rust 实现

### 6.1 环境准备

```bash
# 创建项目
cargo new mcp-weather-server
cd mcp-weather-server

# 添加依赖
# rmcp 是官方 Rust SDK，470 万 + 下载量
cargo add rmcp rmcp-macros
cargo add tokio serde serde_json schemars
```

### 6.2 使用 rmcp 构建天气服务器

```rust
// MCP 天气服务器 - Rust 实现
// 使用 rmcp (官方 Rust SDK)

use rmcp::{
    handler::server::router::tool::ToolRouter,
    model::*,
    service::server::{ServerHandler, ServerInfo},
    tool, ToolHandler,
};
use serde::{Deserialize, Serialize};
use serde_json::json;
use std::collections::HashMap;
use std::sync::Arc;
use tokio::sync::RwLock;

// 天气数据结构
#[derive(Debug, Clone, Serialize, Deserialize)]
struct WeatherData {
    temperature: f64,
    condition: String,
    humidity: f64,
}

// 服务器状态
#[derive(Debug, Clone)]
struct WeatherServer {
    weather_database: Arc<RwLock<HashMap<String, WeatherData>>>,
}

impl WeatherServer {
    fn new() -> Self {
        let mut database = HashMap::new();
        database.insert(
            "beijing".to_string(),
            WeatherData {
                temperature: 25.0,
                condition: "晴朗".to_string(),
                humidity: 45.0,
            },
        );
        database.insert(
            "shanghai".to_string(),
            WeatherData {
                temperature: 28.0,
                condition: "多云".to_string(),
                humidity: 60.0,
            },
        );
        database.insert(
            "guangzhou".to_string(),
            WeatherData {
                temperature: 32.0,
                condition: "小雨".to_string(),
                humidity: 75.0,
            },
        );
        database.insert(
            "shenzhen".to_string(),
            WeatherData {
                temperature: 30.0,
                condition: "雷阵雨".to_string(),
                humidity: 80.0,
            },
        );

        Self {
            weather_database: Arc::new(RwLock::new(database)),
        }
    }
}

impl Default for WeatherServer {
    fn default() -> Self {
        Self::new()
    }
}

// ============== 工具实现 ==============

#[tool(tool_box)]
impl WeatherServer {
    /// 获取指定城市的当前天气信息
    #[tool(description = "获取指定城市的当前天气信息")]
    async fn get_weather(&self, city: String) -> Result<String, ServerError> {
        let city_lower = city.to_lowercase();
        let database = self.weather_database.read().await;

        if let Some(data) = database.get(&city_lower) {
            Ok(format!(
                "天气信息：\n城市：{}\n温度：{:.1}°C\n状况：{}\n湿度：{:.0}%",
                city, data.temperature, data.condition, data.humidity
            ))
        } else {
            let available: Vec<&String> = database.keys().collect();
            Ok(format!(
                "城市 \"{}\" 未找到。可用城市：{}",
                city,
                available.join(", ")
            ))
        }
    }

    /// 在不同温度单位之间转换
    #[tool(description = "在不同温度单位之间转换")]
    async fn convert_temperature(
        &self,
        #[tool(description = "温度值")] value: f64,
        #[tool(description = "源单位 (celsius/fahrenheit/kelvin)")] from_unit: String,
        #[tool(description = "目标单位 (celsius/fahrenheit/kelvin)")] to_unit: String,
    ) -> Result<String, ServerError> {
        // 先转换为摄氏度
        let celsius = match from_unit.as_str() {
            "celsius" => value,
            "fahrenheit" => (value - 32.0) * 5.0 / 9.0,
            "kelvin" => value - 273.15,
            _ => return Err(ServerError::invalid_params("无效的温度单位", None)),
        };

        // 再从摄氏度转换为目标单位
        let result = match to_unit.as_str() {
            "celsius" => celsius,
            "fahrenheit" => celsius * 9.0 / 5.0 + 32.0,
            "kelvin" => celsius + 273.15,
            _ => return Err(ServerError::invalid_params("无效的温度单位", None)),
        };

        Ok(format!("{:.2} {} = {:.2} {}", value, from_unit, result, to_unit))
    }

    /// 获取支持的城市列表
    #[tool(description = "获取所有支持天气查询的城市列表")]
    async fn get_supported_cities(&self) -> Result<String, ServerError> {
        let database = self.weather_database.read().await;
        let cities: Vec<&String> = database.keys().collect();
        Ok(format!("支持的城市：{}", cities.join(", ")))
    }
}

// ============== 服务器处理器 ==============

impl ServerHandler for WeatherServer {
    fn get_info(&self) -> ServerInfo {
        ServerInfo {
            instructions: Some("天气查询 MCP 服务器".into()),
            ..Default::default()
        }
    }
}

// ============== 主函数 ==============

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    eprintln!("🌤️  天气 MCP 服务器启动 (Rust/rmcp)");

    let server = WeatherServer::new();
    let service = server.serve();

    // 使用 stdio 传输
    let transport = rmcp::transport::stdio();
    service.run(transport).await?;

    Ok(())
}
```

### 6.3 使用宏简化实现

```rust
// 使用 rmcp 宏简化代码
use rmcp::{tool, ServerHandler, ServerInfo};
use serde::{Deserialize, Serialize};

// 定义请求结构
#[derive(Debug, Deserialize, schemars::JsonSchema)]
pub struct WeatherRequest {
    #[schemars(description = "城市名称")]
    pub city: String,
}

#[derive(Debug, Deserialize, schemars::JsonSchema)]
pub struct TemperatureConvertRequest {
    #[schemars(description = "温度值")]
    pub value: f64,
    #[schemars(description = "源单位")]
    pub from_unit: String,
    #[schemars(description = "目标单位")]
    pub to_unit: String,
}

// 创建工具盒
#[tool(tool_box)]
impl WeatherTools {
    #[tool(description = "获取城市天气")]
    async fn get_weather(&self, req: WeatherRequest) -> String {
        // 实现逻辑
        format!("{} 的天气信息", req.city)
    }

    #[tool(description = "温度单位转换")]
    async fn convert_temperature(&self, req: TemperatureConvertRequest) -> String {
        // 实现逻辑
        format!("{} {} = {} {}", req.value, req.from_unit, req.value, req.to_unit)
    }
}
```

### 6.4 Cargo.toml 配置

```toml
[package]
name = "mcp-weather-server"
version = "1.0.0"
edition = "2021"
description = "MCP 天气服务器 - Rust 实现"

[dependencies]
rmcp = { version = "1.2", features = ["server", "transport-stdio"] }
rmcp-macros = "1.2"
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
schemars = "1"
anyhow = "1"

[profile.release]
lto = true
codegen-units = 1
```

### 6.5 Rust 实现优势

| 特性 | Python | TypeScript | Go | Rust |
|------|--------|-----------|-----|------|
| **性能** | ⚠️ 慢 | ⚠️ 中 | ✅ 快 | ✅✅ 最快 |
| **内存占用** | ⚠️ 高 | ⚠️ 中 | ✅ 低 | ✅✅ 最低 |
| **类型安全** | ⚠️ 运行时 | ✅ 编译时 | ✅ 编译时 | ✅✅ 最强 |
| **启动速度** | ⚠️ 慢 | ⚠️ 中 | ✅ 快 | ✅✅ 最快 |
| **开发效率** | ✅✅ 最快 | ✅ 快 | ✅ 快 | ⚠️ 较慢 |
| **部署简单** | ✅ 简单 | ⚠️ 需 Node | ✅✅ 单二进制 | ✅✅ 单二进制 |

**Rust 适用场景：**
- ✅ 高性能要求
- ✅ 资源受限环境
- ✅ 需要单二进制部署
- ✅ 对安全性要求高

---

## 七、四种语言对比

### 7.1 代码量对比

实现相同功能（天气查询 + 温度转换）：

| 语言 | 代码行数 | 文件大小 | 依赖数量 |
|------|---------|---------|---------|
| **Python** | ~80 行 | ~3KB | 2-3 个 |
| **TypeScript** | ~120 行 | ~4KB | 2-3 个 |
| **Go (官方)** | ~150 行 | ~5KB | 1 个 |
| **Go (mark3labs)** | ~100 行 | ~4KB | 1 个 |
| **Rust** | ~180 行 | ~6KB | 5-6 个 |

### 7.2 性能对比

| 语言 | 启动时间 | 内存占用 | 请求延迟 | 二进制大小 |
|------|:-------:|:-------:|:-------:|:----------:|
| **Python** | ~500ms | ~50MB | ~10ms | N/A |
| **TypeScript** | ~200ms | ~30MB | ~5ms | N/A |
| **Go** | ~10ms | ~10MB | ~1ms | ~10MB |
| **Rust** | ~1ms | ~5MB | ~0.5ms | ~2MB |

### 7.3 选择建议

| 场景 | 推荐语言 | 理由 |
|------|---------|------|
| **快速原型** | Python | 开发最快，生态丰富 |
| **Web 全栈** | TypeScript | 前后端统一，类型安全 |
| **生产服务** | Go (mark3labs) | 平衡性能和开发效率 |
| **高性能** | Rust | 最佳性能，最小资源 |
| **企业应用** | Go/TypeScript | 人才储备充足 |
| **边缘部署** | Rust | 最小二进制，最低资源 |

---

## 八、生产级部署

### 8.1 Docker 部署

#### Python 版本

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY weather_server.py .

ENV PYTHONUNBUFFERED=1

CMD ["python", "weather_server.py"]
```

#### TypeScript 版本

```dockerfile
FROM denoland/deno:2.1.4

WORKDIR /app

COPY deno.json deno.lock ./
RUN deno install

COPY . .
RUN deno cache mod.ts

EXPOSE 3000

CMD ["deno", "run", "--allow-net", "--allow-read", "mod.ts"]
```

#### Go 版本

```dockerfile
FROM golang:1.21-alpine AS builder

WORKDIR /app
COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o mcp-server .

FROM alpine:latest
RUN apk --no-cache add ca-certificates

COPY --from=builder /app/mcp-server .

CMD ["./mcp-server"]
```

#### Rust 版本

```dockerfile
FROM rust:1.75 AS builder

WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src

RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates && rm -rf /var/lib/apt/lists/*

COPY --from=builder /app/target/release/mcp-weather-server .

CMD ["./mcp-weather-server"]
```

### 8.2 docker-compose.yml

```yaml
version: "3.8"

services:
  # Python MCP 服务器
  mcp-python:
    build: ./python
    ports:
      - "8001:8000"
    environment:
      - WEATHER_API_KEY=${WEATHER_API_KEY}
    restart: unless-stopped

  # TypeScript MCP 服务器
  mcp-typescript:
    build: ./typescript
    ports:
      - "8002:3000"
    restart: unless-stopped

  # Go MCP 服务器
  mcp-go:
    build: ./go
    ports:
      - "8003:8080"
    restart: unless-stopped

  # Rust MCP 服务器
  mcp-rust:
    build: ./rust
    ports:
      - "8004:8080"
    restart: unless-stopped

  # Nginx 反向代理
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - mcp-python
      - mcp-typescript
      - mcp-go
      - mcp-rust
    restart: unless-stopped
```

---

## 九、测试和调试

### 9.1 MCP Inspector

MCP Inspector 是官方提供的调试工具：

```bash
# 安装
npx @modelcontextprotocol/inspector

# 运行（Python 服务器）
npx @modelcontextprotocol/inspector python weather_server.py

# 运行（TypeScript 服务器）
npx @modelcontextprotocol/inspector node build/weather_server.js

# 运行（Go 服务器）
npx @modelcontextprotocol/inspector go run main.go

# 运行（Rust 服务器）
npx @modelcontextprotocol/inspector cargo run
```

访问 http://localhost:5173 查看交互式界面。

### 9.2 单元测试

#### Python (pytest)

```python
import pytest
from weather_server import mcp, WEATHER_DATABASE

@pytest.mark.asyncio
async def test_get_weather_existing_city():
    result = await mcp.get_weather("beijing")
    assert "25" in result

@pytest.mark.asyncio
async def test_convert_temperature():
    result = await mcp.convert_temperature(0, "celsius", "fahrenheit")
    assert result == 32.0
```

#### Go (testing)

```go
func TestGetWeather(t *testing.T) {
    result, err := getWeatherHandler(context.Background(), map[string]interface{}{
        "city": "beijing",
    })
    if err != nil {
        t.Fatalf("Unexpected error: %v", err)
    }
    if !strings.Contains(result.Content[0].(mcp.TextContent).Text, "25") {
        t.Error("Expected temperature 25")
    }
}
```

#### Rust (cargo test)

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_get_weather() {
        let server = WeatherServer::new();
        let result = server.get_weather("beijing".to_string()).await.unwrap();
        assert!(result.contains("25"));
    }

    #[tokio::test]
    async fn test_convert_temperature() {
        let server = WeatherServer::new();
        let result = server.convert_temperature(0.0, "celsius".to_string(), "fahrenheit".to_string()).await.unwrap();
        assert!(result.contains("32"));
    }
}
```

---

## 十、Claude Desktop 集成

### 10.1 配置 Claude Desktop

#### macOS 配置

```json
// ~/Library/Application Support/Claude/claude_desktop_config.json
{
  "mcpServers": {
    "weather-python": {
      "command": "python",
      "args": ["/path/to/weather_server.py"]
    },
    "weather-ts": {
      "command": "node",
      "args": ["/path/to/build/weather_server.js"]
    },
    "weather-go": {
      "command": "/path/to/mcp-weather-server-go"
    },
    "weather-rust": {
      "command": "/path/to/mcp-weather-server"
    }
  }
}
```

### 10.2 在 Claude 中使用

配置完成后，重启 Claude Desktop，然后可以直接询问：

```
北京今天的天气如何？

帮我把 25 摄氏度转换为华氏度

支持哪些城市的天气查询？
```

Claude 会自动发现并调用你的 MCP 服务器工具。

---

## 十一、最佳实践

### 11.1 工具设计原则

```python
# ✅ 推荐：单一职责
@mcp.tool()
async def get_weather(city: str) -> str:
    """获取天气"""

@mcp.tool()
async def get_forecast(city: str, days: int) -> str:
    """获取预报"""

# ❌ 避免：多功能工具
@mcp.tool()
async def handle_weather(city: str, action: str, days: int) -> str:
    """处理天气相关（获取、预报、警告...）"""
```

### 11.2 错误处理

```python
# ✅ 推荐：明确的错误处理
@mcp.tool()
async def get_weather(city: str) -> str:
    try:
        return result
    except CityNotFoundError as e:
        return f"错误：{str(e)}"
    except Exception as e:
        logger.error(f"意外错误：{e}")
        return "发生错误，请稍后重试"
```

### 11.3 输入验证

```python
from pydantic import BaseModel, Field, validator

class WeatherInput(BaseModel):
    city: str = Field(..., min_length=1, max_length=100)
    
    @validator("city")
    def validate_city(cls, v):
        if not v.replace(" ", "").isalpha():
            raise ValueError("城市名只能包含字母和空格")
        return v
```

### 11.4 安全考虑

```python
# ✅ 推荐：参数化查询
@mcp.tool()
async def query_database(sql_query: str, params: list) -> str:
    cursor.execute(sql_query, params)  # 参数化防止 SQL 注入

# ❌ 避免：字符串拼接
@mcp.tool()
async def query_database(sql_query: str) -> str:
    cursor.execute(f"SELECT * FROM {sql_query}")  # SQL 注入风险
```

---

## 十二、总结

### 12.1 语言选择建议

| 语言 | SDK | 优势 | 适用场景 |
|------|-----|------|---------|
| **Python** | FastMCP/mcp | 开发最快，生态丰富 | 快速原型、数据科学 |
| **TypeScript** | 官方 SDK | 类型安全，前端友好 | Web 应用、全栈项目 |
| **Go** | mark3labs/mcp-go | 性能优秀，部署简单 | 生产服务、微服务 |
| **Go** | 官方 SDK | 官方支持 | 企业应用 |
| **Rust** | rmcp | 最佳性能，最小资源 | 高性能、边缘部署 |

### 12.2 学习资源

- **官方文档**：https://modelcontextprotocol.io/
- **GitHub**：https://github.com/modelcontextprotocol
- **示例服务器**：https://github.com/modelcontextprotocol/servers
- **MCP Inspector**：调试工具
- **Rust SDK**：https://github.com/4t145/rmcp
- **Go SDK**：https://github.com/mark3labs/mcp-go

### 12.3 下一步

1. **从简单开始**：先实现一个工具，逐步扩展
2. **使用 Inspector**：充分利用调试工具
3. **参考示例**：学习官方和社区示例
4. **加入社区**：参与讨论和贡献

MCP 正在成为 AI Agent 与外部工具通信的标准协议。掌握 MCP 开发，你将能够构建可复用、安全、标准化的 AI 工具，让任何 MCP 兼容的 AI 客户端都能使用你的服务。开始构建你的第一个 MCP 服务器吧！🚀
