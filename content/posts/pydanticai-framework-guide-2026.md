+++
title = "PydanticAI 开发框架完全指南 2026：类型安全的 AI Agent 实战"
date = "2026-03-18T18:00:00+08:00"

[taxonomies]
tags = ["PydanticAI", "AI Agent", "Python", "类型安全", "结构化输出"]
categories = ["人工智能"]

[extra]
summary = "全面介绍 PydanticAI 类型安全 AI Agent 框架，包含核心概念、结构化输出、工具注册、依赖注入、MCP 集成、可观测性等完整知识，附带丰富的实战代码示例和生产级部署方案。"
author = "博主"
+++

在 AI Agent 开发领域，**PydanticAI** 以其类型安全和开发者友好的特性迅速崛起。作为 Pydantic 团队打造的新一代 Agent 框架，PydanticAI 将 FastAPI 的开发体验带入了生成式 AI 世界，让构建生产级 AI 应用变得简单、可靠、可维护。

本文将深入介绍 PydanticAI 的核心概念、使用方法和最佳实践，通过完整的代码示例，帮助你掌握构建类型安全 AI Agent 的关键技术。

---

## 一、什么是 PydanticAI？

### PydanticAI 简介

**PydanticAI** 是一个 Python Agent 框架，专为构建生产级生成式 AI 应用而设计。它于 2025 年 9 月发布 v1.0 稳定版本，由 Pydantic 团队（同样打造了被 OpenAI、Google、Anthropic 等广泛使用的 Pydantic 库）开发。

**核心理念**：将类型安全、依赖注入和结构化输出带入 AI Agent 开发，让错误在开发时发现，而不是在生产环境中。

**官方网站**：https://ai.pydantic.dev/

**GitHub**：https://github.com/pydantic/pydantic-ai（15.5K+ Stars）

**PyPI**：https://pypi.org/project/pydantic-ai/

### 核心特性

| 特性 | 描述 |
|------|------|
| **类型安全** | 使用 Pydantic 模型定义输入输出，自动验证 |
| **结构化输出** | 强制 LLM 返回符合 schema 的 JSON |
| **依赖注入** | 类型安全的运行时上下文注入 |
| **多模型支持** | 15+ 提供商，切换无需改代码 |
| **工具注册** | 简单的装饰器注册 Python 函数为工具 |
| **流式响应** | 实时流式输出，带即时验证 |
| **MCP 集成** | 原生支持 Model Context Protocol |
| **可观测性** | 集成 Pydantic Logfire 追踪 |

### PydanticAI vs 其他框架

| 框架 | 类型安全 | 结构化输出 | 学习曲线 | 生态系统 | 适用场景 |
|------|---------|-----------|---------|---------|---------|
| **PydanticAI** | ✅ 强类型 | ✅ 自动验证 | 低 | 成长中 | 生产级 Agent |
| **LangChain** | ⚠️ 需手动 | ⚠️ 需 OutputParser | 中高 | 成熟 | RAG、复杂工作流 |
| **CrewAI** | ⚠️ 部分 | ⚠️ 部分 | 低 | 成长中 | 多 Agent 协作 |
| **AutoGen** | ⚠️ 部分 | ⚠️ 部分 | 中 | 成熟 | 对话式协作 |

### 适用场景

- ✅ **需要结构化输出**：API 后端、数据处理管道
- ✅ **类型安全要求高**：金融、医疗等关键领域
- ✅ **需要快速原型**：简单 Agent、单工作流应用
- ✅ **测试驱动开发**：需要模拟和单元测试
- ⚠️ **复杂 RAG 系统**：LangChain/LlamaIndex 更合适
- ⚠️ **多 Agent 编排**：CrewAI/AutoGen 更强大

---

## 二、快速开始

### 2.1 环境要求

- **Python**: 3.9 - 3.13
- **Pydantic**: 2.0+
- **操作系统**: Linux / macOS / Windows

### 2.2 安装 PydanticAI

```bash
# 创建虚拟环境
python -m venv pydanticai-env
source pydanticai-env/bin/activate  # Windows: pydanticai-env\Scripts\activate

# 安装核心包
pip install pydantic-ai

# 安装常用工具
pip install "pydantic-ai[duckduckgo,tavily]"

# 安装异步支持（Jupyter 等）
pip install nest-asyncio

# 安装可观测性
pip install logfire
```

### 2.3 配置 API 密钥

创建 `.env` 文件：

```bash
# .env
OPENAI_API_KEY=sk-your-openai-api-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key
GOOGLE_API_KEY=your-google-api-key
LOGFIRE_API_KEY=your-logfire-key  # 可选，用于追踪
```

在代码中加载：

```python
from dotenv import load_dotenv
load_dotenv()
```

### 2.4 第一个 Agent

```python
from pydantic_ai import Agent

# 创建 Agent
agent = Agent(
    'openai:gpt-4o',  # 模型
    instructions='简洁回答，一句话即可。'  # 系统指令
)

# 运行 Agent
result = agent.run_sync('"Hello World"这个短语来自哪里？')
print(result.output)
# 输出："Hello World"首次出现在 1974 年关于 C 编程语言的教科书中。
```

### 2.5 异步版本

```python
import asyncio
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o')

async def main():
    result = await agent.run('解释量子计算')
    print(result.output)

asyncio.run(main())
```

---

## 三、核心概念

### 3.1 Agent（Agent）

Agent 是 PydanticAI 的核心抽象。

#### 基础 Agent

```python
from pydantic_ai import Agent

# 简单 Agent
agent = Agent(
    'openai:gpt-4o',
    instructions='你是一个乐于助人的助手。',
    name='助手',
    retries=2  # 失败重试次数
)

# 带系统提示的 Agent
@agent.system_prompt
def get_system_prompt():
    return f"当前日期：{datetime.now().strftime('%Y-%m-%d')}"
```

#### 模型提供商

```python
# OpenAI
agent = Agent('openai:gpt-4o')
agent = Agent('openai:gpt-4o-mini')
agent = Agent('openai:o3-mini')

# Anthropic
agent = Agent('anthropic:claude-sonnet-4-20250514')
agent = Agent('anthropic:claude-3-5-sonnet-20241022')

# Google Gemini
agent = Agent('google-gla:gemini-2.0-flash')
agent = Agent('google-gla:gemini-2.5-pro')

# DeepSeek
agent = Agent('deepseek:deepseek-chat')

# Ollama（本地）
agent = Agent('ollama:llama3.1:8b', base_url='http://localhost:11434')

# 其他提供商
agent = Agent('grok:grok-2')
agent = Agent('cohere:command-r-plus')
agent = Agent('mistral:mistral-large')
agent = Agent('azure-ai:gpt-4')
agent = Agent('bedrock:anthropic.claude-3-sonnet-20240229-v1:0')
```

### 3.2 结构化输出（Structured Output）

PydanticAI 的核心优势：强制 LLM 返回符合 Pydantic 模型的数据。

#### 基础结构化输出

```python
from pydantic import BaseModel, Field
from pydantic_ai import Agent

# 定义输出模型
class WeatherReport(BaseModel):
    temperature: float = Field(description="温度（摄氏度）")
    condition: str = Field(description="天气状况")
    humidity: float = Field(description="湿度百分比")
    forecast: str = Field(description="天气预报")

# 创建带结构化输出的 Agent
agent = Agent(
    'openai:gpt-4o',
    output_type=WeatherReport,  # 指定输出类型
    instructions='你是天气预报助手。'
)

# 运行
result = agent.run_sync('北京今天的天气如何？')

# 访问结构化数据
print(f"温度：{result.output.temperature}°C")
print(f"状况：{result.output.condition}")
print(f"湿度：{result.output.humidity}%")
print(f"预报：{result.output.forecast}")

# 类型检查通过
weather: WeatherReport = result.output
```

#### 复杂嵌套模型

```python
from typing import List, Optional, Literal
from pydantic import BaseModel, Field

class Address(BaseModel):
    street: str = Field(description="街道地址")
    city: str = Field(description="城市")
    country: str = Field(description="国家")
    postal_code: str = Field(description="邮政编码")

class Person(BaseModel):
    name: str = Field(description="姓名")
    age: int = Field(ge=0, le=150, description="年龄")
    email: str = Field(description="邮箱地址")
    addresses: List[Address] = Field(description="地址列表")
    occupation: Optional[str] = Field(None, description="职业")
    risk_level: Literal["low", "medium", "high"] = Field(description="风险等级")

agent = Agent(
    'openai:gpt-4o',
    output_type=Person,
    instructions='从文本中提取个人信息。'
)

result = agent.run_sync("""
张三，35 岁，邮箱 zhangsan@example.com。
住在北京市朝阳区建国路 100 号，邮编 100000。
他是一名软件工程师。
""")

print(result.output.name)  # 张三
print(result.output.addresses[0].city)  # 北京市
```

#### 输出验证器

```python
from pydantic_ai import Agent, RunContext, ModelRetry
from pydantic import BaseModel

class WeatherReport(BaseModel):
    temperature: float
    condition: str

agent = Agent('openai:gpt-4o', output_type=WeatherReport)

# 添加输出验证器
@agent.output_validator
async def validate_temperature(ctx: RunContext, output: WeatherReport) -> WeatherReport:
    """验证温度在合理范围内"""
    if output.temperature < -100 or output.temperature > 150:
        raise ModelRetry("温度超出合理范围，请重新生成")
    return output

result = agent.run_sync('火星表面的温度')
```

#### 输出模式对比

| 模式 | 描述 | 适用场景 |
|------|------|---------|
| **ToolOutput**（默认） | 使用函数调用提取数据 | 最可靠，跨提供商 |
| **NativeOutput** | 使用模型内置 JSON Schema | OpenAI Structured Outputs |
| **PromptedOutput** | 依赖提示词引导格式 | 所有模型，较不严格 |

```python
from pydantic_ai import Agent
from pydantic_ai.output import NativeOutput, PromptedOutput

# 使用原生输出（OpenAI）
agent = Agent(
    'openai:gpt-4o',
    output_type=MyModel,
    output_mode=NativeOutput()
)

# 使用提示词输出（兼容所有模型）
agent = Agent(
    'ollama:llama3.1',
    output_type=MyModel,
    output_mode=PromptedOutput()
)
```

### 3.3 工具（Tools）

工具让 Agent 能够执行具体操作。

#### 注册工具

```python
from pydantic_ai import Agent, RunContext
from typing import Annotated

agent = Agent('openai:gpt-4o')

# 简单工具
@agent.tool
def calculate_area(
    length: Annotated[float, "矩形长度"],
    width: Annotated[float, "矩形宽度"]
) -> str:
    """计算矩形面积"""
    area = length * width
    return f"面积：{area} 平方米"

# 带上下文的工具（访问依赖）
@agent.tool
async def get_weather(
    ctx: RunContext,
    city: Annotated[str, "城市名称"]
) -> str:
    """获取城市天气"""
    api_key = ctx.deps.weather_api_key
    # 调用天气 API
    return f"{city}的天气：晴朗，25°C"

# 不带上下文的工具
@agent.tool_plain
def add(a: float, b: float) -> str:
    """计算两个数的和"""
    return str(a + b)
```

#### 内置工具

```python
from pydantic_ai import Agent
from pydantic_ai.common_tools import (
    DuckDuckGoSearchTool,
    TavilySearchTool,
    WebSearchTool
)

# DuckDuckGo 搜索
agent = Agent(
    'openai:gpt-4o',
    tools=[DuckDuckGoSearchTool()]
)

# Tavily 搜索（更适合 AI）
agent = Agent(
    'openai:gpt-4o',
    tools=[TavilySearchTool()]
)

# 通用网页搜索
agent = Agent(
    'openai:gpt-4o',
    tools=[WebSearchTool()]
)

result = agent.run_sync('2026 年 AI 领域的最新进展')
```

#### 工具重试

```python
from pydantic_ai import Agent, RunContext

agent = Agent('openai:gpt-4o')

@agent.tool(retries=2)  # 失败重试 2 次
async def query_database(
    ctx: RunContext,
    sql_query: str
) -> str:
    """执行只读 SQL 查询"""
    # 可能失败的数据库查询
    return "查询结果"
```

### 3.4 依赖注入（Dependency Injection）

PydanticAI 的依赖注入系统让 Agent 可测试、可维护。

#### 基础依赖注入

```python
from pydantic_ai import Agent, RunContext
from pydantic import BaseModel
from dataclasses import dataclass

# 定义依赖类型
@dataclass
class WeatherDeps:
    api_key: str
    api_base_url: str

# 创建 Agent（指定依赖类型）
agent = Agent(
    'openai:gpt-4o',
    deps_type=WeatherDeps
)

@agent.tool
async def get_weather(
    ctx: RunContext[WeatherDeps],  # 类型安全的上下文
    city: str
) -> str:
    """获取天气"""
    # 访问依赖
    api_key = ctx.deps.api_key
    base_url = ctx.deps.api_base_url
    
    # 调用 API
    return f"{city}的天气信息"

# 运行时注入依赖
deps = WeatherDeps(
    api_key="your-api-key",
    api_base_url="https://api.weather.com"
)

result = agent.run_sync('北京天气', deps=deps)
```

#### 复杂依赖

```python
from dataclasses import dataclass
from typing import Optional
import httpx

@dataclass
class DatabaseConnection:
    async def get_user(self, user_id: int) -> dict:
        return {"id": user_id, "name": "张三"}
    
    async def get_balance(self, user_id: int) -> float:
        return 1000.0

@dataclass
class SupportDeps:
    db: DatabaseConnection
    customer_id: int
    api_key: str
    http_client: httpx.AsyncClient

agent = Agent(
    'openai:gpt-4o',
    deps_type=SupportDeps,
    instructions='你是银行客服助手。'
)

@agent.tool
async def get_balance(ctx: RunContext[SupportDeps]) -> str:
    """获取客户余额"""
    balance = await ctx.deps.db.get_balance(ctx.deps.customer_id)
    return f"余额：${balance:.2f}"

@agent.system_prompt
async def add_customer_name(ctx: RunContext[SupportDeps]) -> str:
    """添加客户姓名到系统提示"""
    user = await ctx.deps.db.get_user(ctx.deps.customer_id)
    return f"客户姓名：{user['name']}"

# 运行
deps = SupportDeps(
    db=DatabaseConnection(),
    customer_id=123,
    api_key="key",
    http_client=httpx.AsyncClient()
)

result = await agent.run('我的余额是多少？', deps=deps)
```

#### 测试时的依赖覆盖

```python
from pydantic_ai import Agent
from pydantic_ai.models.test import TestModel

# 模拟依赖
class MockDatabase:
    async def get_balance(self, user_id: int) -> float:
        return 5000.0

# 测试时使用模拟依赖
async def test_agent():
    deps = SupportDeps(
        db=MockDatabase(),
        customer_id=123,
        api_key="test-key",
        http_client=None
    )
    
    # 使用 TestModel 避免 API 调用
    with agent.override(model=TestModel()):
        result = await agent.run('余额', deps=deps)
        assert "5000" in result.output
```

---

## 四、执行模式

### 4.1 同步执行

```python
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o')

# 同步运行
result = agent.run_sync('问题')
print(result.output)
```

### 4.2 异步执行

```python
import asyncio
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o')

async def main():
    result = await agent.run('问题')
    print(result.output)

asyncio.run(main())
```

### 4.3 流式响应

```python
import asyncio
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o')

async def main():
    # 流式运行
    async with agent.run_stream('写一首诗') as response:
        async for chunk in response.output_stream():
            print(chunk, end='', flush=True)

asyncio.run(main())
```

### 4.4 迭代执行（高级）

```python
import asyncio
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o')

async def main():
    # 访问执行图的每个节点
    async with agent.iter('解释量子计算') as run:
        async for node in run:
            print(f"节点：{node.type}")
            print(f"数据：{node.data}")
            
            # 可以在这里添加自定义逻辑
            if node.type == 'tool-call':
                print("调用工具中...")

asyncio.run(main())
```

---

## 五、高级特性

### 5.1 MCP 集成（Model Context Protocol）

```python
from pydantic_ai import Agent
from pydantic_ai.mcp import MCPServerStreamableHttp

# 创建 MCP 服务器连接
mcp_server = MCPServerStreamableHttp(
    url='https://your-mcp-server.com/mcp',
    headers={'Authorization': 'Bearer token'}
)

agent = Agent(
    'openai:gpt-4o',
    instructions='你可以使用 MCP 工具。',
    mcp_servers=[mcp_server]
)

async with agent.run_mcp_servers():
    result = await agent.run('查询数据库中的用户信息')
    print(result.output)
```

### 5.2 多 Agent 协作

```python
from pydantic_ai import Agent
from pydantic import BaseModel

class ResearchResult(BaseModel):
    findings: list[str]
    sources: list[str]

class AnalysisResult(BaseModel):
    insights: list[str]
    recommendations: list[str]

# 研究 Agent
researcher = Agent(
    'openai:gpt-4o',
    output_type=ResearchResult,
    instructions='研究主题并收集信息。'
)

# 分析 Agent
analyst = Agent(
    'openai:gpt-4o',
    output_type=AnalysisResult,
    instructions='基于研究结果提供洞察。'
)

async def multi_agent_workflow(topic: str):
    # 第一步：研究
    research_result = await researcher.run(f'研究{topic}')
    
    # 第二步：分析（使用前一步的结果）
    analysis_result = await analyst.run(
        f'基于以下研究发现提供分析：{research_result.output}'
    )
    
    return analysis_result

result = asyncio.run(multi_agent_workflow('AI 发展趋势'))
print(result.output.insights)
```

### 5.3 持久化执行（Durable Execution）

```python
from pydantic_ai import Agent
from pydantic_ai.temporal import TemporalToolset

agent = Agent('openai:gpt-4o')

# 使用 Temporal 实现持久化执行
async def durable_workflow(user_request: str):
    async with TemporalToolset(agent) as durable_agent:
        result = await durable_agent.run(user_request)
        return result

# 即使进程重启，工作流也会继续
result = asyncio.run(durable_workflow('处理复杂任务'))
```

### 5.4 可观测性（Logfire 集成）

```python
from pydantic_ai import Agent
import logfire

# 配置 Logfire
logfire.configure(api_key='your-api-key')

# 创建带追踪的 Agent
agent = Agent(
    'openai:gpt-4o',
    instrument=True  # 启用 OpenTelemetry 追踪
)

# 运行（自动记录到 Logfire）
result = agent.run_sync('问题')

# 在 Logfire dashboard 查看追踪
# https://logfire.pydantic.dev
```

### 5.5 批量处理

```python
import asyncio
from pydantic_ai import Agent

agent = Agent('openai:gpt-4o')

async def batch_process(questions: list[str]):
    # 并发执行多个查询
    tasks = [agent.run(q) for q in questions]
    results = await asyncio.gather(*tasks)
    return [r.output for r in results]

questions = ['问题 1', '问题 2', '问题 3']
results = asyncio.run(batch_process(questions))
```

---

## 六、实战项目

### 6.1 客户支持 Agent

```python
"""
银行客户支持 Agent - 类型安全、可测试
"""

import asyncio
from dataclasses import dataclass
from typing import Literal
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext, ModelRetry
import httpx

# 定义依赖
@dataclass
class SupportDeps:
    customer_id: int
    db: object  # 数据库连接
    api_key: str
    http_client: httpx.AsyncClient

# 定义结构化输出
class SupportResponse(BaseModel):
    category: Literal["账户", "交易", "贷款", "其他"] = Field(
        description="问题类别"
    )
    priority: Literal["low", "medium", "high"] = Field(
        description="优先级"
    )
    escalate: bool = Field(
        description="是否需要升级给人工客服"
    )
    support_advice: str = Field(
        description="给客户的建议",
        max_length=500
    )
    risk_level: int = Field(
        description="风险等级 1-10",
        ge=1,
        le=10
    )

# 创建 Agent
support_agent = Agent(
    'openai:gpt-4o',
    deps_type=SupportDeps,
    output_type=SupportResponse,
    instructions="""你是银行客服助手。
    提供有帮助的客户支持，并评估查询的风险等级。
    使用提供的工具获取客户信息。"""
)

# 添加工具
@support_agent.tool
async def get_customer_name(ctx: RunContext[SupportDeps]) -> str:
    """获取客户姓名"""
    customer = await ctx.deps.db.get_customer(ctx.deps.customer_id)
    return f"客户姓名：{customer['name']}"

@support_agent.tool
async def get_account_balance(ctx: RunContext[SupportDeps]) -> str:
    """获取账户余额"""
    balance = await ctx.deps.db.get_balance(ctx.deps.customer_id)
    return f"账户余额：${balance:.2f}"

@support_agent.tool
async def get_recent_transactions(
    ctx: RunContext[SupportDeps],
    limit: int = 5
) -> str:
    """获取最近交易记录"""
    transactions = await ctx.deps.db.get_transactions(
        ctx.deps.customer_id,
        limit=limit
    )
    return f"最近{limit}笔交易：{transactions}"

# 输出验证
@support_agent.output_validator
async def validate_risk(ctx: RunContext[SupportDeps], output: SupportResponse):
    """验证风险等级合理性"""
    if output.escalate and output.risk_level < 5:
        raise ModelRetry("需要升级的问题风险等级应该>=5")
    return output

# 使用示例
async def main():
    # 模拟数据库
    class MockDB:
        async def get_customer(self, id: int):
            return {"id": id, "name": "张三"}
        async def get_balance(self, id: int):
            return 10000.0
        async def get_transactions(self, id: int, limit: int):
            return [{"amount": 100, "merchant": "超市"} for _ in range(limit)]
    
    deps = SupportDeps(
        customer_id=123,
        db=MockDB(),
        api_key="key",
        http_client=httpx.AsyncClient()
    )
    
    result = await support_agent.run(
        '我的卡丢了，怎么办？',
        deps=deps
    )
    
    print(f"类别：{result.output.category}")
    print(f"优先级：{result.output.priority}")
    print(f"升级：{result.output.escalate}")
    print(f"建议：{result.output.support_advice}")
    print(f"风险等级：{result.output.risk_level}")

if __name__ == "__main__":
    asyncio.run(main())
```

### 6.2 数据分析 Agent

```python
"""
销售数据分析 Agent - 结构化输出、多工具协作
"""

import asyncio
from dataclasses import dataclass
from typing import List
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext
import pandas as pd

# 依赖
@dataclass
class AnalysisDeps:
    database_url: str
    api_key: str

# 输出模型
class RegionalMetrics(BaseModel):
    region: str = Field(description="地区名称")
    revenue: float = Field(description="收入（美元）")
    units_sold: int = Field(description="销售数量")
    average_price: float = Field(description="平均价格")
    performance_rating: str = Field(
        description="绩效评级",
        enum=["excellent", "good", "needs_improvement"]
    )

class SalesAnalysis(BaseModel):
    total_revenue: float = Field(description="总收入")
    total_units: int = Field(description="总销量")
    regional_breakdown: List[RegionalMetrics] = Field(
        description="地区细分"
    )
    top_performer: str = Field(description="表现最佳地区")
    areas_for_improvement: List[str] = Field(
        description="需要改进的方面"
    )
    quarterly_grade: str = Field(
        description="季度等级（A-F）",
        pattern=r"^[A-F]$"
    )

# 创建 Agent
analysis_agent = Agent(
    'openai:gpt-4o',
    deps_type=AnalysisDeps,
    output_type=SalesAnalysis,
    instructions="""你是销售分析专家。
    提供详细的地区级销售分析，包含绩效评级和改进建议。"""
)

# 工具
@analysis_agent.tool
async def query_database(
    ctx: RunContext[AnalysisDeps],
    region: str
) -> str:
    """从数据库查询地区销售数据"""
    # 模拟数据库查询
    data = {
        "North": {"revenue": 385000, "units": 470},
        "South": {"revenue": 256000, "units": 334},
        "East": {"revenue": 391500, "units": 381},
        "West": {"revenue": 395500, "units": 457}
    }
    if region in data:
        d = data[region]
        return f"{region}: 收入${d['revenue']:,}, 销量{d['units']}件"
    return "未找到数据"

@analysis_agent.tool
def calculate_metrics(
    revenues: List[float],
    units: List[int]
) -> str:
    """计算总体指标"""
    total_revenue = sum(revenues)
    total_units = sum(units)
    avg_price = total_revenue / total_units if total_units > 0 else 0
    return f"总收入：${total_revenue:,.2f}, 总销量：{total_units}, 均价：${avg_price:.2f}"

@analysis_agent.tool
async def get_exchange_rate(
    ctx: RunContext[AnalysisDeps],
    from_currency: str,
    to_currency: str
) -> float:
    """获取汇率"""
    # 调用汇率 API
    return 0.92  # USD to EUR

# 运行
async def main():
    deps = AnalysisDeps(
        database_url="postgresql://...",
        api_key="key"
    )
    
    result = await analysis_agent.run(
        """分析 Q1 2025 销售表现：
        - North: $385k 收入，470 件
        - South: $256k 收入，334 件
        - East: $391.5k 收入，381 件
        - West: $395.5k 收入，457 件""",
        deps=deps
    )
    
    output = result.output
    print(f"总收入：${output.total_revenue:,.0f}")
    print(f"总销量：{output.total_units}件")
    print(f"最佳地区：{output.top_performer}")
    print(f"季度等级：{output.quarterly_grade}")
    
    print("\n地区细分:")
    for region in output.regional_breakdown:
        print(f"{region.region}: {region.performance_rating} (${region.average_price:.2f})")

if __name__ == "__main__":
    asyncio.run(main())
```

### 6.3 代码审查 Agent

```python
"""
代码审查 Agent - 流式输出、多工具
"""

import asyncio
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext
import httpx

class CodeReview(BaseModel):
    summary: str = Field(description="代码变更摘要")
    issues: list[str] = Field(description="发现的问题列表")
    suggestions: list[str] = Field(description="改进建议列表")
    approve: bool = Field(description="是否批准变更")
    severity: str = Field(
        description="严重程度",
        enum=["low", "medium", "high", "critical"]
    )

@dataclass
class ReviewDeps:
    github_token: str
    repo: str

review_agent = Agent(
    'openai:gpt-4o',
    deps_type=ReviewDeps,
    output_type=CodeReview,
    instructions="""你是资深代码审查员。
    分析 PR 变更，查找 bug、安全问题、性能问题和风格问题。
    提供具体可操作的反馈。"""
)

@review_agent.tool
async def get_pr_diff(
    ctx: RunContext[ReviewDeps],
    pr_number: int
) -> str:
    """获取 PR 差异"""
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f"https://api.github.com/repos/{ctx.deps.repo}/pulls/{pr_number}",
            headers={
                "Authorization": f"token {ctx.deps.github_token}",
                "Accept": "application/vnd.github.v3.diff"
            }
        )
        return response.text

@review_agent.tool
async def get_pr_comments(
    ctx: RunContext[ReviewDeps],
    pr_number: int
) -> str:
    """获取 PR 评论"""
    async with httpx.AsyncClient() as client:
        response = await client.get(
            f"https://api.github.com/repos/{ctx.deps.repo}/pulls/{pr_number}/comments",
            headers={"Authorization": f"token {ctx.deps.github_token}"}
        )
        comments = response.json()
        return "\n".join([c['body'] for c in comments])

async def review_pr(pr_number: int):
    deps = ReviewDeps(
        github_token="your-token",
        repo="owner/repo"
    )
    
    # 流式输出
    async with review_agent.run_stream(
        f'审查 PR #{pr_number}',
        deps=deps
    ) as response:
        async for chunk in response.output_stream():
            print(chunk, end='', flush=True)
    
    return response.output

if __name__ == "__main__":
    result = asyncio.run(review_pr(123))
    print(f"\n批准：{result.approve}")
    print(f"严重程度：{result.severity}")
```

### 6.4 发票处理 Agent（多模态）

```python
"""
发票处理 Agent - 多模态输入、嵌套模型
"""

import asyncio
import base64
from typing import List
from pydantic import BaseModel, Field
from pydantic_ai import Agent, RunContext
from dataclasses import dataclass
from openai import OpenAI

# 嵌套输出模型
class LineItem(BaseModel):
    description: str = Field(description="项目描述")
    quantity: int = Field(description="数量")
    unit_price: float = Field(description="单价")
    total_price: float = Field(description="总价")

class InvoiceExtraction(BaseModel):
    total_amount: float = Field(description="总金额")
    sender: str = Field(description="发送方")
    date: str = Field(description="日期")
    invoice_number: str = Field(description="发票编号")
    line_items: List[LineItem] = Field(description="项目列表")

@dataclass
class InvoiceDeps:
    llm_client: OpenAI
    image_path: str

# 发票提取 Agent
invoice_agent = Agent(
    'openai:gpt-4o',
    deps_type=InvoiceDeps,
    output_type=InvoiceExtraction,
    instructions='从发票图像中提取结构化信息。'
)

@invoice_agent.tool
async def extract_from_image(
    ctx: RunContext[InvoiceDeps]
) -> InvoiceExtraction:
    """从发票图像提取信息"""
    with open(ctx.deps.image_path, 'rb') as f:
        base64_image = base64.b64encode(f.read()).decode('utf-8')
    
    response = ctx.deps.llm_client.chat.completions.create(
        model="gpt-4o",
        messages=[{
            "role": "user",
            "content": [
                {"type": "text", "text": "提取发票信息"},
                {"type": "image_url", "image_url": {"url": f"data:image/png;base64,{base64_image}"}}
            ]
        }],
        response_format=InvoiceExtraction.model_json_schema()
    )
    
    return InvoiceExtraction.model_validate_json(response.choices[0].message.content)

# 摘要 Agent
summary_agent = Agent(
    'openai:gpt-4o',
    output_type=str,
    instructions='将发票信息总结为几句话。'
)

async def process_invoice(image_path: str):
    deps = InvoiceDeps(
        llm_client=OpenAI(),
        image_path=image_path
    )
    
    # 第一步：提取
    extraction = await invoice_agent.run(
        '提取发票详情',
        deps=deps
    )
    
    # 第二步：摘要
    summary = await summary_agent.run(
        f'总结：{extraction.output}'
    )
    
    return extraction.output, summary.output

if __name__ == "__main__":
    extraction, summary = asyncio.run(process_invoice('invoice.png'))
    print(f"总金额：${extraction.total_amount}")
    print(f"发送方：{extraction.sender}")
    print(f"摘要：{summary}")
```

---

## 七、测试策略

### 7.1 使用 TestModel

```python
from pydantic_ai import Agent
from pydantic_ai.models.test import TestModel
from pydantic import BaseModel

class Output(BaseModel):
    value: int

agent = Agent('openai:gpt-4o', output_type=Output)

def test_agent():
    # 使用 TestModel 避免 API 调用
    with agent.override(model=TestModel(
        custom_result_args={'value': 42}
    )):
        result = agent.run_sync('测试')
        assert result.output.value == 42

test_agent()
```

### 7.2 模拟依赖

```python
from pydantic_ai import Agent
from dataclasses import dataclass

@dataclass
class Deps:
    db: object

agent = Agent('openai:gpt-4o', deps_type=Deps)

class MockDB:
    async def query(self, sql: str) -> list:
        return [{"id": 1, "name": "测试"}]

async def test_with_mock():
    deps = Deps(db=MockDB())
    
    with agent.override(model=TestModel()):
        result = await agent.run('查询', deps=deps)
        # 验证结果

asyncio.run(test_with_mock())
```

### 7.3 集成测试

```python
import pytest
from pydantic_ai import Agent

@pytest.fixture
def agent():
    return Agent('openai:gpt-4o')

def test_basic_query(agent):
    result = agent.run_sync('1+1 等于几？')
    assert '2' in result.output

@pytest.mark.asyncio
async def test_async_query(agent):
    result = await agent.run('测试')
    assert result.output is not None
```

---

## 八、生产部署

### 8.1 FastAPI 集成

```python
from fastapi import FastAPI, BackgroundTasks
from pydantic_ai import Agent
from pydantic import BaseModel

app = FastAPI()

class QueryRequest(BaseModel):
    question: str

class QueryResponse(BaseModel):
    answer: str
    sources: list[str]

agent = Agent(
    'openai:gpt-4o',
    output_type=QueryResponse
)

@app.post("/query")
async def query(request: QueryRequest):
    result = await agent.run(request.question)
    return result.output

@app.post("/query/background")
async def query_background(
    request: QueryRequest,
    background_tasks: BackgroundTasks
):
    job_id = generate_job_id()
    
    async def process():
        result = await agent.run(request.question)
        save_result(job_id, result)
    
    background_tasks.add_task(process)
    return {"job_id": job_id}

# 运行：uvicorn main:app --reload
```

### 8.2 Docker 部署

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV PYTHONUNBUFFERED=1

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  api:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - LOGFIRE_API_KEY=${LOGFIRE_API_KEY}
    volumes:
      - ./logs:/app/logs
```

### 8.3 监控和日志

```python
import logfire
from pydantic_ai import Agent

# 配置 Logfire
logfire.configure(
    api_key='your-key',
    send_to_logfire=True
)

# 创建带追踪的 Agent
agent = Agent(
    'openai:gpt-4o',
    instrument=True
)

# 自定义追踪
@logfire.instrument('处理查询：{question}')
async def process_query(question: str):
    result = await agent.run(question)
    logfire.info('查询完成', result=result.output)
    return result

# 在 Logfire dashboard 查看
# https://logfire.pydantic.dev
```

---

## 九、最佳实践

### 9.1 输出模型设计

```python
# ✅ 推荐：扁平模型
class SimpleOutput(BaseModel):
    field1: str
    field2: int

# ✅ 推荐：清晰的字段描述
class WellDocumented(BaseModel):
    user_id: int = Field(description="用户唯一标识符")
    email: str = Field(description="用户邮箱地址")

# ❌ 避免：过深的嵌套
class BadDesign(BaseModel):
    level1: Level1
    # Level1 包含 Level2，Level2 包含 Level3...

# ❌ 避免：模糊的字段名
class Vague(BaseModel):
    data: str  # 什么数据？
    value: int  # 什么值？
```

### 9.2 工具设计

```python
# ✅ 推荐：单一职责
@agent.tool
def get_weather(city: str) -> str:
    """获取城市天气"""
    ...

@agent.tool
def get_forecast(city: str, days: int) -> str:
    """获取天气预报"""
    ...

# ❌ 避免：多功能工具
@agent.tool
def handle_weather(city: str, action: str, days: int) -> str:
    """处理天气相关（获取、预报、警告...）"""
    ...

# ✅ 推荐：清晰的参数描述
@agent.tool
def search(
    query: Annotated[str, "搜索关键词"],
    limit: Annotated[int, "返回结果数量，默认 10"] = 10
) -> str:
    ...
```

### 9.3 依赖注入

```python
# ✅ 推荐：使用 dataclass
@dataclass
class Deps:
    db: Database
    api_key: str

# ✅ 推荐：类型注解
async def tool(ctx: RunContext[Deps], arg: str) -> str:
    ...

# ❌ 避免：全局状态
global_db = Database()  # 难以测试

@agent.tool
def tool(arg: str) -> str:
    global_db.query(...)  # 难以模拟
```

### 9.4 错误处理

```python
# ✅ 推荐：使用 ModelRetry
@agent.output_validator
async def validate(ctx: RunContext, output: Output):
    if output.value < 0:
        raise ModelRetry("值不能为负")
    return output

# ✅ 推荐：工具重试
@agent.tool(retries=2)
async def flaky_api(arg: str) -> str:
    ...

# ❌ 避免：静默失败
@agent.tool
def tool(arg: str) -> str:
    try:
        ...
    except:
        return "错误"  # 没有上下文
```

### 9.5 成本控制

```python
# ✅ 推荐：限制输出长度
class ConciseOutput(BaseModel):
    summary: str = Field(max_length=200)

# ✅ 推荐：使用较小模型
agent = Agent('openai:gpt-4o-mini')  # 便宜、快速

# ✅ 推荐：限制工具调用
agent = Agent(
    'openai:gpt-4o',
    model_settings={'max_tokens': 1000}
)

# ❌ 避免：无限制
agent = Agent('openai:gpt-4o')  # 可能产生高额费用
```

---

## 十、总结

PydanticAI 以其类型安全、结构化输出和优秀的开发者体验，正在成为生产级 AI Agent 开发的首选框架。

**核心优势：**
- ✅ **类型安全**：Pydantic 模型验证，错误早发现
- ✅ **结构化输出**：强制 LLM 返回符合 schema 的数据
- ✅ **依赖注入**：类型安全的运行时上下文
- ✅ **多模型支持**：15+ 提供商，切换无需改代码
- ✅ **工具注册**：简单的装饰器
- ✅ **流式响应**：实时输出带验证
- ✅ **可观测性**：Logfire 集成
- ✅ **易测试**：TestModel 和依赖模拟

**学习路径建议：**

1. **入门**：安装 → 第一个 Agent → 理解结构化输出
2. **进阶**：工具注册 → 依赖注入 → 流式响应
3. **高级**：MCP 集成 → 多 Agent → 持久化执行
4. **生产**：测试 → 监控 → 部署

**参考资源：**
- 官方文档：https://ai.pydantic.dev/
- GitHub：https://github.com/pydantic/pydantic-ai
- Pydantic Logfire：https://logfire.pydantic.dev/
- Discord 社区：https://discord.gg/pydantic

开始使用 PydanticAI 构建类型安全的 AI Agent 吧！🚀
