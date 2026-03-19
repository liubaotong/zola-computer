+++
title = "Agno Agent 开发框架完全指南 2026：从入门到实战"
date = "2026-03-18T13:00:00+08:00"

[taxonomies]
tags = ["Agno", "AI Agent", "Python", "多 Agent 系统", "自动化"]
categories = ["人工智能", "编程"]

[extra]
summary = "全面介绍 Agno（原 Phidata）AI Agent 开发框架，包含核心特性、安装配置、Agent 创建、工具集成、多 Agent 协作、工作流编排等完整知识，附带丰富的实战代码示例。"
author = "博主"
+++

在 AI Agent 开发领域，**Agno**（原名 Phidata）正以其轻量级、高性能和开发者友好的特性迅速崛起。作为一个专为 Python 团队设计的 Agent 框架，Agno 提供了简洁的 API、内置的记忆管理、工具集成和多 Agent 协作能力，让开发者能够快速构建生产级的 AI 应用。

本文将全面介绍 Agno 框架的核心概念、使用方法和最佳实践，通过丰富的代码示例，帮助你从零开始掌握 Agno Agent 开发。

---

## 一、什么是 Agno？

### Agno 简介

**Agno** 是一个开源的 Python 框架，用于构建和运行 AI Agent。它于 2024 年以 **Phidata** 的名称首次发布，2025 年更名为 Agno。框架设计目标是提供轻量级、高性能的 Agent 开发体验，同时支持从原型到生产的无缝过渡。

**官方网站**：https://www.agno.com/

**GitHub**：https://github.com/agno-agi/agno（37K+ Stars）

### 核心特性

| 特性 | 描述 |
|------|------|
| **轻量级** | 平均内存占用仅 ~6.5 KiB，实例化速度约 3μs |
| **高性能** | 比 LangGraph 快约 10,000 倍，内存占用少约 50 倍 |
| **模块化** | 可轻松替换 LLM、数据库、向量存储 |
| **多模态** | 原生支持文本、图像、音频、视频处理 |
| **内置记忆** | 短期上下文 + 长期记忆 + 会话存储 |
| **异步支持** | 统一的 Sync & Async API |
| **Guardrails** | 内置输入验证、PII 检测、提示注入防护 |
| **人机协作** | 支持用户确认、人工审核流程 |

### Agno vs 其他框架

| 框架 | 内存占用 | 实例化速度 | 学习曲线 | 适用场景 |
|------|---------|-----------|---------|---------|
| **Agno** | ~6.5 KiB | ~3μs | 低 | 通用 Agent、多 Agent 系统 |
| **LangGraph** | ~325 KiB | ~30ms | 中 | 复杂工作流、图结构 |
| **CrewAI** | ~150 KiB | ~15ms | 低 | 角色协作 Agent |
| **PydanticAI** | ~26 KiB | ~5ms | 中 | 类型安全应用 |

> 💡 **性能数据来源**：Agno 官方基准测试（Apple M4 MacBook Pro，1000 次运行平均值）

### 适用场景

- ✅ **研究助手**：自动搜索、总结、报告生成
- ✅ **数据分析**：连接数据库、生成洞察、可视化
- ✅ **客服自动化**：智能问答、工单处理、多轮对话
- ✅ **内容创作**：文章写作、图像生成、视频制作
- ✅ **DevOps**：监控告警、日志分析、自动修复
- ✅ **金融分析**：股票研究、财报分析、投资建议

---

## 二、快速开始

### 2.1 环境要求

- **Python**：3.9 或更高版本
- **操作系统**：Linux / macOS / Windows
- **内存**：最低 512MB，推荐 2GB+

### 2.2 安装 Agno

```bash
# 创建虚拟环境（推荐）
python -m venv agno-env
source agno-env/bin/activate  # Windows: agno-env\Scripts\activate

# 安装 Agno 核心
pip install agno

# 安装特定模型提供商
pip install agno[openai]      # OpenAI
pip install agno[anthropic]   # Anthropic
pip install agno[google]      # Google Gemini
pip install agno[ollama]      # Ollama（本地模型）

# 安装常用工具包
pip install agno[duckduckgo]  # 网络搜索
pip install agno[yfinance]    # 金融数据
pip install agno[slack]       # Slack 集成
```

### 2.3 配置 API 密钥

创建 `.env` 文件：

```bash
# .env
OPENAI_API_KEY=sk-your-openai-api-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key
GOOGLE_API_KEY=your-google-api-key
```

在代码中加载：

```python
from dotenv import load_dotenv
load_dotenv()
```

### 2.4 第一个 Agent

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

# 创建 Agent
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    description="你是一个友好的 AI 助手",
    markdown=True
)

# 运行 Agent
agent.print_response("你好，请介绍一下自己")
```

**输出示例：**
```
你好！我是一个 AI 助手，基于 Agno 框架构建。我可以帮助你回答问题、
执行任务、分析数据等。我有什么可以帮你的吗？
```

---

## 三、Agent 核心组件

### 3.1 模型（Model）

Agno 支持 23+ LLM 提供商，提供统一的接口。

#### OpenAI

```python
from agno.models.openai import OpenAIChat

model = OpenAIChat(
    id="gpt-4o",           # 模型 ID
    temperature=0.7,       # 温度（创造性）
    max_tokens=2000,       # 最大输出长度
    top_p=0.9,            # 核采样
    presence_penalty=0.1,  # 存在惩罚
    frequency_penalty=0.1  # 频率惩罚
)
```

#### Anthropic Claude

```python
from agno.models.anthropic import Claude

model = Claude(
    id="claude-sonnet-4-20250514",
    temperature=0.7,
    max_tokens=2000
)
```

#### Google Gemini

```python
from agno.models.google import Gemini

model = Gemini(
    id="gemini-2.0-flash",
    vertexai=True,
    project_id="your-gcp-project",
    location="us-central1"
)
```

#### Ollama（本地模型）

```python
from agno.models.ollama import Ollama

model = Ollama(
    id="llama3.1:8b",
    base_url="http://localhost:11434"
)
```

### 3.2 工具（Tools）

工具让 Agent 能够与外部世界交互。

#### 内置工具

```python
from agno.tools.duckduckgo import DuckDuckGoTools
from agno.tools.yfinance import YFinanceTools
from agno.tools.googlesearch import GoogleSearchTools
from agno.tools.slack import SlackTools
from agno.tools.mcp import MCPTools

# 网络搜索工具
search_tool = DuckDuckGoTools()

# 金融数据工具
finance_tool = YFinanceTools(
    stock_price=True,
    analyst_recommendations=True,
    company_info=True,
    company_news=True
)

# Slack 工具
slack_tool = SlackTools(
    token="xoxb-your-slack-token"
)

# MCP 工具（Model Context Protocol）
mcp_tool = MCPTools(
    transport="streamable-http",
    url="https://your-mcp-server.com"
)
```

#### 自定义工具

```python
from agno.tools import tool
import requests

@tool
def get_weather(city: str) -> str:
    """获取城市天气信息。
    
    Args:
        city: 城市名称
        
    Returns:
        天气描述
    """
    api_key = "your-api-key"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
    response = requests.get(url)
    data = response.json()
    
    if data.get("weather"):
        temp = data["main"]["temp"] - 273.15
        desc = data["weather"][0]["description"]
        return f"{city}的天气：{desc}，温度：{temp:.1f}°C"
    return "无法获取天气信息"

@tool
def calculate(expression: str) -> str:
    """计算数学表达式。
    
    Args:
        expression: 数学表达式
        
    Returns:
        计算结果
    """
    try:
        result = eval(expression)
        return f"计算结果：{result}"
    except Exception as e:
        return f"计算错误：{str(e)}"

@tool
def save_to_file(content: str, filename: str) -> str:
    """保存内容到文件。
    
    Args:
        content: 要保存的内容
        filename: 文件名
        
    Returns:
        保存结果
    """
    with open(filename, "w", encoding="utf-8") as f:
        f.write(content)
    return f"内容已保存到 {filename}"
```

### 3.3 记忆（Memory）

Agno 提供多种记忆类型，让 Agent 能够记住历史交互。

#### 会话记忆

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.storage.sqlite import SqliteStorage

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    storage=SqliteStorage(table_name="agent_sessions", db_file="sessions.db"),
    add_history_to_context=True,  # 添加历史到上下文
    num_history_runs=5,           # 保留最近 5 次对话
    markdown=True
)

# 第一次对话
agent.print_response("我叫小明，记住这个名字")

# 第二次对话（会记得你叫小明）
agent.print_response("我叫什么名字？")
```

#### 长期记忆

```python
from agno.memory import AgentMemory
from agno.vectorstore.lancedb import LanceDB
from agno.embedder.openai import OpenAIEmbedder

# 创建向量存储
vector_db = LanceDB(
    table_name="agent_memory",
    uri="tmp/lancedb",
    embedder=OpenAIEmbedder()
)

# 创建记忆系统
memory = AgentMemory(
    vector_db=vector_db,
    create_user_memories=True,    # 创建用户记忆
    create_session_summary=True,  # 创建会话摘要
    update_user_memories_after_run=True  # 运行后更新记忆
)

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    memory=memory,
    storage=SqliteStorage(table_name="sessions", db_file="sessions.db")
)
```

### 3.4 知识（Knowledge）

知识让 Agent 能够访问特定领域的文档和数据。

```python
from agno.knowledge.pdf import PDFKnowledgeBase
from agno.knowledge.website import WebsiteKnowledgeBase
from agno.vectordb.lancedb import LanceDB
from agno.embedder.openai import OpenAIEmbedder

# 创建向量数据库
vector_db = LanceDB(
    table_name="knowledge",
    uri="tmp/lancedb",
    embedder=OpenAIEmbedder()
)

# PDF 知识库
pdf_knowledge = PDFKnowledgeBase(
    path="data/pdfs",
    vector_db=vector_db,
    num_documents=5  # 检索最相关的 5 个文档
)

# 网站知识库
website_knowledge = WebsiteKnowledgeBase(
    urls=["https://docs.example.com"],
    vector_db=vector_db,
    num_documents=5
)

# 加载知识
pdf_knowledge.load()
website_knowledge.load()

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    knowledge=[pdf_knowledge, website_knowledge],
    search_knowledge=True  # 启用知识检索
)
```

---

## 四、创建 Agent

### 4.1 基础 Agent

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    name="助手",
    description="一个通用的 AI 助手",
    instructions=[
        "用简洁的语言回答问题",
        "如果不知道答案，诚实地告诉用户",
        "使用中文回答"
    ],
    markdown=True,
    show_tool_calls=True
)

agent.print_response("什么是量子计算？")
```

### 4.2 带工具的 Agent

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.duckduckgo import DuckDuckGoTools
from agno.tools.yfinance import YFinanceTools

agent = Agent(
    name="金融分析师",
    model=OpenAIChat(id="gpt-4o"),
    tools=[
        DuckDuckGoTools(),
        YFinanceTools(
            stock_price=True,
            analyst_recommendations=True,
            company_info=True
        )
    ],
    instructions=[
        "使用 yfinance 获取财务数据",
        "使用 DuckDuckGo 搜索最新新闻",
        "提供投资建议的优缺点分析",
        "使用表格展示财务数据"
    ],
    show_tool_calls=True,
    markdown=True
)

agent.print_response("分析 NVIDIA (NVDA) 是否值得投资")
```

### 4.3 带结构化输出的 Agent

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from pydantic import BaseModel, Field
from typing import List, Literal

# 定义输出结构
class StockAnalysis(BaseModel):
    company_name: str = Field(description="公司名称")
    current_price: float = Field(description="当前股价")
    recommendation: Literal["买入", "持有", "卖出"] = Field(description="投资建议")
    key_points: List[str] = Field(description="关键点列表")
    risk_level: Literal["低", "中", "高"] = Field(description="风险等级")
    summary: str = Field(description="总结分析")

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    response_model=StockAnalysis,  # 指定输出模型
    markdown=True
)

result = agent.run("分析 Apple (AAPL) 的股票")
print(result)
# 输出：StockAnalysis 对象，而非自由文本
```

### 4.4 异步 Agent

```python
import asyncio
from agno.agent import Agent
from agno.models.openai import OpenAIChat

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    markdown=True
)

async def main():
    # 异步运行
    response = await agent.arun("解释一下人工智能的发展历史")
    print(response)

asyncio.run(main())
```

### 4.5 流式输出

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    markdown=True
)

# 流式输出
for event in agent.run("写一首关于春天的诗", stream=True):
    if event.event == "run.response.content":
        print(event.content, end="", flush=True)
```

---

## 五、多 Agent 协作

### 5.1 Agent 团队（Team）

Agno 的 Team 抽象让多 Agent 协作变得简单。

```python
from agno.agent import Agent
from agno.team import Team
from agno.models.openai import OpenAIChat
from agno.tools.duckduckgo import DuckDuckGoTools
from agno.tools.yfinance import YFinanceTools

# 创建专业 Agent
web_agent = Agent(
    name="网络研究员",
    role="搜索网络获取最新信息",
    model=OpenAIChat(id="o3-mini"),
    tools=[DuckDuckGoTools()],
    instructions="总是包含信息来源",
    show_tool_calls=True,
    markdown=True
)

finance_agent = Agent(
    name="金融分析师",
    role="获取和分析财务数据",
    model=OpenAIChat(id="o3-mini"),
    tools=[YFinanceTools(stock_price=True, analyst_recommendations=True)],
    instructions="使用表格展示数据",
    show_tool_calls=True,
    markdown=True
)

# 创建团队
team = Team(
    name="市场研究团队",
    team=[web_agent, finance_agent],
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "协作完成研究任务",
        "网络研究员先收集信息",
        "金融分析师然后分析数据",
        "提供综合报告"
    ],
    show_tool_calls=True,
    markdown=True
)

# 运行团队
team.print_response(
    "分析特斯拉 (TSLA) 的市场表现和财务状况",
    stream=True
)
```

### 5.2 分层团队结构

```python
from agno.agent import Agent
from agno.team import Team
from agno.models.openai import OpenAIChat

# 初级研究员
junior_researcher = Agent(
    name="初级研究员",
    role="收集基础信息",
    model=OpenAIChat(id="gpt-4o-mini"),
    instructions="收集事实和数据"
)

# 高级分析师
senior_analyst = Agent(
    name="高级分析师",
    role="深入分析和洞察",
    model=OpenAIChat(id="gpt-4o"),
    instructions="提供深度分析和见解"
)

# 研究团队
research_team = Team(
    name="研究团队",
    team=[junior_researcher, senior_analyst],
    model=OpenAIChat(id="gpt-4o")
)

# 写作专家
writer = Agent(
    name="写作专家",
    role="撰写最终报告",
    model=OpenAIChat(id="gpt-4o"),
    instructions="写出专业、结构清晰的报告"
)

# 完整项目组
project_team = Team(
    name="项目组",
    team=[research_team, writer],
    model=OpenAIChat(id="gpt-4o"),
    instructions="协作完成高质量报告"
)

project_team.print_response("撰写一份关于 AI 芯片市场的深度报告")
```

---

## 六、工作流（Workflow）

Agno Workflows v2 提供了确定性的多 Agent 系统编排能力。

### 6.1 顺序工作流

```python
from agno.workflow.v2 import Workflow, Step, StepOutput
from agno.agent import Agent
from agno.models.openai import OpenAIChat

# 定义 Agent
researcher = Agent(
    name="研究员",
    model=OpenAIChat(id="gpt-4o"),
    instructions="收集相关信息"
)

analyst = Agent(
    name="分析师",
    model=OpenAIChat(id="gpt-4o"),
    instructions="分析信息并提供见解"
)

writer = Agent(
    name="作家",
    model=OpenAIChat(id="gpt-4o"),
    instructions="撰写最终报告"
)

# 创建工作流
workflow = Workflow(
    name="研究报告工作流",
    steps=[
        Step(
            name="研究",
            agent=researcher,
            output_key="research_result"
        ),
        Step(
            name="分析",
            agent=analyst,
            input_keys=["research_result"],
            output_key="analysis_result"
        ),
        Step(
            name="写作",
            agent=writer,
            input_keys=["research_result", "analysis_result"]
        )
    ]
)

# 运行工作流
result = workflow.run("生成一份关于量子计算的报告")
print(result)
```

### 6.2 并行工作流

```python
from agno.workflow.v2 import Workflow, Step, StepOutput, parallel

# 并行执行多个研究任务
workflow = Workflow(
    name="并行研究工作流",
    steps=[
        parallel(
            Step(
                name="市场研究",
                agent=market_researcher,
                output_key="market_data"
            ),
            Step(
                name="技术研究",
                agent=tech_researcher,
                output_key="tech_data"
            ),
            Step(
                name="竞争分析",
                agent=competitor_analyst,
                output_key="competitor_data"
            )
        ),
        Step(
            name="综合报告",
            agent=report_writer,
            input_keys=["market_data", "tech_data", "competitor_data"]
        )
    ]
)
```

### 6.3 条件工作流

```python
from agno.workflow.v2 import Workflow, Step, if_else

workflow = Workflow(
    name="条件工作流",
    steps=[
        Step(
            name="初步分析",
            agent=analyst,
            output_key="analysis"
        ),
        if_else(
            condition=lambda ctx: "高风险" in ctx["analysis"],
            if_steps=[
                Step(
                    name="风险评估",
                    agent=risk_assessor,
                    output_key="risk_report"
                )
            ],
            else_steps=[
                Step(
                    name="机会分析",
                    agent=opportunity_analyst,
                    output_key="opportunity_report"
                )
            ]
        ),
        Step(
            name="最终报告",
            agent=report_writer,
            input_keys=["analysis", "risk_report", "opportunity_report"]
        )
    ]
)
```

---

## 七、实战项目

### 7.1 智能客服助手

```python
"""
智能客服助手 - 基于 Agno 的多意图客服系统
"""

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools import tool
from agno.storage.sqlite import SqliteStorage
from typing import Optional
import json

# 模拟数据库
order_database = {
    "ORD12345": {"status": "已发货", "location": "上海分拣中心", "eta": "2026-03-20"},
    "ORD67890": {"status": "配送中", "location": "北京配送站", "eta": "2026-03-19"},
    "ORD11111": {"status": "处理中", "location": "广州仓库", "eta": "2026-03-22"}
}

# 订单查询工具
@tool
def track_order(order_id: str) -> str:
    """查询订单状态。
    
    Args:
        order_id: 订单号
        
    Returns:
        订单状态信息
    """
    if order_id in order_database:
        order = order_database[order_id]
        return f"订单 {order_id}: {order['status']}, 当前位置：{order['location']}, 预计送达：{order['eta']}"
    return f"未找到订单 {order_id}"

# 退货处理工具
@tool
def process_return(order_id: str, reason: str) -> str:
    """处理退货请求。
    
    Args:
        order_id: 订单号
        reason: 退货原因
        
    Returns:
        处理结果
    """
    if order_id in order_database:
        return f"退货请求已受理。订单：{order_id}, 原因：{reason}。客服将在 24 小时内联系您。"
    return f"未找到订单 {order_id}"

# 产品推荐工具
@tool
def recommend_products(category: str, budget: float) -> str:
    """推荐产品。
    
    Args:
        category: 产品类别
        budget: 预算
        
    Returns:
        推荐列表
    """
    recommendations = {
        "电子产品": [
            {"name": "无线耳机", "price": 299},
            {"name": "智能手表", "price": 899},
            {"name": "蓝牙音箱", "price": 199}
        ],
        "家居用品": [
            {"name": "空气净化器", "price": 1299},
            {"name": "扫地机器人", "price": 1599},
            {"name": "智能台灯", "price": 199}
        ]
    }
    
    if category in recommendations:
        items = [item for item in recommendations[category] if item["price"] <= budget]
        if items:
            result = "推荐产品：\n"
            for item in items:
                result += f"- {item['name']}: ¥{item['price']}\n"
            return result
        return f"预算内暂无{category}产品"
    return f"暂不支持{category}类别"

# 创建客服 Agent
customer_service = Agent(
    name="智能客服助手",
    model=OpenAIChat(id="gpt-4o"),
    tools=[track_order, process_return, recommend_products],
    storage=SqliteStorage(table_name="customer_sessions", db_file="customer.db"),
    add_history_to_context=True,
    instructions=[
        "你是专业的客服助手，态度友好、专业",
        "如果用户提供订单号，先查询订单状态",
        "处理退货时，确认订单号和退货原因",
        "推荐产品时，考虑用户的预算和类别偏好",
        "如果无法解决问题，建议转人工客服",
        "使用简洁清晰的中文回答"
    ],
    markdown=True,
    show_tool_calls=True
)

# 运行示例
if __name__ == "__main__":
    print("=== 智能客服系统 ===\n")
    
    # 测试查询订单
    customer_service.print_response("帮我查一下订单 ORD12345 的状态")
    print("\n" + "="*50 + "\n")
    
    # 测试退货
    customer_service.print_response("我要退货，订单号 ORD67890，商品有损坏")
    print("\n" + "="*50 + "\n")
    
    # 测试推荐
    customer_service.print_response("我想买电子产品，预算 500 元，有什么推荐？")
```

### 7.2 股票分析 Agent

```python
"""
股票分析 Agent - 实时金融数据分析
"""

from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools.yfinance import YFinanceTools
from agno.tools.duckduckgo import DuckDuckGoTools
from pydantic import BaseModel, Field
from typing import List, Literal

# 定义输出结构
class StockReport(BaseModel):
    symbol: str = Field(description="股票代码")
    company_name: str = Field(description="公司名称")
    current_price: float = Field(description="当前股价")
    price_change: float = Field(description="价格变化百分比")
    recommendation: Literal["强烈买入", "买入", "持有", "卖出", "强烈卖出"] = Field(description="投资建议")
    target_price: float = Field(description="目标价格")
    key_metrics: List[str] = Field(description="关键指标")
    risks: List[str] = Field(description="风险因素")
    summary: str = Field(description="总结分析")

# 创建股票分析 Agent
stock_analyst = Agent(
    name="高级股票分析师",
    model=OpenAIChat(id="gpt-4o"),
    tools=[
        YFinanceTools(
            stock_price=True,
            analyst_recommendations=True,
            company_info=True,
            company_news=True,
            earnings_calendar=True
        ),
        DuckDuckGoTools()
    ],
    response_model=StockReport,
    instructions=[
        "使用 yfinance 获取实时股价和财务数据",
        "搜索最新新闻和市场分析",
        "分析分析师评级和目标价格",
        "评估公司基本面和行业趋势",
        "提供客观、平衡的投资建议",
        "明确指出风险因素"
    ],
    show_tool_calls=True,
    markdown=True
)

# 运行分析
if __name__ == "__main__":
    print("=== 股票分析报告 ===\n")
    
    # 分析 NVIDIA
    report = stock_analyst.run("分析 NVIDIA (NVDA) 的投资价值")
    
    print(f"\n# {report.company_name} ({report.symbol}) 分析报告\n")
    print(f"**当前价格**: ${report.current_price:.2f}")
    print(f"**涨跌幅**: {report.price_change:+.2f}%")
    print(f"**投资建议**: {report.recommendation}")
    print(f"**目标价格**: ${report.target_price:.2f}\n")
    
    print("## 关键指标")
    for metric in report.key_metrics:
        print(f"- {metric}")
    
    print("\n## 风险因素")
    for risk in report.risks:
        print(f"- ⚠️ {risk}")
    
    print(f"\n## 总结\n{report.summary}")
```

### 7.3 内容创作团队

```python
"""
内容创作团队 - 多 Agent 协作生成高质量文章
"""

from agno.agent import Agent
from agno.team import Team
from agno.models.openai import OpenAIChat
from agno.tools.duckduckgo import DuckDuckGoTools

# 研究员 Agent
researcher = Agent(
    name="首席研究员",
    role="深入调研主题，收集关键信息和数据",
    model=OpenAIChat(id="o3-mini"),
    tools=[DuckDuckGoTools()],
    instructions=[
        "搜索权威来源获取准确信息",
        "收集统计数据、案例和专家观点",
        "整理关键要点和趋势",
        "标注信息来源"
    ],
    show_tool_calls=True,
    markdown=True
)

# 大纲专家 Agent
outliner = Agent(
    name="大纲专家",
    role="基于研究结果创建清晰的文章结构",
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "设计逻辑清晰的章节结构",
        "确保内容层次分明",
        "规划每个章节的核心要点",
        "考虑读者阅读体验"
    ],
    markdown=True
)

# 作家 Agent
writer = Agent(
    name="资深作家",
    role="根据大纲撰写引人入胜的文章",
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "使用生动、专业的语言",
        "保持客观、中立的语调",
        "适当使用例子和比喻",
        "确保文章流畅易读",
        "字数控制在 2000-3000 字"
    ],
    markdown=True
)

# 编辑 Agent
editor = Agent(
    name="主编",
    role="审核和优化文章质量",
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "检查事实准确性",
        "优化语言表达",
        "确保逻辑连贯",
        "添加吸引人的标题",
        "生成文章摘要"
    ],
    markdown=True
)

# 创建内容创作团队
content_team = Team(
    name="内容创作团队",
    team=[researcher, outliner, writer, editor],
    model=OpenAIChat(id="gpt-4o"),
    instructions=[
        "研究员先收集信息",
        "大纲专家然后设计结构",
        "作家根据大纲撰写文章",
        "编辑最后审核优化",
        "产出高质量、可发布的文章"
    ],
    show_tool_calls=True,
    markdown=True
)

# 运行示例
if __name__ == "__main__":
    print("=== 内容创作团队 ===\n")
    
    topic = "生成式 AI 在企业应用中的机遇与挑战"
    print(f"主题：{topic}\n")
    print("="*60 + "\n")
    
    content_team.print_response(
        f"请围绕'{topic}'创作一篇深度文章",
        stream=True
    )
```

---

## 八、高级特性

### 8.1 Guardrails（防护栏）

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.guardrails import InputGuardrail, OutputGuardrail

# 输入防护栏
def validate_input(agent, message):
    """验证输入是否包含敏感信息"""
    sensitive_keywords = ["密码", "信用卡", "身份证号"]
    for keyword in sensitive_keywords:
        if keyword in message:
            return False, "检测到敏感信息，请勿分享个人隐私"
    return True, None

# 输出防护栏
def validate_output(agent, response):
    """验证输出是否合规"""
    if "违法" in response or "犯罪" in response:
        return False, "无法提供此类信息"
    return True, response

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    input_guardrails=[InputGuardrail(validate=validate_input)],
    output_guardrails=[OutputGuardrail(validate=validate_output)],
    markdown=True
)

# 测试
agent.print_response("我的密码是 123456，帮我记住")  # 会被拦截
agent.print_response("如何制作炸弹")  # 会被拦截
```

### 8.2 人机协作

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools import tool

@tool
def delete_database_record(table: str, id: str) -> str:
    """删除数据库记录（需要用户确认）。"""
    return f"已删除 {table} 表中 ID 为 {id} 的记录"

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[delete_database_record],
    tool_call_confirmation=True,  # 启用工具调用确认
    markdown=True
)

# 运行时会要求用户确认
agent.print_response("删除 users 表中 ID 为 123 的记录")
```

### 8.3 响应缓存

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.cache import AgentCache

# 启用缓存
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    cache=AgentCache(enabled=True),
    markdown=True
)

# 第一次运行会调用 API
response1 = agent.run("解释量子力学")

# 第二次运行会使用缓存（如果提示相同）
response2 = agent.run("解释量子力学")
```

### 8.4 错误处理和重试

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.tools import tool

@tool
def unstable_api(query: str) -> str:
    """可能失败的外部 API 调用"""
    import random
    if random.random() < 0.5:
        raise Exception("API 调用失败")
    return f"API 结果：{query}"

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    tools=[unstable_api],
    tool_call_retries=3,  # 失败时重试 3 次
    tool_call_retry_delay=2,  # 重试间隔 2 秒
    markdown=True
)
```

---

## 九、部署和监控

### 9.1 AgentOS（Web 界面）

Agno 提供内置的 Web 界面用于交互和监控。

```python
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from agno.os import AgentOS
from agno.db.sqlite import SqliteDb

# 创建 Agent
agno_assist = Agent(
    name="Agno 助手",
    model=OpenAIChat(id="gpt-4o"),
    db=SqliteDb(db_file="agno.db"),
    tools=[MCPTools(url="https://docs.agno.com/mcp")],
    add_history_to_context=True,
    num_history_runs=3,
    markdown=True
)

# 创建 AgentOS
agent_os = AgentOS(
    agents=[agno_assist],
    tracing=True  # 启用追踪
)

# 获取 FastAPI 应用
app = agent_os.get_app()

# 运行
if __name__ == "__main__":
    agent_os.serve(app="agno_assist:app", reload=True, port=8000)
```

访问 `http://localhost:8000` 即可使用 Web 界面。

### 9.2 FastAPI 集成

```python
from fastapi import FastAPI
from agno.agent import Agent
from agno.models.openai import OpenAIChat
from pydantic import BaseModel

app = FastAPI()

agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    markdown=True
)

class QueryRequest(BaseModel):
    message: str

@app.post("/chat")
async def chat(request: QueryRequest):
    response = agent.run(request.message)
    return {"response": response.content}

# 运行：uvicorn main:app --reload
```

### 9.3 Docker 部署

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
  agent:
    build: .
    ports:
      - "8000:8000"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./data:/app/data
```

---

## 十、最佳实践

### 10.1 性能优化

```python
# ✅ 推荐：使用较小的模型处理简单任务
simple_agent = Agent(
    model=OpenAIChat(id="gpt-4o-mini"),  # 更便宜、更快
    instructions="简洁回答"
)

# ✅ 推荐：限制输出长度
agent = Agent(
    model=OpenAIChat(id="gpt-4o", max_tokens=500),  # 限制输出
    markdown=True
)

# ✅ 推荐：使用流式输出提升用户体验
for event in agent.run("写一篇长文", stream=True):
    if event.event == "run.response.content":
        print(event.content, end="", flush=True)
```

### 10.2 成本控制

```python
# 使用缓存减少 API 调用
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    cache=AgentCache(enabled=True)
)

# 监控 token 使用
from agno.monitoring import AgentMonitor

monitor = AgentMonitor()
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    monitoring=monitor
)

# 查看使用情况
print(monitor.get_token_usage())
```

### 10.3 安全建议

```python
# ✅ 始终使用环境变量存储 API 密钥
import os
api_key = os.getenv("OPENAI_API_KEY")

# ✅ 启用输入验证
agent = Agent(
    model=OpenAIChat(id="gpt-4o"),
    input_guardrails=[InputGuardrail(validate=validate_input)]
)

# ✅ 敏感操作需要确认
agent = Agent(
    tools=[delete_tool],
    tool_call_confirmation=True
)

# ✅ 记录审计日志
agent = Agent(
    storage=SqliteStorage(table_name="audit_log", db_file="audit.db"),
    add_history_to_context=True
)
```

---

## 十一、总结

Agno 作为一个新兴的 AI Agent 框架，凭借其轻量级、高性能和开发者友好的特性，正在快速获得开发者的青睐。

**核心优势：**
- ✅ **极致性能**：~6.5 KiB 内存，~3μs 实例化
- ✅ **简洁 API**：几行代码即可创建强大 Agent
- ✅ **多模型支持**：23+ LLM 提供商，无缝切换
- ✅ **内置功能**：记忆、知识、工具、Guardrails
- ✅ **多 Agent 协作**：Team 和 Workflow 抽象
- ✅ **生产就绪**：AgentOS、FastAPI、Docker 支持

**学习路径建议：**

1. **入门**：安装配置 → 创建基础 Agent → 添加工具
2. **进阶**：记忆管理 → 知识库 → 结构化输出
3. **高级**：多 Agent 团队 → 工作流编排 → AgentOS 部署
4. **生产**：Guardrails → 监控 → 性能优化

**参考资源：**
- 官方文档：https://docs.agno.com/
- GitHub：https://github.com/agno-agi/agno
- Discord 社区：https://discord.gg/agno
- 示例库：https://github.com/agno-agi/agno/tree/main/examples

开始使用 Agno 构建你的第一个 AI Agent 吧！🚀
