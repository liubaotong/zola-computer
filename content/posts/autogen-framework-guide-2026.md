+++
title = "AutoGen 开发框架完全指南 2026：微软多 Agent 协作系统实战"
date = "2026-03-18T16:00:00+08:00"

[taxonomies]
tags = ["AutoGen", "Microsoft", "AI Agent", "多 Agent 系统", "Python"]
categories = ["人工智能", "编程"]

[extra]
summary = "全面介绍微软 AutoGen 多 Agent 协作框架，包含核心概念、架构设计、Agent 创建、对话模式、工具集成、代码执行等完整知识，附带丰富的实战代码示例和项目案例。"
author = "博主"
+++

在多 Agent AI 系统领域，**AutoGen** 以其灵活的对话式协作架构和强大的代码执行能力独树一帜。作为微软研究院开源的框架，AutoGen 让多个 AI Agent 能够像人类团队一样通过对话协作完成复杂任务，支持完全自主或人机协作的工作流程。

本文将深入介绍 AutoGen 的核心概念、使用方法和最佳实践，通过完整的代码示例，帮助你掌握构建生产级多 Agent 系统的关键技术。

---

## 一、什么是 AutoGen？

### AutoGen 简介

**AutoGen** 是微软研究院于 2023 年开源的编程框架，用于构建 AI Agent 并促进多个 Agent 之间的协作。它旨在提供一个易用且灵活的框架，加速 Agent AI 的开发和研究。

**核心理念**：通过对话式协作，让多个 Agent 共同解决复杂任务，每个 Agent 可以扮演不同角色、使用不同工具、拥有不同能力。

**官方网站**：https://microsoft.github.io/autogen/

**GitHub**：https://github.com/microsoft/autogen（35K+ Stars）

**版本说明**：当前稳定版本为 v0.4，v0.2 版本仍有大量用户。2026 年微软推出了新的 **Microsoft Agent Framework**，AutoGen 用户可迁移至新框架。

### 核心特性

| 特性 | 描述 |
|------|------|
| **对话式 Agent** | Agent 之间可以相互发送和接收消息 |
| **可定制化** | 支持自定义 Agent 角色、能力和行为 |
| **多对话模式** | 支持单聊、群聊、层级聊天等多种模式 |
| **代码执行** | 支持自动生成和执行代码 |
| **人机协作** | 支持人在回路（Human-in-the-Loop） |
| **工具集成** | 支持调用外部 API、函数和工具 |
| **异步架构** | 基于事件驱动的异步设计 |
| **可扩展性** | 支持分布式运行时和跨语言 |

### AutoGen vs 其他框架

| 框架 | 架构风格 | 适用场景 | 学习曲线 |
|------|---------|---------|---------|
| **AutoGen** | 对话驱动 | 开放式任务、代码密集型 | 中 |
| **CrewAI** | 角色协作 | 流程化任务、业务工作流 | 低 |
| **LangGraph** | 图结构 | 复杂工作流、状态管理 | 中高 |
| **Agno** | 轻量级 | 快速原型、简单任务 | 低 |

### 适用场景

- ✅ **代码生成与调试**：自动编写、执行和修复代码
- ✅ **数据分析**：多 Agent 协作进行数据探索和分析
- ✅ **研究任务**：信息收集、分析、报告生成
- ✅ **内容创作**：博客文章、技术文档自动生成
- ✅ **客户支持**：多 Agent 协作处理复杂咨询
- ✅ **教育辅导**：个性化学习和问题解决

---

## 二、快速开始

### 2.1 环境要求

- **Python**: 3.10 - 3.13
- **操作系统**: Linux / macOS / Windows
- **内存**: 最低 2GB，推荐 4GB+

### 2.2 安装 AutoGen

```bash
# 创建虚拟环境
python -m venv autogen-env
source autogen-env/bin/activate  # Windows: autogen-env\Scripts\activate

# 安装 AutoGen AgentChat（推荐，简化 API）
pip install autogen-agentchat

# 安装 OpenAI 扩展
pip install "autogen-ext[openai]"

# 安装完整功能（包括代码执行）
pip install "autogen-agentchat[web-surfer]"

# 安装其他工具
pip install rich python-dotenv httpx
```

### 2.3 配置 API 密钥

创建 `.env` 文件：

```bash
# .env
OPENAI_API_KEY=sk-your-openai-api-key
AZURE_OPENAI_API_KEY=your-azure-key
AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com/
```

在代码中加载：

```python
from dotenv import load_dotenv
load_dotenv()
```

### 2.4 第一个 Agent

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

# 创建模型客户端
model_client = OpenAIChatCompletionClient(
    model="gpt-4o-mini",
    api_key="your-api-key"
)

# 创建 Assistant Agent
agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    system_message="你是一个友好的 AI 助手。"
)

# 运行 Agent
async def main():
    result = await agent.run(task="讲一个关于编程的笑话")
    print(result.messages[-1].content)

asyncio.run(main())
```

---

## 三、核心组件

### 3.1 Agent 类型

AutoGen 提供多种预构建的 Agent 类型。

#### AssistantAgent

```python
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

assistant = AssistantAgent(
    name="助手",
    model_client=model_client,
    system_message="""你是一个专业的研究助手。
    你的职责：
    1. 收集和分析信息
    2. 提供详细的报告
    3. 在任务完成后回复 "TERMINATE"
    """,
    description="研究和分析专家"
)
```

#### UserProxyAgent

```python
from autogen_agentchat.agents import UserProxyAgent

user_proxy = UserProxyAgent(
    name="用户代理",
    system_message="一个代表用户的代理。",
    human_input_mode="NEVER",  # 从不询问人工输入
    # 其他选项："ALWAYS"（总是询问）、"TERMINATE"（仅在终止时询问）
    code_execution_config={
        "executor": "local"  # 本地执行代码
    }
)
```

#### CodeExecutorAgent

```python
from autogen_agentchat.agents import CodeExecutorAgent
from autogen_ext.code_executors import LocalCommandLineCodeExecutor

# 创建代码执行器
code_executor = LocalCommandLineCodeExecutor(work_dir="./code_runs")

# 创建代码执行 Agent
code_agent = CodeExecutorAgent(
    name="代码执行器",
    code_executor=code_executor,
    description="执行 Python 代码"
)
```

#### MultimodalWebSurfer（网页浏览）

```python
from autogen_ext.agents.web_surfer import MultimodalWebSurfer

web_surfer = MultimodalWebSurfer(
    name="网页浏览者",
    model_client=model_client,
    description="浏览网页并提取信息"
)
```

### 3.2 模型客户端

AutoGen 支持多种 LLM 提供商。

#### OpenAI

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

client = OpenAIChatCompletionClient(
    model="gpt-4o",
    api_key="sk-...",
    temperature=0.7,
    max_tokens=2000
)
```

#### Azure OpenAI

```python
from autogen_ext.models.openai import AzureOpenAIChatCompletionClient

client = AzureOpenAIChatCompletionClient(
    model="gpt-4",
    azure_endpoint="https://your-resource.openai.azure.com/",
    api_key="your-key",
    api_version="2024-02-15-preview"
)
```

#### Ollama（本地模型）

```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

client = OpenAIChatCompletionClient(
    model="llama3.1:8b",
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # Ollama 不需要真实 key
)
```

#### 自定义模型

```python
from autogen_core.models import ChatCompletionClient

class CustomModelClient(ChatCompletionClient):
    async def create(self, messages, **kwargs):
        # 实现自定义推理逻辑
        pass
```

### 3.3 工具（Tools）

工具让 Agent 能够执行具体操作。

#### 定义工具

```python
from autogen_agentchat.agents import AssistantAgent
from typing import Annotated
import requests

# 定义工具函数
def get_weather(city: Annotated[str, "城市名称"]) -> str:
    """获取城市天气信息。"""
    api_key = "your-api-key"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
    response = requests.get(url)
    data = response.json()
    
    if data.get("weather"):
        temp = data["main"]["temp"] - 273.15
        desc = data["weather"][0]["description"]
        return f"{city}的天气：{desc}，温度：{temp:.1f}°C"
    return "无法获取天气信息"

def calculate(expression: Annotated[str, "数学表达式"]) -> str:
    """计算数学表达式。"""
    try:
        result = eval(expression)
        return f"计算结果：{result}"
    except Exception as e:
        return f"计算错误：{str(e)}"

# 创建带工具的 Agent
agent = AssistantAgent(
    name="助手",
    model_client=model_client,
    tools=[get_weather, calculate]
)
```

#### 使用 Pydantic 结构化输出

```python
from pydantic import BaseModel, Field
from typing import List, Literal

class WeatherReport(BaseModel):
    city: str = Field(description="城市名称")
    temperature: float = Field(description="温度（摄氏度）")
    condition: str = Field(description="天气状况")
    recommendation: str = Field(description="出行建议")

class StockAnalysis(BaseModel):
    symbol: str = Field(description="股票代码")
    current_price: float = Field(description="当前股价")
    recommendation: Literal["买入", "持有", "卖出"] = Field(description="投资建议")
    key_points: List[str] = Field(description="关键点列表")

# 使用结构化输出
agent = AssistantAgent(
    name="分析师",
    model_client=model_client,
    model_client_stream=True,
    reflect_on_tool_use=True
)

# 运行并获取结构化输出
result = await agent.run(
    task="分析 Apple (AAPL) 的股票",
    output_content_type=StockAnalysis
)
```

---

## 四、对话模式

AutoGen 支持多种对话模式，适应不同场景需求。

### 4.1 单聊（Two-Agent Chat）

最简单的对话模式，两个 Agent 之间来回对话。

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

# 创建助手 Agent
assistant = AssistantAgent(
    name="助手",
    model_client=model_client,
    system_message="你是一个专业的助手。完成任务后回复 'TERMINATE'。"
)

# 创建用户代理
user_proxy = UserProxyAgent(
    name="用户",
    system_message="代表用户的代理。",
    human_input_mode="NEVER"
)

# 设置终止条件
termination = TextMentionTermination("TERMINATE")

async def main():
    # 开始对话
    await Console(
        assistant.run_stream(
            task="今天北京的天气如何？",
            termination_condition=termination
        )
    )

asyncio.run(main())
```

### 4.2 群聊（Group Chat）

多个 Agent 参与讨论，由 GroupChatManager 协调。

```python
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent

# 创建多个 Agent
researcher = AssistantAgent(
    name="研究员",
    model_client=model_client,
    system_message="你负责收集和分析信息。"
)

writer = AssistantAgent(
    name="作家",
    model_client=model_client,
    system_message="你负责撰写内容。"
)

reviewer = AssistantAgent(
    name="审核员",
    model_client=model_client,
    system_message="你负责审核内容质量。"
)

user_proxy = UserProxyAgent(
    name="用户",
    human_input_mode="NEVER"
)

# 创建群聊团队（轮询模式）
team = RoundRobinGroupChat(
    agents=[researcher, writer, reviewer, user_proxy],
    termination_condition=TextMentionTermination("TERMINATE")
)

async def main():
    await Console(
        team.run_stream(task="撰写一篇关于 AI 发展的文章")
    )

asyncio.run(main())
```

### 4.3 层级聊天（Hierarchical Chat）

Manager Agent 分配任务给 Worker Agents。

```python
from autogen_agentchat.teams import SelectorGroupChat

# 创建 Manager
manager = AssistantAgent(
    name="经理",
    model_client=model_client,
    system_message="你负责分配任务给团队成员。"
)

# 创建 Worker
coder = AssistantAgent(
    name="程序员",
    model_client=model_client,
    system_message="你负责编写代码。"
)

tester = AssistantAgent(
    name="测试员",
    model_client=model_client,
    system_message="你负责测试代码。"
)

# 创建选择器群聊（Manager 决定谁发言）
team = SelectorGroupChat(
    agents=[manager, coder, tester],
    model_client=model_client,
    termination_condition=TextMentionTermination("TERMINATE")
)

async def main():
    await Console(
        team.run_stream(task="创建一个计算器应用")
    )

asyncio.run(main())
```

### 4.4 顺序聊天（Sequential Chats）

多个对话按顺序执行，前一个对话的结果传递给下一个。

```python
from autogen_agentchat import initiate_chats

# 定义聊天配置
chat_configs = [
    {
        "sender": user_proxy,
        "recipient": financial_assistant,
        "message": "查询 NVDA 和 TSLA 的当前股价",
        "summary_method": "last_msg"  # 使用最后一条消息作为摘要
    },
    {
        "sender": user_proxy,
        "recipient": research_assistant,
        "message": "分析股价波动的原因",
        "carryover": "使用前一个对话的摘要",  # 传递前一个对话的结果
        "summary_method": "reflection_with_llm"  # 使用 LLM 反思并总结
    },
    {
        "sender": user_proxy,
        "recipient": writer,
        "message": "撰写一篇分析报告",
        "carryover": "包含数据和原因分析"
    }
]

# 执行顺序对话
chat_results = initiate_chats(chat_configs)
```

---

## 五、代码执行

AutoGen 的核心特性之一是能够自动生成和执行代码。

### 5.1 本地代码执行

```python
from autogen_agentchat.agents import UserProxyAgent
from autogen_ext.code_executors import LocalCommandLineCodeExecutor

# 创建代码执行器
code_executor = LocalCommandLineCodeExecutor(
    work_dir="./code_runs",  # 代码执行目录
    timeout=60  # 超时时间（秒）
)

# 创建用户代理（带代码执行）
user_proxy = UserProxyAgent(
    name="用户代理",
    human_input_mode="NEVER",
    code_execution_config={"executor": code_executor}
)

assistant = AssistantAgent(
    name="助手",
    model_client=model_client,
    system_message="""你是一个编程专家。
    当需要解决问题时，编写 Python 代码来解决。
    将代码放在 ```python ``` 代码块中。
    """
)

async def main():
    await Console(
        user_proxy.initiate_chat(
            assistant,
            message="计算 1 到 100 的素数之和"
        )
    )

asyncio.run(main())
```

### 5.2 Docker 代码执行（更安全）

```python
from autogen_ext.code_executors import DockerCommandLineCodeExecutor

# 创建 Docker 代码执行器
docker_executor = DockerCommandLineCodeExecutor(
    work_dir="./code_runs",
    image="python:3.11-slim"  # 使用 Python Docker 镜像
)

user_proxy = UserProxyAgent(
    name="用户代理",
    human_input_mode="NEVER",
    code_execution_config={"executor": docker_executor}
)
```

### 5.3 代码执行示例

```python
# 完整的代码生成和执行示例
import asyncio
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_ext.code_executors import LocalCommandLineCodeExecutor

model_client = OpenAIChatCompletionClient(model="gpt-4o")

# 创建代码执行器
code_executor = LocalCommandLineCodeExecutor(work_dir="./runs")

# 创建助手（负责写代码）
coder = AssistantAgent(
    name="程序员",
    model_client=model_client,
    system_message="""你是一个资深工程师。
    逐步思考，然后只输出可运行的 Python 代码，放在 ```python ``` 代码块中。
    不要包含解释，只输出代码。
    """
)

# 创建用户代理（负责执行代码）
user_proxy = UserProxyAgent(
    name="用户",
    human_input_mode="NEVER",
    code_execution_config={"executor": code_executor},
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("exit")
)

async def main():
    termination = TextMentionTermination("exit", sources=["user"])
    
    await Console(
        user_proxy.initiate_chat(
            coder,
            message="写一个 Python 脚本来打印星星图案，然后运行它",
            termination_condition=termination
        )
    )

asyncio.run(main())
```

---

## 六、实战项目

### 6.1 自动化博客生成引擎

```python
"""
自动化博客生成引擎 - 7 个 Agent 协作
"""

import asyncio
import os
import httpx
from pydantic import BaseModel, Field
from typing import List
from dotenv import load_dotenv
from rich.console import Console
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.ui import Console as UIConsole
from autogen_ext.models.openai import OpenAIChatCompletionClient

load_dotenv()

console = Console()

# 模型客户端
groq_client = OpenAIChatCompletionClient(
    model="gpt-4o-mini",
    api_key=os.getenv('GROQ_API_KEY')
)

perplexity_client = OpenAIChatCompletionClient(
    model="sonar-pro",  # Perplexity 的搜索模型
    api_key=os.getenv('PERPLEXITY_API_KEY'),
    base_url="https://api.perplexity.ai"
)

# 定义输出结构
class Section(BaseModel):
    heading: str
    subtopics: List[str]

class OutlineSchema(BaseModel):
    title: str
    target_audience: str
    technical_depth: str
    include_code: bool
    include_diagrams: bool
    sections: List[Section]

class QuerySchema(BaseModel):
    queries: List[str]

class FactCheckSchema(BaseModel):
    factual_errors: List[str]
    needs_revision: bool

class ReviewSchema(BaseModel):
    clarity_score: int
    structure_score: int
    seo_score: int
    missing_sections: List[str]
    approved: bool

# 网络搜索工具
def search_web(query: str) -> str:
    """使用 Perplexity 搜索网络"""
    response = httpx.post(
        "https://api.perplexity.ai/chat/completions",
        headers={
            "Authorization": f"Bearer {os.getenv('PERPLEXITY_API_KEY')}",
            "Content-Type": "application/json"
        },
        json={
            "model": "sonar-pro",
            "messages": [
                {"role": "system", "content": "搜索并提供准确的信息。"},
                {"role": "user", "content": query}
            ]
        }
    )
    return response.json()["choices"][0]["message"]["content"]

# Agent 1：大纲架构师
outline_agent = AssistantAgent(
    name="OutlineArchitect",
    model_client=groq_client,
    system_message="""你是一个资深技术编辑。
    生成结构化的博客大纲。
    规则：
    - 定义目标读者
    - 定义技术深度
    - 决定是否需要代码示例
    - 决定是否需要图表
    - 创建逻辑清晰的章节划分
    返回严格匹配 schema 的 JSON。
    """,
    description="创建博客大纲"
)

# Agent 2：查询生成器
query_agent = AssistantAgent(
    name="QueryGenerator",
    model_client=groq_client,
    system_message="""根据大纲生成具体的搜索查询。
    返回 JSON 格式：{"queries": ["query1", "query2", ...]}
    """,
    description="生成搜索查询"
)

# Agent 3：研究综合器
research_agent = AssistantAgent(
    name="ResearchSynthesizer",
    model_client=groq_client,
    system_message="""综合搜索结果，创建连贯的摘要。
    移除矛盾信息，保留准确数据。
    """,
    description="综合研究结果"
)

# Agent 4：作家
writer_agent = AssistantAgent(
    name="WriterAgent",
    model_client=groq_client,
    system_message="""根据研究结果撰写引人入胜的博客文章。
    使用生动的语言，包含代码示例。
    """,
    description="撰写博客内容"
)

# Agent 5：事实核查员
fact_checker = AssistantAgent(
    name="FactChecker",
    model_client=groq_client,
    system_message="""你是一个挑剔的事实核查员。
    检查文章中的事实错误。
    返回 JSON：{"factual_errors": [...], "needs_revision": true/false}
    """,
    description="核查事实"
)

# Agent 6：审核员
reviewer = AssistantAgent(
    name="Reviewer",
    model_client=groq_client,
    system_message="""评估文章质量。
    评分标准：清晰度、结构、SEO。
    返回 JSON：{"clarity_score": 1-10, "structure_score": 1-10, "seo_score": 1-10, "approved": true/false}
    """,
    description="审核质量"
)

# Agent 7：HTML 格式化
html_agent = AssistantAgent(
    name="HTMLFormatter",
    model_client=groq_client,
    system_message="""将 Markdown 转换为干净的语义 HTML。
    包含适当的标签和类名。
    """,
    description="格式化 HTML"
)

# 用户代理
user_proxy = UserProxyAgent(
    name="用户",
    human_input_mode="ALWAYS",  # 关键步骤需要人工确认
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE")
)

async def collect_research(queries: List[str]) -> str:
    """收集研究数据"""
    research_results = []
    for query in queries:
        console.print(f"\n[bold yellow]🔎 搜索：[/bold yellow] {query}")
        result = search_web(query)
        research_results.append(result)
    return "\n\n".join(research_results)

async def main():
    console.print("\n[bold green]=== 博客引擎启动 ===[/bold green]\n")
    
    topic = input("输入博客主题：")
    
    # 1. 大纲生成阶段
    console.print("\n[bold blue]阶段 1: 生成大纲[/bold blue]")
    outline_result = await outline_agent.run(
        task=f"为'{topic}'创建博客大纲"
    )
    outline_struct = outline_result.messages[-1].content
    console.print(f"[green]✓ 大纲完成[/green]")
    
    # 人工确认
    input(f"\n按回车继续或提供反馈：")
    
    # 2. 查询生成
    console.print("\n[bold blue]阶段 2: 生成搜索查询[/bold blue]")
    query_result = await query_agent.run(
        task=f"根据以下大纲生成搜索查询：{outline_struct}"
    )
    
    # 3. 研究收集
    console.print("\n[bold blue]阶段 3: 收集研究数据[/bold blue]")
    # 解析查询并执行搜索
    research_data = await collect_research(["AI 发展趋势 2026"])
    
    # 4. 撰写文章
    console.print("\n[bold blue]阶段 4: 撰写文章[/bold blue]")
    draft_result = await writer_agent.run(
        task=f"基于以下研究数据撰写文章：{research_data}"
    )
    
    # 5. 事实核查
    console.print("\n[bold blue]阶段 5: 事实核查[/bold blue]")
    fact_check = await fact_checker.run(
        task=f"核查以下文章的事实：{draft_result.messages[-1].content}"
    )
    
    # 6. 质量审核
    console.print("\n[bold blue]阶段 6: 质量审核[/bold blue]")
    review = await reviewer.run(
        task=f"审核以下文章：{draft_result.messages[-1].content}"
    )
    
    # 7. HTML 格式化
    console.print("\n[bold blue]阶段 7: HTML 格式化[/bold blue]")
    html_result = await html_agent.run(
        task=f"将以下 Markdown 转换为 HTML：{draft_result.messages[-1].content}"
    )
    
    console.print("\n[bold green]=== 博客生成完成 ===[/bold green]")
    print(html_result.messages[-1].content)

if __name__ == "__main__":
    asyncio.run(main())
```

### 6.2 数据分析团队

```python
"""
数据分析团队 - 多 Agent 协作分析
"""

import asyncio
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_ext.code_executors import LocalCommandLineCodeExecutor
from typing import Annotated
import pandas as pd

model_client = OpenAIChatCompletionClient(model="gpt-4o")
code_executor = LocalCommandLineCodeExecutor(work_dir="./analysis")

# Agent 1：数据分析师
data_analyst = AssistantAgent(
    name="数据分析师",
    model_client=model_client,
    system_message="""你是数据分析专家。职责：
    - 分析数据需求并提出分析方法
    - 使用 pandas 和 numpy 编写分析代码
    - 解释统计结果并提供洞察
    - 创建可视化图表
    在分析前始终先解释方法论。
    """,
    description="数据分析专家"
)

# Agent 2：代码审核员
code_reviewer = AssistantAgent(
    name="代码审核员",
    model_client=model_client,
    system_message="""你是资深代码审核员。职责：
    - 检查代码正确性、效率和最佳实践
    - 确保代码遵循 PEP 8 规范并有良好文档
    - 提供建设性反馈和具体建议
    """,
    description="代码审核专家"
)

# Agent 3：报告撰写员
report_writer = AssistantAgent(
    name="报告撰写员",
    model_client=model_client,
    system_message="""你是技术作家。职责：
    - 将分析结果综合成清晰的报告
    - 为非技术受众创建执行摘要
    - 突出关键发现和可操作建议
    - 逻辑清晰地组织内容
    """,
    description="报告撰写专家"
)

# 用户代理
user_proxy = UserProxyAgent(
    name="用户",
    human_input_mode="NEVER",
    code_execution_config={"executor": code_executor},
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE")
)

# 工具函数
def load_data(file_path: Annotated[str, "数据文件路径"]) -> str:
    """加载数据文件"""
    try:
        if file_path.endswith('.csv'):
            df = pd.read_csv(file_path)
        elif file_path.endswith('.xlsx'):
            df = pd.read_excel(file_path)
        else:
            return "不支持的文件格式"
        
        return f"数据加载成功：{df.shape[0]} 行，{df.shape[1]} 列\n列名：{list(df.columns)}"
    except Exception as e:
        return f"加载失败：{str(e)}"

# 添加工具到数据分析师
data_analyst.tools = [load_data]

# 创建团队
team = RoundRobinGroupChat(
    agents=[user_proxy, data_analyst, code_reviewer, report_writer],
    termination_condition=TextMentionTermination("TERMINATE")
)

async def main():
    console_message = """分析销售数据文件 'sales_data.csv'：
    1. 加载并探索数据结构
    2. 计算关键指标（总销售额、平均订单价值等）
    3. 识别销售趋势和模式
    4. 创建可视化图表
    5. 生成综合报告
    """
    
    await Console(
        team.run_stream(task=console_message)
    )

asyncio.run(main())
```

### 6.3 旅行规划助手

```python
"""
旅行规划助手 - 个性化行程生成
"""

import asyncio
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent
from autogen_agentchat.conditions import TextMentionTermination
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient
from typing import Annotated
import os
from dotenv import load_dotenv

load_dotenv()

model_client = OpenAIChatCompletionClient(model="gpt-4o")

# 工具：获取天气
def get_weather(city: Annotated[str, "城市名称"], date: str = None) -> dict:
    """获取指定城市的天气"""
    import requests
    from datetime import datetime
    
    api_key = os.getenv("VISUAL_CROSSING_API_KEY")
    if not date:
        date = datetime.now().strftime("%Y-%m-%d")
    
    url = f"https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline/{city}/{date}?unitGroup=metric&key={api_key}&contentType=json"
    response = requests.get(url)
    data = response.json()
    
    return {
        "city": city,
        "date": date,
        "temperature": data["days"][0]["temp"],
        "condition": data["days"][0]["conditions"]
    }

# 工具：查询航班
def search_flights(origin: Annotated[str, "出发城市"], 
                   destination: Annotated[str, "目的地"],
                   date: Annotated[str, "日期"]) -> str:
    """查询航班信息"""
    # 模拟航班搜索
    return f"找到从{origin}到{destination}的航班，日期：{date}，价格范围：$500-$800"

# 工具：查询酒店
def search_hotels(city: Annotated[str, "城市"], 
                  check_in: Annotated[str, "入住日期"],
                  check_out: Annotated[str, "退房日期"]) -> str:
    """查询酒店信息"""
    # 模拟酒店搜索
    return f"找到{city}的酒店，{check_in}入住，{check_out}退房，价格范围：$100-$300/晚"

# Agent 1：旅行规划师
planner = AssistantAgent(
    name="旅行规划师",
    model_client=model_client,
    tools=[get_weather, search_flights, search_hotels],
    system_message="""你是专业的旅行规划师。职责：
    - 根据用户偏好创建个性化行程
    - 查询天气、航班和酒店信息
    - 提供详细的每日行程
    - 考虑预算和时间限制
    提供实用、可执行的建议。
    """,
    description="旅行规划专家"
)

# Agent 2：预算顾问
budget_advisor = AssistantAgent(
    name="预算顾问",
    model_client=model_client,
    system_message="""你是预算规划专家。职责：
    - 分析和优化旅行预算
    - 提供省钱建议
    - 确保行程在预算范围内
    """,
    description="预算规划专家"
)

# 用户代理
user_proxy = UserProxyAgent(
    name="用户",
    human_input_mode="ALWAYS",  # 总是询问用户
    is_termination_msg=lambda x: x.get("content", "").rstrip().endswith("TERMINATE")
)

async def main():
    termination = TextMentionTermination("TERMINATE", sources=["用户"])
    
    await Console(
        user_proxy.initiate_chat(
            planner,
            message="""我想计划一次日本之旅：
            - 时间：7 天
            - 预算：$3000
            - 兴趣：文化体验、美食、自然
            - 出发地：北京
            请帮我规划详细行程。""",
            termination_condition=termination
        )
    )

asyncio.run(main())
```

---

## 七、AutoGen Studio（可视化界面）

### 7.1 安装和启动

```bash
# 安装 AutoGen Studio
pip install autogenstudio

# 启动服务
autogenstudio ui --port 8080 --appdir ./my_agents

# 访问 http://localhost:8080
```

### 7.2 主要功能

| 功能模块 | 描述 |
|---------|------|
| **Team Builder** | 拖拽式 Agent 团队构建 |
| **Playground** | 交互式测试环境 |
| **Gallery** | 社区组件库 |
| **Labs** | 实验性功能 |
| **Deploy** | 导出为 Python 代码 |

### 7.3 使用示例

1. **创建 Agent**：在 Team Builder 中定义角色、系统消息、工具
2. **配置工作流**：设置 Agent 之间的对话模式
3. **测试运行**：在 Playground 中测试 Agent 团队
4. **导出部署**：导出为 Python 代码用于生产环境

---

## 八、高级特性

### 8.1 记忆系统

```python
from autogen_ext.memory import ChatCompletionMemory

# 创建记忆组件
memory = ChatCompletionMemory(
    model_client=model_client,
    max_tokens=1000
)

agent = AssistantAgent(
    name="助手",
    model_client=model_client,
    memory=memory  # 添加记忆
)
```

### 8.2 流式输出

```python
# 启用流式输出
agent = AssistantAgent(
    name="助手",
    model_client=model_client,
    model_client_stream=True
)

# 流式运行
async for message in agent.run_stream(task="写一首诗"):
    print(message.content, end="", flush=True)
```

### 8.3 嵌套聊天

```python
# Agent 可以启动内部对话来"思考"
inner_agent = AssistantAgent(
    name="内部助手",
    model_client=model_client
)

outer_agent = AssistantAgent(
    name="外部代理",
    model_client=model_client,
    system_message="""你可以启动内部对话来深入思考问题。
    使用 initiate_chat() 与内部助手交流。
    """
)
```

### 8.4 分布式运行时

```python
from autogen_core import AgentRuntime
from autogen_ext.runtimes.grpc import GrpcAgentRuntime

# 创建分布式运行时
runtime = GrpcAgentRuntime(host="localhost", port=50051)

# 注册 Agent
await runtime.register_factory(
    type="assistant",
    factory=lambda: AssistantAgent(...)
)

# 启动运行时
await runtime.start()
```

---

## 九、最佳实践

### 9.1 Agent 设计原则

```python
# ✅ 推荐：明确的角色定义
agent = AssistantAgent(
    name="数据分析师",
    system_message="""你是数据分析专家。
    职责：
    1. 分析数据结构
    2. 计算统计指标
    3. 创建可视化
    完成后回复 'TERMINATE'
    """
)

# ❌ 避免：模糊的角色
bad_agent = AssistantAgent(
    name="助手",
    system_message="帮助解决问题"  # 太模糊
)
```

### 9.2 终止条件

```python
from autogen_agentchat.conditions import (
    TextMentionTermination,
    TimeoutTermination,
    MaxMessageTermination
)

# 文本终止
termination1 = TextMentionTermination("TERMINATE")

# 超时终止
termination2 = TimeoutTermination(timeout_seconds=300)

# 最大消息数终止
termination3 = MaxMessageTermination(max_messages=20)

# 组合终止条件
from autogen_agentchat.conditions import SourceMatchTermination
termination = termination1 | termination2 | termination3
```

### 9.3 错误处理

```python
try:
    result = await agent.run(task="复杂任务")
except Exception as e:
    print(f"错误：{e}")
    # 实现重试或降级逻辑
```

### 9.4 成本控制

```python
# 使用较便宜的模型进行简单任务
simple_agent = AssistantAgent(
    name="简单任务",
    model_client=OpenAIChatCompletionClient(model="gpt-4o-mini")
)

# 限制消息数量
team = RoundRobinGroupChat(
    agents=[...],
    max_turns=10  # 限制最多 10 轮对话
)

# 启用缓存
# AutoGen 会自动缓存相同请求的响应
```

---

## 十、总结

AutoGen 作为微软研究院开源的多 Agent 协作框架，提供了灵活、强大的对话式 AI 系统构建能力。

**核心优势：**
- ✅ **对话驱动**：自然的对话式协作
- ✅ **代码执行**：自动生成和执行代码
- ✅ **灵活架构**：支持多种对话模式
- ✅ **人机协作**：完善的人在回路支持
- ✅ **可扩展性**：支持分布式运行时
- ✅ **可视化工具**：AutoGen Studio 低代码界面

**学习路径建议：**

1. **入门**：安装 → 第一个 Agent → 理解对话模式
2. **进阶**：工具集成 → 代码执行 → 群聊编排
3. **高级**：自定义 Agent → 分布式运行时 → 性能优化
4. **生产**：错误处理 → 监控 → 部署

**重要说明**：2026 年微软推出了新的 **Microsoft Agent Framework**，AutoGen 用户可参考迁移指南过渡到新框架：https://aka.ms/autogen-to-af

**参考资源：**
- 官方文档：https://microsoft.github.io/autogen/
- GitHub：https://github.com/microsoft/autogen
- AutoGen Studio：https://microsoft.github.io/autogen/stable/
- Discord 社区：https://discord.gg/pAbnFJrkgZ

开始使用 AutoGen 构建你的多 Agent 协作系统吧！🚀
