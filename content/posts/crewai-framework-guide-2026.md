+++
title = "CrewAI 框架完全指南 2026：多 Agent 协作系统实战"
date = "2026-03-18T15:00:00+08:00"

[taxonomies]
tags = ["CrewAI", "AI Agent", "多 Agent 系统", "Python", "自动化"]

[extra]
summary = "全面介绍 CrewAI 多 Agent 协作框架，包含核心概念、安装配置、Agent 创建、任务编排、工具集成、流程控制等完整知识，附带丰富的实战代码示例和项目案例。"
author = "博主"
+++

# CrewAI 框架完全指南 2026：多 Agent 协作系统实战

在多 Agent AI 系统领域，**CrewAI** 以其独特的角色协作架构和简洁的 API 设计脱颖而出。作为一个从零构建的独立框架，CrewAI 不依赖 LangChain 或其他库，专注于让多个 specialized AI Agent 像人类团队一样协作完成复杂任务。

本文将深入介绍 CrewAI 的核心概念、使用方法和最佳实践，通过完整的代码示例，帮助你掌握构建生产级多 Agent 系统的关键技术。

---

## 一、什么是 CrewAI？

### CrewAI 简介

**CrewAI** 是一个轻量级的 Python 框架，用于编排基于角色的多 Agent 系统。它于 2023 年发布，迅速成为构建协作式 AI 团队的热门选择。

**核心理念**：让 AI Agent 像人类团队一样工作，每个成员有明确的角色、目标和职责，通过协作完成复杂任务。

**官方网站**：https://www.crewai.com/

**GitHub**：https://github.com/crewAIInc/crewAI（20K+ Stars）

**文档**：https://docs.crewai.com/

### 核心特性

| 特性 | 描述 |
|------|------|
| **角色基础** | 每个 Agent 有明确的角色、目标和背景故事 |
| **独立架构** | 不依赖 LangChain，轻量快速 |
| **双工作流** | Crews（自主协作）+ Flows（事件驱动编排） |
| **多模型支持** | OpenAI、Anthropic、Google、Ollama 等 |
| **工具集成** | 内置 50+ 工具，支持自定义工具 |
| **过程控制** | 顺序执行或层级管理两种流程 |
| **记忆系统** | 支持短期和长期记忆 |
| **本地优先** | 原生支持 Ollama 和本地 LLM |

### CrewAI vs 其他框架

| 框架 | 架构 | 学习曲线 | 适用场景 | 依赖 |
|------|------|---------|---------|------|
| **CrewAI** | 角色协作 | 低 | 团队协作型任务 | 无 |
| **LangGraph** | 图结构 | 中 | 复杂工作流 | LangChain |
| **AutoGen** | 对话驱动 | 中 | 研究实验 | 微软生态 |
| **Agno** | 轻量级 | 低 | 快速原型 | 无 |

### 适用场景

- ✅ **内容创作团队**：研究员 + 作家 + 编辑
- ✅ **市场研究**：数据收集 + 分析 + 报告
- ✅ **客户支持**：情感分析 + 知识检索 + 回复生成
- ✅ **代码开发**：架构师 + 开发者 + 测试员
- ✅ **金融分析**：数据收集 + 分析 + 风险评估
- ✅ **教育辅导**：评估 + 规划 + 教学

---

## 二、快速开始

### 2.1 环境要求

- **Python**: 3.10 - 3.13
- **操作系统**: Linux / macOS / Windows
- **内存**: 最低 2GB，推荐 4GB+

### 2.2 安装 CrewAI

```bash
# 创建虚拟环境（推荐）
python -m venv crewai-env
source crewai-env/bin/activate  # Windows: crewai-env\Scripts\activate

# 使用 uv 安装（CrewAI 推荐，更快）
pip install uv
uv tool install crewai

# 或使用 pip 安装
pip install crewai
pip install crewai-tools  # 工具包
```

### 2.3 配置 API 密钥

创建 `.env` 文件：

```bash
# .env
OPENAI_API_KEY=sk-your-openai-api-key
ANTHROPIC_API_KEY=sk-ant-your-anthropic-key
SERPER_API_KEY=your-serper-api-key  # 网络搜索
```

在代码中加载：

```python
from dotenv import load_dotenv
load_dotenv()
```

### 2.4 第一个 Crew

```python
from crewai import Agent, Task, Crew, Process

# 创建 Agent
researcher = Agent(
    role='高级研究分析师',
    goal='发现和分析 {topic} 领域的最新发展',
    backstory='你是一位拥有 10 年经验的人工智能研究专家，擅长识别新兴趋势。',
    verbose=True,
    allow_delegation=False
)

writer = Agent(
    role='技术作家',
    goal='基于研究数据创建详细的报告',
    backstory='你是一位严谨的分析师，擅长将复杂信息转化为清晰的叙述。',
    verbose=True,
    allow_delegation=False
)

# 创建任务
research_task = Task(
    description='研究 {topic} 的最新发展，重点关注：\n1. 最新技术突破\n2. 主要参与者\n3. 市场趋势',
    expected_output='包含 5-8 个关键发现的研究报告',
    agent=researcher
)

write_task = Task(
    description='基于研究结果撰写一篇 comprehensive 报告',
    expected_output='一篇结构完整的技术报告，包含代码示例',
    agent=writer,
    context=[research_task]  # 依赖研究任务的结果
)

# 创建 Crew
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential,  # 顺序执行
    verbose=True
)

# 运行
result = crew.kickoff(inputs={'topic': '生成式 AI'})
print(result)
```

---

## 三、核心组件

### 3.1 Agent（Agent）

Agent 是 CrewAI 的核心，代表具有特定角色的 AI 工作者。

#### 基本 Agent

```python
from crewai import Agent

agent = Agent(
    role='数据分析师',
    goal='从数据中提取有价值的洞察',
    backstory='你是一位经验丰富的数据科学家，擅长发现数据中的模式。',
    verbose=True,           # 显示详细日志
    allow_delegation=False, # 不允许委派任务
    max_iter=15,           # 最大推理迭代次数
    max_rpm=None,          # 速率限制（每分钟请求数）
    cache=True             # 启用缓存
)
```

#### 指定 LLM

```python
from crewai import Agent, LLM

# 使用 OpenAI
agent1 = Agent(
    role='研究员',
    goal='研究主题',
    llm='gpt-4o'  # 直接使用模型名称
)

# 使用 Anthropic
agent2 = Agent(
    role='作家',
    goal='写作',
    llm='claude-sonnet-4-20250514'
)

# 使用 Ollama（本地）
agent3 = Agent(
    role='助手',
    goal='帮助',
    llm=LLM(
        model='ollama/mistral',
        base_url='http://localhost:11434',
        temperature=0.7
    )
)

# 使用 Google Gemini
agent4 = Agent(
    role='分析师',
    goal='分析',
    llm='gemini/gemini-2.0-flash'
)
```

#### Agent 配置参数

| 参数 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `role` | str | 必需 | Agent 的角色 |
| `goal` | str | 必需 | Agent 的目标 |
| `backstory` | str | 必需 | Agent 的背景故事 |
| `verbose` | bool | False | 详细日志 |
| `allow_delegation` | bool | False | 允许委派 |
| `max_iter` | int | 15 | 最大迭代次数 |
| `max_rpm` | int | None | 速率限制 |
| `cache` | bool | True | 启用缓存 |
| `llm` | str/LLM | gpt-4o-mini | 语言模型 |
| `tools` | list | [] | 可用工具 |
| `memory` | bool | False | 启用记忆 |

### 3.2 Task（任务）

Task 代表需要完成的具体工作单元。

#### 基本任务

```python
from crewai import Task

task = Task(
    description='分析 {topic} 的市场趋势',
    expected_output='包含关键洞察的分析报告',
    agent=researcher,
    async_execution=False,  # 同步执行
    output_file='output.md', # 保存到文件
    human_input=True        # 需要人工确认
)
```

#### 任务依赖

```python
# 任务 1：研究
research_task = Task(
    description='研究 {topic}',
    expected_output='研究报告',
    agent=researcher
)

# 任务 2：写作（依赖研究结果）
write_task = Task(
    description='基于研究撰写文章',
    expected_output='完整文章',
    agent=writer,
    context=[research_task]  # 自动接收 research_task 的输出
)

# 任务 3：编辑（依赖写作结果）
edit_task = Task(
    description='编辑和润色文章',
    expected_output='最终版本',
    agent=editor,
    context=[write_task]
)
```

#### 任务配置参数

| 参数 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `description` | str | 必需 | 任务描述 |
| `expected_output` | str | 必需 | 期望的输出 |
| `agent` | Agent | 必需 | 执行任务的 Agent |
| `context` | list | [] | 依赖的其他任务 |
| `async_execution` | bool | False | 异步执行 |
| `output_file` | str | None | 输出文件路径 |
| `output_json` | dict | None | JSON 输出格式 |
| `output_pydantic` | Model | None | Pydantic 输出模型 |
| `human_input` | bool | False | 需要人工输入 |
| `tools` | list | [] | 任务专用工具 |

### 3.3 Crew（团队）

Crew 是 Agent 和任务的编排器。

#### 基本 Crew

```python
from crewai import Crew, Process

crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, write_task, edit_task],
    process=Process.sequential,  # 顺序执行
    verbose=True
)

result = crew.kickoff(inputs={'topic': 'AI 发展'})
```

#### 流程类型

```python
from crewai import Process

# 顺序流程：任务按定义顺序执行
crew1 = Crew(
    agents=[agent1, agent2],
    tasks=[task1, task2],
    process=Process.sequential
)

# 层级流程：Manager Agent 动态分配任务
crew2 = Crew(
    agents=[manager, agent1, agent2],
    tasks=[task1, task2],
    process=Process.hierarchical,
    manager_llm='gpt-4o'  # Manager 使用的模型
)
```

#### Crew 配置参数

| 参数 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `agents` | list | 必需 | Agent 列表 |
| `tasks` | list | 必需 | 任务列表 |
| `process` | Process | sequential | 执行流程 |
| `verbose` | bool | False | 详细日志 |
| `memory` | bool | False | 启用记忆 |
| `cache` | bool | True | 启用缓存 |
| `manager_llm` | str | None | Manager 模型（层级流程） |
| `max_rpm` | int | None | 速率限制 |
| `share_crew` | bool | False | 共享到 CrewAI 社区 |

### 3.4 Tools（工具）

工具让 Agent 能够与外部世界交互。

#### 内置工具

```python
from crewai_tools import (
    SerperDevTool,           # 网络搜索
    ScrapeWebsiteTool,       # 网页抓取
    FileReadTool,            # 文件读取
    FileWriterTool,          # 文件写入
    DirectorySearchTool,     # 目录搜索
    TXTSearchTool,           # 文本搜索
    PDFSearchTool,           # PDF 搜索
    GithubSearchTool,        # GitHub 搜索
    CodeInterpreterTool,     # 代码解释
    DallETool,               # 图像生成
    VisionTool               # 图像理解
)

# 使用工具
search_tool = SerperDevTool()
scrape_tool = ScrapeWebsiteTool()

agent = Agent(
    role='研究员',
    goal='研究主题',
    tools=[search_tool, scrape_tool]
)
```

#### 自定义工具

```python
from crewai_tools import BaseTool
from pydantic import BaseModel, Field

# 定义输入模式
class WeatherInput(BaseModel):
    city: str = Field(description="城市名称")

# 创建工具
class WeatherTool(BaseTool):
    name: str = "获取天气"
    description: str = "获取指定城市的天气信息"
    args_schema: type[BaseModel] = WeatherInput
    
    def _run(self, city: str) -> str:
        # 调用天气 API
        import requests
        api_key = "your-api-key"
        url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
        response = requests.get(url)
        data = response.json()
        
        if data.get("weather"):
            temp = data["main"]["temp"] - 273.15
            desc = data["weather"][0]["description"]
            return f"{city}的天气：{desc}，温度：{temp:.1f}°C"
        return "无法获取天气信息"

# 使用工具
weather_tool = WeatherTool()
agent = Agent(
    role='助手',
    goal='提供帮助',
    tools=[weather_tool]
)
```

#### 函数装饰器工具

```python
from crewai_tools import tool

@tool
def calculate(expression: str) -> str:
    """计算数学表达式。
    
    Args:
        expression: 数学表达式，如 "2 + 2"
        
    Returns:
        计算结果
    """
    try:
        result = eval(expression)
        return f"计算结果：{result}"
    except Exception as e:
        return f"计算错误：{str(e)}"

@tool
def get_stock_price(symbol: str) -> str:
    """获取股票价格。
    
    Args:
        symbol: 股票代码
        
    Returns:
        当前股价
    """
    import yfinance as yf
    stock = yf.Ticker(symbol)
    price = stock.history(period='1d')['Close'].iloc[-1]
    return f"{symbol} 当前价格：${price:.2f}"

agent = Agent(
    role='金融分析师',
    goal='分析股票',
    tools=[calculate, get_stock_price]
)
```

---

## 四、高级特性

### 4.1 规划（Planning）

CrewAI 的规划功能为 Agent 团队生成共享的工作路线图。

```python
from crewai import Crew, Agent, Task, Planning

# 启用规划
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential,
    planning=True,  # 启用规划
    planning_llm='gpt-4o'  # 规划使用的模型
)

# 规划会先生成步骤大纲，然后 Agent 按照计划执行
result = crew.kickoff(inputs={'topic': '量子计算'})
```

**规划的优势：**
- 所有 Agent 共享同一路线图
- 减少重复工作和偏离主题
- 提高工作流的清晰度和可预测性
- 适合多阶段任务（研究→写作→编辑）

### 4.2 记忆系统

```python
from crewai import Crew, Agent, Task
from crewai.memory import ShortTermMemory, LongTermMemory

# 创建记忆系统
short_memory = ShortTermMemory()
long_memory = LongTermMemory()

crew = Crew(
    agents=[agent1, agent2],
    tasks=[task1, task2],
    memory=True,  # 启用记忆
    short_term_memory=short_memory,
    long_term_memory=long_memory
)
```

### 4.3 缓存

```python
from crewai import Crew

crew = Crew(
    agents=[agent1, agent2],
    tasks=[task1, task2],
    cache=True  # 启用缓存（默认开启）
)

# 缓存会保存 Agent 的响应，避免重复调用 LLM
# 相同输入会直接返回缓存结果
```

### 4.4 速率限制

```python
crew = Crew(
    agents=[agent1, agent2],
    tasks=[task1, task2],
    max_rpm=100  # 限制每分钟 100 次请求
)

# 防止 API 配额超限
```

### 4.5 人工审核

```python
task = Task(
    description='撰写重要报告',
    expected_output='最终报告',
    agent=writer,
    human_input=True  # 需要人工确认
)

# 运行时会暂停，等待人工输入
```

### 4.6 异步执行

```python
# 任务异步执行（不等待前一个任务完成）
task1 = Task(
    description='研究主题 A',
    agent=agent1,
    async_execution=True
)

task2 = Task(
    description='研究主题 B',
    agent=agent2,
    async_execution=True
)

crew = Crew(
    agents=[agent1, agent2],
    tasks=[task1, task2],
    process=Process.sequential
)
```

---

## 五、项目结构

### 5.1 使用 CLI 创建项目

```bash
# 创建新项目
crewai create project my_crew

# 项目结构
my_crew/
├── .env                  # 环境变量
├── .gitignore
├── pyproject.toml        # 项目依赖
├── README.md
├── knowledge/            # 知识库文件
├── src/
│   └── my_crew/
│       ├── __init__.py
│       ├── main.py       # 入口文件
│       ├── crew.py       # Crew 定义
│       ├── tools/        # 自定义工具
│       │   ├── __init__.py
│       │   └── custom_tool.py
│       └── config/
│           ├── agents.yaml  # Agent 配置
│           └── tasks.yaml   # 任务配置
```

### 5.2 YAML 配置方式

#### agents.yaml

```yaml
researcher:
  role: >
    高级研究分析师
  goal: >
    发现和分析 {topic} 领域的最新发展
  backstory: >
    你是一位拥有 10 年经验的人工智能研究专家，
    擅长识别新兴趋势和关键技术突破。
  verbose: true
  allow_delegation: false

writer:
  role: >
    技术作家
  goal: >
    基于研究数据创建详细的报告
  backstory: >
    你是一位严谨的分析师，擅长将复杂信息
    转化为清晰的叙述。
  verbose: true
```

#### tasks.yaml

```yaml
research_task:
  description: >
    研究 {topic} 的最新发展，重点关注：
    1. 最新技术突破
    2. 主要参与者和公司
    3. 市场趋势和预测
  expected_output: >
    包含 5-8 个关键发现的研究报告，
    每个发现附带来源链接
  agent: researcher

write_task:
  description: >
    基于研究结果撰写一篇 comprehensive 报告
  expected_output: >
    一篇结构完整的技术报告，包含代码示例
  agent: writer
  context:
    - research_task
```

#### crew.py

```python
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task

@CrewBase
class MyCrew:
    """MyCrew crew"""
    
    agents_config = 'config/agents.yaml'
    tasks_config = 'config/tasks.yaml'
    
    @agent
    def researcher(self) -> Agent:
        return Agent(config=self.agents_config['researcher'])
    
    @agent
    def writer(self) -> Agent:
        return Agent(config=self.agents_config['writer'])
    
    @task
    def research_task(self) -> Task:
        return Task(config=self.tasks_config['research_task'])
    
    @task
    def write_task(self) -> Task:
        return Task(config=self.tasks_config['write_task'])
    
    @crew
    def crew(self) -> Crew:
        return Crew(
            agents=self.agents,
            tasks=self.tasks,
            process=Process.sequential,
            verbose=True
        )
```

#### main.py

```python
#!/usr/bin/env python
from my_crew.crew import MyCrew

def run():
    """运行 Crew"""
    inputs = {
        'topic': '生成式 AI 的最新发展'
    }
    result = MyCrew().crew().kickoff(inputs=inputs)
    print(result)

if __name__ == "__main__":
    run()
```

---

## 六、实战项目

### 6.1 内容创作团队

```python
"""
内容创作团队 - 研究员 + 作家 + 编辑
"""

from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool, ScrapeWebsiteTool
from dotenv import load_dotenv

load_dotenv()

# 工具
search_tool = SerperDevTool()
scrape_tool = ScrapeWebsiteTool()

# Agent 1：研究员
researcher = Agent(
    role='首席研究员',
    goal='深入调研{topic}，收集关键信息和数据',
    backstory='你是一位资深研究员，擅长从权威来源获取准确信息。',
    tools=[search_tool, scrape_tool],
    verbose=True,
    allow_delegation=False
)

# Agent 2：作家
writer = Agent(
    role='资深作家',
    goal='根据研究结果撰写引人入胜的文章',
    backstory='你是一位经验丰富的技术作家，擅长用生动的语言解释复杂概念。',
    verbose=True,
    allow_delegation=False
)

# Agent 3：编辑
editor = Agent(
    role='主编',
    goal='审核和优化文章质量',
    backstory='你是一位严格的主编，确保每篇文章都准确、清晰、有深度。',
    verbose=True
)

# 任务
research_task = Task(
    description='''研究{topic}的最新发展：
    1. 搜索最新的新闻和文章
    2. 收集统计数据和案例
    3. 整理关键趋势和洞察''',
    expected_output='详细的研究报告，包含来源链接',
    agent=researcher
)

write_task = Task(
    description='''基于研究报告撰写文章：
    1. 设计清晰的文章结构
    2. 用生动的语言撰写内容
    3. 添加代码示例和图表说明''',
    expected_output='2000-3000 字的完整文章',
    agent=writer,
    context=[research_task]
)

edit_task = Task(
    description='''编辑和优化文章：
    1. 检查事实准确性
    2. 优化语言表达
    3. 添加吸引人的标题和摘要''',
    expected_output='最终发布版本',
    agent=editor,
    context=[write_task]
)

# 创建 Crew
crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, write_task, edit_task],
    process=Process.sequential,
    verbose=True
)

# 运行
if __name__ == "__main__":
    result = crew.kickoff(inputs={'topic': 'AI Agent 技术的发展趋势'})
    print("\n" + "="*60)
    print("最终文章:")
    print("="*60)
    print(result)
```

### 6.2 市场研究系统

```python
"""
市场研究系统 - 数据收集 + 分析 + 报告
"""

from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool, ScrapeWebsiteTool, FileReadTool
from pydantic import BaseModel, Field
from typing import List

# 定义输出结构
class MarketReport(BaseModel):
    market_size: str = Field(description="市场规模")
    growth_rate: str = Field(description="增长率")
    key_players: List[str] = Field(description="主要参与者")
    trends: List[str] = Field(description="市场趋势")
    opportunities: List[str] = Field(description="市场机会")
    threats: List[str] = Field(description="市场威胁")
    summary: str = Field(description="总结分析")

# Agent
data_collector = Agent(
    role='数据收集专家',
    goal='收集{industry}的市场数据和统计信息',
    backstory='你擅长从各种来源获取准确的市场数据。',
    tools=[SerperDevTool(), ScrapeWebsiteTool()],
    verbose=True
)

analyst = Agent(
    role='市场分析师',
    goal='分析市场数据，提取关键洞察',
    backstory='你是一位经验丰富的分析师，擅长发现数据背后的趋势。',
    verbose=True
)

reporter = Agent(
    role='报告撰写人',
    goal='创建结构化的市场研究报告',
    backstory='你擅长将复杂分析转化为清晰的报告。',
    verbose=True
)

# 任务
collect_task = Task(
    description='''收集{industry}的市场信息：
    1. 市场规模和增长率
    2. 主要公司和市场份额
    3. 最新发展和并购活动''',
    expected_output='详细的数据汇总',
    agent=data_collector
)

analyze_task = Task(
    description='''分析收集的数据：
    1. 进行 SWOT 分析
    2. 识别关键趋势
    3. 评估市场机会''',
    expected_output='结构化分析结果',
    agent=analyst,
    context=[collect_task],
    output_pydantic=MarketReport
)

report_task = Task(
    description='''撰写市场研究报告：
    1. 执行摘要
    2. 市场概况
    3. 竞争格局
    4. 趋势和预测
    5. 建议和结论''',
    expected_output='完整的市场研究报告',
    agent=reporter,
    context=[analyze_task]
)

# Crew
crew = Crew(
    agents=[data_collector, analyst, reporter],
    tasks=[collect_task, analyze_task, report_task],
    process=Process.sequential,
    verbose=True
)

if __name__ == "__main__":
    result = crew.kickoff(inputs={'industry': '电动汽车市场'})
    print(result)
```

### 6.3 客户支持自动化

```python
"""
客户支持自动化 - 情感分析 + 知识检索 + 回复生成
"""

from crewai import Agent, Task, Crew, Process
from crewai_tools import TXTSearchTool

# 知识库工具
knowledge_tool = TXTSearchTool(
    txt='knowledge_base.txt'  # 产品文档
)

# Agent 1：情感分析
sentiment_analyzer = Agent(
    role='情感分析专家',
    goal='分析客户消息的情感和紧急程度',
    backstory='你擅长理解客户情绪，能准确判断问题的紧急性。',
    verbose=True
)

# Agent 2：知识检索
knowledge_agent = Agent(
    role='知识检索专家',
    goal='从知识库中找到相关解决方案',
    backstory='你熟悉所有产品文档，能快速找到准确信息。',
    tools=[knowledge_tool],
    verbose=True
)

# Agent 3：回复生成
response_agent = Agent(
    role='客户支持代表',
    goal='生成专业、友好的回复',
    backstory='你是一位经验丰富的客服代表，擅长解决客户问题。',
    verbose=True
)

# Agent 4：合规审核
compliance_agent = Agent(
    role='合规审核员',
    goal='确保回复符合公司政策',
    backstory='你负责确保所有客户沟通都符合公司标准。',
    verbose=True
)

# 任务
sentiment_task = Task(
    description='分析客户消息："{customer_message}"\n判断情感倾向和紧急程度',
    expected_output='情感分析结果（正面/负面/中性，紧急程度）',
    agent=sentiment_analyzer
)

search_task = Task(
    description='根据客户问题检索相关知识',
    expected_output='相关的解决方案和文档片段',
    agent=knowledge_agent,
    context=[sentiment_task]
)

response_task = Task(
    description='基于检索结果生成客户回复',
    expected_output='专业、友好的客服回复',
    agent=response_agent,
    context=[sentiment_task, search_task]
)

compliance_task = Task(
    description='审核回复是否符合公司政策',
    expected_output='审核通过的最终回复',
    agent=compliance_agent,
    context=[response_task]
)

# Crew
crew = Crew(
    agents=[sentiment_analyzer, knowledge_agent, response_agent, compliance_agent],
    tasks=[sentiment_task, search_task, response_task, compliance_task],
    process=Process.sequential,
    verbose=True
)

if __name__ == "__main__":
    result = crew.kickoff(inputs={
        'customer_message': '我的订单已经一周了还没到，非常着急！'
    })
    print("最终回复:")
    print(result)
```

### 6.4 代码开发团队

```python
"""
代码开发团队 - 架构师 + 开发者 + 测试员
"""

from crewai import Agent, Task, Crew, Process
from crewai_tools import CodeInterpreterTool, FileReadTool, FileWriterTool

# Agent
architect = Agent(
    role='软件架构师',
    goal='设计清晰的系统架构',
    backstory='你是一位资深架构师，擅长设计可扩展的系统。',
    verbose=True
)

developer = Agent(
    role='高级开发者',
    goal='实现高质量的功能代码',
    backstory='你是一位经验丰富的开发者，擅长编写优雅的代码。',
    tools=[FileWriterTool(), FileReadTool()],
    verbose=True
)

tester = Agent(
    role='测试工程师',
    goal='编写全面的测试用例',
    backstory='你注重细节，确保代码质量。',
    tools=[CodeInterpreterTool()],
    verbose=True
)

reviewer = Agent(
    role='代码审查员',
    goal='审查代码质量和最佳实践',
    backstory='你熟悉各种设计模式和编码规范。',
    verbose=True
)

# 任务
design_task = Task(
    description='为{project_requirement}设计系统架构',
    expected_output='架构设计文档，包含组件图和接口定义',
    agent=architect
)

develop_task = Task(
    description='根据架构设计实现功能',
    expected_output='完整的源代码文件',
    agent=developer,
    context=[design_task]
)

test_task = Task(
    description='为实现的代码编写测试',
    expected_output='单元测试和集成测试代码',
    agent=tester,
    context=[develop_task]
)

review_task = Task(
    description='审查代码质量',
    expected_output='代码审查报告和改进建议',
    agent=reviewer,
    context=[develop_task, test_task]
)

# Crew
crew = Crew(
    agents=[architect, developer, tester, reviewer],
    tasks=[design_task, develop_task, test_task, review_task],
    process=Process.sequential,
    verbose=True
)

if __name__ == "__main__":
    result = crew.kickoff(inputs={
        'project_requirement': '创建一个 REST API，支持用户注册、登录和个人信息管理'
    })
    print(result)
```

---

## 七、Flows：事件驱动编排

CrewAI Flows 提供了更高级的事件驱动编排能力。

```python
from crewai.flow.flow import Flow, listen, start, router
from crewai import Agent, Task, Crew

class ContentFlow(Flow):
    
    @start()
    def research(self):
        """开始研究任务"""
        researcher = Agent(
            role='研究员',
            goal='研究{topic}'
        )
        task = Task(
            description='研究{topic}',
            agent=researcher
        )
        crew = Crew(agents=[researcher], tasks=[task])
        result = crew.kickoff(inputs={'topic': 'AI 趋势'})
        return result
    
    @router(research)
    def route_by_sentiment(self):
        """根据情感路由"""
        # 分析研究结果的情感
        # 返回不同的路径
        return "positive" if "增长" in str(self.research) else "negative"
    
    @listen("positive")
    def write_optimistic(self):
        """撰写乐观的文章"""
        writer = Agent(role='作家', goal='撰写积极的文章')
        task = Task(description='基于研究写文章', agent=writer)
        crew = Crew(agents=[writer], tasks=[task])
        return crew.kickoff()
    
    @listen("negative")
    def write_cautious(self):
        """撰写谨慎的文章"""
        writer = Agent(role='作家', goal='撰写谨慎的文章')
        task = Task(description='基于研究写文章', agent=writer)
        crew = Crew(agents=[writer], tasks=[task])
        return crew.kickoff()

# 运行 Flow
flow = ContentFlow()
flow.kickoff()
```

---

## 八、最佳实践

### 8.1 Agent 设计原则

```python
# ✅ 推荐：单一职责
researcher = Agent(role='研究员', goal='研究主题')
writer = Agent(role='作家', goal='撰写文章')

# ❌ 避免：职责过多
bad_agent = Agent(
    role='研究员兼作家兼编辑',  # 太宽泛
    goal='做所有事情'
)
```

### 8.2 任务分解

```python
# ✅ 推荐：小任务
task1 = Task(description='收集数据', agent=agent1)
task2 = Task(description='分析数据', agent=agent2)
task3 = Task(description='生成报告', agent=agent3)

# ❌ 避免：大任务
bad_task = Task(
    description='完成整个项目',  # 太模糊
    agent=agent1
)
```

### 8.3 成本控制

```python
# 使用不同模型优化成本
senior_agent = Agent(
    role='高级分析师',
    llm='gpt-4o'  # 复杂任务用高端模型
)

junior_agent = Agent(
    role='数据收集员',
    llm='gpt-4o-mini'  # 简单任务用便宜模型
)

# 限制迭代次数
agent = Agent(
    role='研究员',
    max_iter=10  # 避免无限循环
)

# 启用缓存
crew = Crew(
    agents=[...],
    cache=True  # 避免重复调用
)
```

### 8.4 错误处理

```python
from crewai import Crew

try:
    crew = Crew(
        agents=[agent1, agent2],
        tasks=[task1, task2],
        max_rpm=100  # 速率限制
    )
    result = crew.kickoff(inputs={'topic': 'AI'})
except Exception as e:
    print(f"错误：{e}")
    # 实现重试逻辑或降级策略
```

### 8.5 性能优化

```python
# 并行执行独立任务
task1 = Task(description='研究 A', agent=agent1, async_execution=True)
task2 = Task(description='研究 B', agent=agent2, async_execution=True)

# 使用本地模型减少延迟
local_agent = Agent(
    role='助手',
    llm='ollama/mistral'  # 本地运行
)

# 减少 verbose 输出（生产环境）
crew = Crew(
    agents=[...],
    verbose=False  # 生产环境关闭详细日志
)
```

---

## 九、部署和监控

### 9.1 FastAPI 集成

```python
from fastapi import FastAPI, BackgroundTasks
from crewai import Crew
import uuid

app = FastAPI()

# 存储运行结果
results = {}

@app.post("/run-crew")
async def run_crew(topic: str, background_tasks: BackgroundTasks):
    """异步运行 Crew"""
    job_id = str(uuid.uuid4())
    
    async def process():
        crew = Crew(agents=[...], tasks=[...])
        result = crew.kickoff(inputs={'topic': topic})
        results[job_id] = result
    
    background_tasks.add_task(process)
    return {"job_id": job_id, "status": "processing"}

@app.get("/result/{job_id}")
async def get_result(job_id: str):
    """获取结果"""
    if job_id in results:
        return {"status": "completed", "result": results[job_id]}
    return {"status": "processing"}

# 运行：uvicorn main:app --reload
```

### 9.2 日志和监控

```python
import logging

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)

# CrewAI 内置日志
crew = Crew(
    agents=[...],
    tasks=[...],
    verbose=True  # 启用详细日志
)

# 监控 Token 使用
# 通过 LLM 提供商的 dashboard 监控
```

### 9.3 Docker 部署

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV PYTHONUNBUFFERED=1

CMD ["python", "main.py"]
```

---

## 十、总结

CrewAI 以其独特的角色协作架构，为多 Agent 系统开发提供了简洁而强大的解决方案。

**核心优势：**
- ✅ **角色基础**：清晰的职责分工
- ✅ **独立架构**：不依赖 LangChain，轻量快速
- ✅ **双工作流**：Crews + Flows 灵活编排
- ✅ **多模型支持**：23+ 提供商
- ✅ **工具丰富**：50+ 内置工具
- ✅ **本地优先**：Ollama 原生支持
- ✅ **生产就绪**：速率限制、缓存、记忆

**学习路径建议：**

1. **入门**：安装 → 第一个 Crew → 理解 Agent/Task/Crew
2. **进阶**：工具集成 → YAML 配置 → 流程控制
3. **高级**：Flows → 自定义工具 → 性能优化
4. **生产**：部署 → 监控 → 错误处理

**参考资源：**
- 官方文档：https://docs.crewai.com/
- GitHub：https://github.com/crewAIInc/crewAI
- 社区：https://discord.gg/crewai
- 工具库：https://github.com/crewAIInc/crewAI-tools

开始使用 CrewAI 构建你的多 Agent 协作系统吧！🚀
