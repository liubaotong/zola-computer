+++
title = "AgentScope 完全指南 2026：阿里巴巴多 Agent 协作框架详解"
date = "2026-03-19T11:00:00+08:00"

[taxonomies]
tags = ["AgentScope", "多 Agent", "阿里巴巴", "Python", "AI 框架", "ModelScope"]
categories = ["编程", "人工智能"]

[extra]
summary = "全面介绍阿里巴巴开源的 AgentScope 多 Agent 协作框架，包含核心架构、安装配置、Agent 创建、多 Agent 协作、MCP 集成、Agentic RL、语音智能体等完整知识，附带丰富的实战代码示例。"
author = "博主"
+++

# AgentScope 完全指南 2026：阿里巴巴多 Agent 协作框架详解

在 AI Agent 开发领域，**AgentScope** 作为阿里巴巴开源的生产级多 Agent 框架，正以其开发者友好的设计和强大的功能迅速崛起。从简单的对话机器人到复杂的多 Agent 协作系统，AgentScope 提供了一套完整的工具链，让开发者能够在 5 分钟内开始构建智能体应用。

本文将深入介绍 AgentScope 的核心架构、使用方法和最佳实践，通过完整的代码示例，帮助你掌握构建生产级多 Agent 系统的关键技术。

---

## 一、什么是 AgentScope？

### 1.1 AgentScope 简介

**AgentScope** 是阿里巴巴 ModelScope 团队开源的多 Agent 协作框架，专注于开发者体验和生产级部署。它于 2024 年首次发布，2025 年推出 1.0 版本，现已成为构建多 Agent 应用的主流选择之一。

**核心理念**：让开发者能够简单、灵活、可靠地构建和部署多 Agent 应用。

**GitHub**：https://github.com/agentscope-ai/agentscope

**文档**：https://doc.agentscope.io/

**Discord**：https://discord.gg/agentscope

### 1.2 核心特性

| 特性 | 描述 |
|------|------|
| **简洁性** | 5 分钟内开始构建，内置 ReAct Agent、工具、记忆、人机交互等 |
| **可扩展性** | 大量工具、记忆、可观测性集成；支持 MCP 和 A2A 协议 |
| **生产就绪** | 支持本地、云端 Serverless、K8s 集群部署；内置 OpenTelemetry |
| **多模态** | 支持文本、语音、图像多模态交互 |
| **灵活编排** | MsgHub 消息中心、顺序/并发/辩论等多种协作模式 |

### 1.3 2025-2026 新特性

```
┌─────────────────────────────────────────────────────────┐
│              AgentScope 2025-2026 新特性                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  🎙️ 实时语音智能体 (Realtime Voice Agent)              │
│     • 支持语音理解和语音回复                            │
│     • 多智能体语音交互（狼人杀游戏）                    │
│     • TTS 文本转语音支持                                │
│                                                         │
│  🔗 A2A (Agent-to-Agent) 协议支持                       │
│     • 跨框架智能体互操作                                │
│     • 与 LangChain、AutoGen 等框架互通                  │
│                                                         │
│  🧠 长期记忆增强 (ReMe 集成)                            │
│     • 数据库支持 & 记忆压缩                             │
│     • 持久化记忆存储                                    │
│                                                         │
│  🎯 Agentic RL (通过 Trinity-RFT)                       │
│     • 强化学习训练智能体                                │
│     • 数学推理、游戏策略等场景                          │
│                                                         │
│  🛠️ Anthropic Agent Skill 支持                          │
│     • 兼容 Claude Skills 规范                           │
│     • 技能市场集成                                      │
│                                                         │
│  📊 可观测性增强                                         │
│     • OpenTelemetry 语义规范                            │
│     • AgentScope Studio 可视化                          │
│     • LoongSuite 集成                                   │
└─────────────────────────────────────────────────────────┘
```

### 1.4 AgentScope vs 其他框架

| 特性 | AgentScope | LangChain | CrewAI | AutoGen |
|------|:---------:|:---------:|:------:|:-------:|
| **多 Agent 协作** | ✅✅ | ✅ | ✅✅ | ✅✅ |
| **语音支持** | ✅✅ | ⚠️ | ❌ | ⚠️ |
| **MCP 集成** | ✅ | ✅ | ⚠️ | ⚠️ |
| **A2A 协议** | ✅ | ❌ | ❌ | ❌ |
| **Agentic RL** | ✅ | ❌ | ❌ | ❌ |
| **可视化 Studio** | ✅ | ✅ | ❌ | ⚠️ |
| **学习曲线** | 低 | 高 | 低 | 中 |
| **生产部署** | ✅✅ | ✅ | ⚠️ | ⚠️ |
| **中文支持** | ✅✅ | ⚠️ | ⚠️ | ⚠️ |

---

## 二、快速开始

### 2.1 环境要求

- **Python**: 3.10+
- **操作系统**: Linux / macOS / Windows
- **内存**: 最低 2GB，推荐 4GB+

### 2.2 安装 AgentScope

```bash
# 方式一：从 PyPI 安装（推荐）
pip install agentscope
# 或使用 uv（更快）
uv pip install agentscope

# 方式二：从源码安装
git clone -b main https://github.com/agentscope-ai/agentscope.git
cd agentscope
pip install -e .
# 或使用 uv
# uv pip install -e .

# 验证安装
python -c "import agentscope; print(agentscope.__version__)"
```

### 2.3 配置 API 密钥

```bash
# 阿里云 DashScope（通义千问）
export DASHSCOPE_API_KEY=your-dashscope-api-key

# OpenAI
export OPENAI_API_KEY=sk-your-openai-api-key

# Anthropic
export ANTHROPIC_API_KEY=sk-ant-your-anthropic-key
```

### 2.4 Hello AgentScope!

```python
"""
第一个 AgentScope 应用 - 简单对话
"""

from agentscope.agent import ReActAgent, UserAgent
from agentscope.model import DashScopeChatModel
from agentscope.formatter import DashScopeChatFormatter
from agentscope.memory import InMemoryMemory
import os
import asyncio

async def main():
    # 创建模型
    model = DashScopeChatModel(
        model_name="qwen-max",
        api_key=os.environ["DASHSCOPE_API_KEY"],
        stream=True,  # 流式输出
    )
    
    # 创建智能体
    agent = ReActAgent(
        name="Friday",
        sys_prompt="你是一个名为 Friday 的友好助手。",
        model=model,
        memory=InMemoryMemory(),
        formatter=DashScopeChatFormatter(),
    )
    
    # 创建用户代理
    user = UserAgent(name="用户")
    
    # 开始对话
    msg = None
    print("🤖 Friday: 你好！我是 Friday，有什么可以帮助你的？(输入 'exit' 退出)")
    
    while True:
        msg = await agent(msg)
        msg = await user(msg)
        
        if msg.get_text_content() == "exit":
            print("🤖 Friday: 再见！祝你有美好的一天！")
            break

if __name__ == "__main__":
    asyncio.run(main())
```

**运行效果：**
```
🤖 Friday: 你好！我是 Friday，有什么可以帮助你的？(输入 'exit' 退出)
👤 用户：今天天气怎么样？
🤖 Friday: 抱歉，我无法获取实时天气信息。不过你可以查看当地天气预报...
👤 用户：exit
🤖 Friday: 再见！祝你有美好的一天！
```

---

## 三、核心组件详解

### 3.1 Agent（智能体）

AgentScope 提供多种内置 Agent 类型：

#### ReActAgent（推理 + 行动）

```python
"""
ReActAgent - 支持工具调用的智能体
"""

from agentscope.agent import ReActAgent
from agentscope.model import DashScopeChatModel
from agentscope.memory import InMemoryMemory
from agentscope.tool import Toolkit, execute_python_code, execute_shell_command
import os

# 创建工具包
toolkit = Toolkit()
toolkit.register_tool_function(execute_python_code)
toolkit.register_tool_function(execute_shell_command)

# 创建智能体
agent = ReActAgent(
    name="CodeAssistant",
    sys_prompt="""你是一个编程助手，名为 CodeAssistant。
你可以执行 Python 代码和 Shell 命令来帮助用户。
请确保代码安全可靠。""",
    model=DashScopeChatModel(
        model_name="qwen-max",
        api_key=os.environ["DASHSCOPE_API_KEY"],
        stream=True,
    ),
    memory=InMemoryMemory(),
    toolkit=toolkit,
    verbose=True,  # 显示思考过程
)

# 使用示例
import asyncio

async def main():
    from agentscope.message import Msg
    
    # 用户请求
    msg = Msg("user", "帮我计算 1 到 100 的和", "user")
    
    # 智能体响应（会自动调用工具）
    response = await agent(msg)
    print(response.get_text_content())

asyncio.run(main())
```

#### UserAgent（用户代理）

```python
"""
UserAgent - 模拟用户输入
"""

from agentscope.agent import UserAgent
import asyncio

async def main():
    user = UserAgent(
        name="测试用户",
        require_url=False,  # 不需要 URL 输入
    )
    
    # 等待用户输入
    msg = await user()
    print(f"用户输入：{msg.get_text_content()}")

asyncio.run(main())
```

#### 自定义 Agent

```python
"""
自定义 Agent - 继承 AgentBase
"""

from agentscope.agent import AgentBase
from agentscope.message import Msg
from typing import Optional, Union, Sequence

class CustomAgent(AgentBase):
    """自定义智能体"""
    
    def __init__(
        self,
        name: str,
        sys_prompt: str,
        model_config: dict,
        **kwargs,
    ):
        super().__init__(
            name=name,
            sys_prompt=sys_prompt,
            model_config=model_config,
            **kwargs,
        )
    
    async def reply(self, x: Optional[Union[Msg, Sequence[Msg]]] = None) -> Msg:
        """自定义回复逻辑"""
        # 记录消息到记忆
        self.memory.add(x)
        
        # 调用模型生成响应
        response = self.model.generate(
            messages=self.memory.get_messages(),
            sys_prompt=self.sys_prompt,
        )
        
        # 创建响应消息
        msg = Msg(
            name=self.name,
            content=response.text,
            role="assistant",
        )
        
        # 记录响应到记忆
        self.memory.add(msg)
        
        return msg

# 使用示例
async def main():
    agent = CustomAgent(
        name="CustomBot",
        sys_prompt="你是一个自定义智能体。",
        model_config={
            "model_type": "dashscope_chat",
            "model_name": "qwen-max",
            "api_key": os.environ["DASHSCOPE_API_KEY"],
        },
    )
    
    msg = Msg("user", "你好", "user")
    response = await agent(msg)
    print(response.get_text_content())

asyncio.run(main())
```

### 3.2 Model（模型）

AgentScope 支持多种模型提供商：

#### DashScope（阿里云）

```python
from agentscope.model import DashScopeChatModel

model = DashScopeChatModel(
    model_name="qwen-max",      # 通义千问 Max
    api_key=os.environ["DASHSCOPE_API_KEY"],
    stream=True,                # 流式输出
    temperature=0.7,            # 温度参数
    max_tokens=2000,            # 最大输出长度
)
```

#### OpenAI

```python
from agentscope.model import OpenAIChatModel

model = OpenAIChatModel(
    model_name="gpt-4o",
    api_key=os.environ["OPENAI_API_KEY"],
    stream=True,
    temperature=0.7,
)
```

#### Anthropic

```python
from agentscope.model import AnthropicChatModel

model = AnthropicChatModel(
    model_name="claude-sonnet-4-20250514",
    api_key=os.environ["ANTHROPIC_API_KEY"],
    stream=True,
)
```

#### 本地模型（Ollama）

```python
from agentscope.model import OllamaChatModel

model = OllamaChatModel(
    model_name="llama3.1:8b",
    host="http://localhost:11434",
    stream=True,
)
```

### 3.3 Memory（记忆）

#### InMemoryMemory（内存记忆）

```python
from agentscope.memory import InMemoryMemory

memory = InMemoryMemory(
    max_size=100,  # 最大消息数
)

# 添加消息
memory.add(Msg("user", "你好", "user"))
memory.add(Msg("assistant", "你好！有什么可以帮助你的？", "assistant"))

# 获取消息
messages = memory.get_messages()
for msg in messages:
    print(f"{msg.name}: {msg.content}")

# 清空记忆
memory.clear()
```

#### 长期记忆（ReMe 集成）

```python
from agentscope.memory import LongTermMemory

memory = LongTermMemory(
    db_path="./memory.db",  # 数据库路径
    compression=True,        # 启用压缩
)

# 添加长期记忆
memory.add_long_term("用户喜欢 Python 编程")
memory.add_long_term("项目截止日期是 2026-04-01")

# 检索相关记忆
relevant = memory.retrieve("编程相关", top_k=3)
print(relevant)
```

### 3.4 Tool（工具）

#### 内置工具

```python
from agentscope.tool import (
    Toolkit,
    execute_python_code,
    execute_shell_command,
    read_file,
    write_file,
    search_web,
)

# 创建工具包
toolkit = Toolkit()

# 注册内置工具
toolkit.register_tool_function(execute_python_code)
toolkit.register_tool_function(execute_shell_command)
toolkit.register_tool_function(read_file)
toolkit.register_tool_function(write_file)
toolkit.register_tool_function(search_web)
```

#### 自定义工具

```python
"""
自定义工具 - 天气查询
"""

from agentscope.tool import Toolkit
import requests
import os

def get_weather(city: str) -> str:
    """
    获取城市天气信息。
    
    Args:
        city: 城市名称
    
    Returns:
        天气信息字符串
    """
    api_key = os.environ.get("WEATHER_API_KEY")
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
    
    try:
        response = requests.get(url)
        data = response.json()
        
        if data.get("weather"):
            temp = data["main"]["temp"] - 273.15
            desc = data["weather"][0]["description"]
            return f"{city}的天气：{desc}，温度：{temp:.1f}°C"
        return "无法获取天气信息"
    except Exception as e:
        return f"天气查询失败：{str(e)}"

# 注册工具
toolkit = Toolkit()
toolkit.register_tool_function(get_weather)

# 智能体使用
agent = ReActAgent(
    name="WeatherBot",
    sys_prompt="你是一个天气助手。",
    model=model,
    toolkit=toolkit,
)
```

#### MCP 工具集成

```python
"""
MCP 工具集成 - 高德地图
"""

from agentscope.mcp import HttpStatelessClient
from agentscope.tool import Toolkit
import os
import asyncio

async def mcp_tool_example():
    # 创建 MCP 客户端
    client = HttpStatelessClient(
        name="gaode_mcp",
        transport="streamable_http",
        url=f"https://mcp.amap.com/mcp?key={os.environ['GAODE_API_KEY']}",
    )
    
    # 获取 MCP 工具
    func = await client.get_callable_function(func_name="maps_geo")
    
    # 方式 1: 直接调用
    result = await func(address="天安门", city="北京")
    print(result)
    
    # 方式 2: 注册为智能体工具
    toolkit = Toolkit()
    toolkit.register_tool_function(func)
    
    agent = ReActAgent(
        name="MapAssistant",
        sys_prompt="你是一个地图助手。",
        model=model,
        toolkit=toolkit,
    )

asyncio.run(mcp_tool_example())
```

---

## 四、多 Agent 协作

### 4.1 MsgHub 消息中心

MsgHub 是 AgentScope 的核心编排组件，用于管理多 Agent 对话。

```python
"""
MsgHub - 多 Agent 对话管理
"""

from agentscope.pipeline import MsgHub, sequential_pipeline
from agentscope.message import Msg
from agentscope.agent import ReActAgent
import asyncio

async def multi_agent_conversation():
    # 创建多个智能体
    researcher = ReActAgent(
        name="研究员",
        sys_prompt="你是一位资深研究员，负责收集和分析信息。",
        model=model,
    )
    
    writer = ReActAgent(
        name="作家",
        sys_prompt="你是一位技术作家，负责撰写文章。",
        model=model,
    )
    
    editor = ReActAgent(
        name="编辑",
        sys_prompt="你是一位资深编辑，负责审核和优化文章。",
        model=model,
    )
    
    # 创建消息中心
    async with MsgHub(
        participants=[researcher, writer, editor],
        announcement=Msg("主持人", "请开始讨论 AI 发展趋势。", "assistant"),
    ) as hub:
        # 顺序执行：研究员 → 作家 → 编辑
        await sequential_pipeline([researcher, writer, editor])
        
        # 动态添加参与者
        reviewer = ReActAgent(
            name="审核员",
            sys_prompt="你负责最终审核。",
            model=model,
        )
        hub.add(reviewer)
        
        # 广播消息
        await hub.broadcast(Msg("主持人", "讨论结束，请审核。", "assistant"))
        
        # 移除参与者
        hub.delete(researcher)

asyncio.run(multi_agent_conversation())
```

### 4.2 顺序管道

```python
"""
顺序管道 - 流水线式处理
"""

from agentscope.pipeline import sequential_pipeline
from agentscope.message import Msg

async def sequential_example():
    # 创建智能体链
    agent1 = ReActAgent(name="分析员", sys_prompt="分析数据", model=model)
    agent2 = ReActAgent(name="报告员", sys_prompt="生成报告", model=model)
    agent3 = ReActAgent(name="审核员", sys_prompt="审核报告", model=model)
    
    # 初始消息
    initial_msg = Msg("user", "请分析销售数据并生成报告", "user")
    
    # 顺序执行
    result = await sequential_pipeline(
        [agent1, agent2, agent3],
        initial_msg,
    )
    
    print(result.get_text_content())

asyncio.run(sequential_example())
```

### 4.3 并发对话

```python
"""
并发对话 - 多个智能体同时响应
"""

from agentscope.pipeline import concurrent_pipeline
from agentscope.message import Msg
import asyncio

async def concurrent_example():
    # 创建多个智能体
    agent1 = ReActAgent(name="专家 A", sys_prompt="从技术角度分析", model=model)
    agent2 = ReActAgent(name="专家 B", sys_prompt="从商业角度分析", model=model)
    agent3 = ReActAgent(name="专家 C", sys_prompt="从用户角度分析", model=model)
    
    # 初始消息
    question = Msg("user", "如何评价这款新产品？", "user")
    
    # 并发执行
    results = await concurrent_pipeline(
        [agent1, agent2, agent3],
        question,
    )
    
    # 收集所有响应
    for result in results:
        print(f"{result.name}: {result.get_text_content()}")

asyncio.run(concurrent_example())
```

### 4.4 辩论模式

```python
"""
辩论模式 - 正反方辩论
"""

from agentscope.pipeline import debate_pipeline
from agentscope.message import Msg

async def debate_example():
    # 创建正反方
    pro_agent = ReActAgent(
        name="正方",
        sys_prompt="你支持这个观点，请提供论据。",
        model=model,
    )
    
    con_agent = ReActAgent(
        name="反方",
        sys_prompt="你反对这个观点，请提供论据。",
        model=model,
    )
    
    # 辩论主题
    topic = Msg("主持人", "AI 是否会取代人类工作？", "assistant")
    
    # 开始辩论（3 轮）
    result = await debate_pipeline(
        [pro_agent, con_agent],
        topic,
        rounds=3,
    )
    
    # 总结
    print("辩论结束")
    for msg in result:
        print(f"{msg.name}: {msg.get_text_content()}")

asyncio.run(debate_example())
```

---

## 五、高级特性

### 5.1 实时语音智能体

```python
"""
实时语音智能体 - 语音交互
"""

from agentscope.agent import RealtimeVoiceAgent
from agentscope.model import DashScopeChatModel
import asyncio

async def voice_agent_example():
    # 创建语音智能体
    agent = RealtimeVoiceAgent(
        name="VoiceAssistant",
        sys_prompt="你是一个语音助手。",
        model=DashScopeChatModel(
            model_name="qwen-max",
            api_key=os.environ["DASHSCOPE_API_KEY"],
        ),
        # 语音配置
        tts_config={
            "voice": "female",  # 女声
            "speed": 1.0,       # 语速
        },
        asr_config={
            "language": "zh-CN",  # 中文
        },
    )
    
    # 启动 Web 界面
    await agent.run_webui(
        host="0.0.0.0",
        port=8080,
    )

asyncio.run(voice_agent_example())
```

### 5.2 Agentic RL（强化学习训练）

```python
"""
Agentic RL - 使用强化学习训练智能体
"""

from agentscope.rl import AgentTrainer
from agentscope.agent import ReActAgent
from trinity_rft import RLTrainer

async def rl_training_example():
    # 创建基础智能体
    agent = ReActAgent(
        name="MathAgent",
        sys_prompt="你是一个数学推理助手。",
        model=model,
    )
    
    # 创建训练器
    trainer = AgentTrainer(
        agent=agent,
        reward_function="accuracy",  # 准确率作为奖励
        learning_rate=0.001,
    )
    
    # 训练数据
    training_data = [
        {"question": "1+1=?", "answer": "2"},
        {"question": "2*3=?", "answer": "6"},
        # ... 更多数据
    ]
    
    # 开始训练
    await trainer.train(
        training_data,
        epochs=10,
        batch_size=32,
    )
    
    # 测试训练后的智能体
    result = await agent(Msg("user", "5+7=?", "user"))
    print(result.get_text_content())

asyncio.run(rl_training_example())
```

### 5.3 可观测性（AgentScope Studio）

```python
"""
可观测性 - 集成 AgentScope Studio
"""

from agentscope import init

# 初始化可观测性
init(
    project_name="my_agent_app",
    studio_url="http://localhost:8080",  # AgentScope Studio 地址
    trace_mode="realtime",  # 实时追踪
)

# 所有智能体调用会自动上报到 Studio
agent = ReActAgent(
    name="TrackedAgent",
    sys_prompt="被追踪的智能体。",
    model=model,
)

# 在 Studio 中可以查看：
# - 智能体对话历史
# - 工具调用详情
# - 性能指标
# - 错误日志
```

### 5.4 A2A 协议（跨框架互操作）

```python
"""
A2A 协议 - 与其他框架的智能体互操作
"""

from agentscope.a2a import A2AClient

async def a2a_example():
    # 创建 A2A 客户端
    client = A2AClient(
        name="MyAgent",
        registry_url="http://nacos-server:8848",  # Nacos 注册中心
    )
    
    # 发现其他框架的智能体
    available_agents = await client.discover_agents()
    print(f"可用智能体：{available_agents}")
    
    # 调用其他框架的智能体
    result = await client.call_agent(
        agent_name="LangChainAgent",
        message="你好",
    )
    print(result)

asyncio.run(a2a_example())
```

---

## 六、实战项目

### 6.1 智能客服系统

```python
"""
智能客服系统 - 多 Agent 协作
"""

from agentscope.agent import ReActAgent
from agentscope.pipeline import MsgHub, sequential_pipeline
from agentscope.message import Msg
from agentscope.tool import Toolkit
import asyncio
import os

class CustomerServiceSystem:
    """智能客服系统"""
    
    def __init__(self):
        # 创建工具包
        toolkit = Toolkit()
        toolkit.register_tool_function(self.query_order)
        toolkit.register_tool_function(self.process_refund)
        
        # 创建智能体
        self.greeter = ReActAgent(
            name="接待员",
            sys_prompt="你是客服接待员，负责问候和初步分类。",
            model=self._create_model(),
        )
        
        self.order_agent = ReActAgent(
            name="订单助手",
            sys_prompt="你负责处理订单查询。",
            model=self._create_model(),
            toolkit=toolkit,
        )
        
        self.refund_agent = ReActAgent(
            name="退款助手",
            sys_prompt="你负责处理退款请求。",
            model=self._create_model(),
            toolkit=toolkit,
        )
        
        self.human_agent = ReActAgent(
            name="人工客服",
            sys_prompt="你是人工客服，处理复杂问题。",
            model=self._create_model(),
        )
    
    def _create_model(self):
        return DashScopeChatModel(
            model_name="qwen-max",
            api_key=os.environ["DASHSCOPE_API_KEY"],
        )
    
    async def query_order(self, order_id: str) -> str:
        """查询订单"""
        # 模拟订单查询
        return f"订单 {order_id} 状态：已发货"
    
    async def process_refund(self, order_id: str) -> str:
        """处理退款"""
        # 模拟退款处理
        return f"订单 {order_id} 退款已受理"
    
    async def route_request(self, user_msg: str):
        """路由用户请求"""
        # 接待员初步分类
        classification = await self.greeter(
            Msg("user", user_msg, "user")
        )
        
        # 根据分类路由
        if "订单" in classification.get_text_content():
            result = await self.order_agent(classification)
        elif "退款" in classification.get_text_content():
            result = await self.refund_agent(classification)
        else:
            result = await self.human_agent(classification)
        
        return result
    
    async def run(self):
        """运行客服系统"""
        print("🤖 智能客服系统已启动")
        
        while True:
            user_input = input("👤 您：")
            if user_input == "exit":
                break
            
            response = await self.route_request(user_input)
            print(f"🤖 客服：{response.get_text_content()}")

# 使用示例
async def main():
    system = CustomerServiceSystem()
    await system.run()

asyncio.run(main())
```

### 6.2 数据分析团队

```python
"""
数据分析团队 - 多 Agent 协作分析数据
"""

from agentscope.agent import ReActAgent
from agentscope.pipeline import sequential_pipeline, MsgHub
from agentscope.message import Msg
from agentscope.tool import Toolkit, execute_python_code
import asyncio
import os

async def data_analysis_team():
    # 创建工具包
    toolkit = Toolkit()
    toolkit.register_tool_function(execute_python_code)
    
    # 数据分析师
    analyst = ReActAgent(
        name="数据分析师",
        sys_prompt="""你是资深数据分析师。
你可以执行 Python 代码来分析数据。
请使用 pandas、numpy 等库。""",
        model=DashScopeChatModel(
            model_name="qwen-max",
            api_key=os.environ["DASHSCOPE_API_KEY"],
        ),
        toolkit=toolkit,
    )
    
    # 可视化专家
    visualizer = ReActAgent(
        name="可视化专家",
        sys_prompt="""你是数据可视化专家。
你负责创建图表和可视化报告。
请使用 matplotlib、seaborn 等库。""",
        model=DashScopeChatModel(
            model_name="qwen-max",
            api_key=os.environ["DASHSCOPE_API_KEY"],
        ),
        toolkit=toolkit,
    )
    
    # 报告撰写员
    reporter = ReActAgent(
        name="报告撰写员",
        sys_prompt="""你是报告撰写员。
你负责根据分析结果撰写专业报告。
报告应包含关键发现和建议。""",
        model=DashScopeChatModel(
            model_name="qwen-max",
            api_key=os.environ["DASHSCOPE_API_KEY"],
        ),
    )
    
    # 创建消息中心
    async with MsgHub(
        participants=[analyst, visualizer, reporter],
        announcement=Msg(
            "项目经理",
            "请分析销售数据并生成报告。数据文件：sales_data.csv",
            "assistant",
        ),
    ) as hub:
        # 顺序执行分析流程
        await sequential_pipeline([analyst, visualizer, reporter])
        
        # 获取最终报告
        final_report = await reporter(
            Msg("system", "请总结分析结果。", "system")
        )
        
        print("=== 最终报告 ===")
        print(final_report.get_text_content())

asyncio.run(data_analysis_team())
```

---

## 七、部署和监控

### 7.1 Docker 部署

```dockerfile
# Dockerfile
FROM python:3.10-slim

WORKDIR /app

# 安装依赖
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 复制代码
COPY . .

# 环境变量
ENV DASHSCOPE_API_KEY=your-api-key

# 运行
CMD ["python", "main.py"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  agent:
    build: .
    ports:
      - "8080:8080"
    environment:
      - DASHSCOPE_API_KEY=${DASHSCOPE_API_KEY}
    volumes:
      - ./data:/app/data
    restart: unless-stopped
  
  studio:
    image: agentscope/studio:latest
    ports:
      - "8081:8081"
    depends_on:
      - agent
    restart: unless-stopped
```

### 7.2 AgentScope Studio

```bash
# 安装 Studio
pip install agentscope-studio

# 启动 Studio
agentscope-studio start \
  --host 0.0.0.0 \
  --port 8081 \
  --db-path ./studio.db

# 访问 http://localhost:8081 查看可视化界面
```

**Studio 功能：**
- 📊 对话历史可视化
- 🔍 智能体行为追踪
- 📈 性能指标监控
- 🐛 错误日志分析
- 🔔 实时告警

---

## 八、最佳实践

### 8.1 智能体设计原则

```python
# ✅ 推荐：单一职责
data_agent = ReActAgent(
    name="数据分析师",
    sys_prompt="你负责数据分析。",
    model=model,
)

visualization_agent = ReActAgent(
    name="可视化专家",
    sys_prompt="你负责数据可视化。",
    model=model,
)

# ❌ 避免：职责过多
bad_agent = ReActAgent(
    name="全能助手",
    sys_prompt="你负责分析、可视化、报告、审核...",  # 太宽泛
    model=model,
)
```

### 8.2 提示词优化

```python
# ✅ 推荐：具体明确的系统提示
good_prompt = """你是客服订单助手。
你的职责：
1. 查询订单状态
2. 处理订单修改
3. 解答订单相关问题

如果无法处理，请转接人工客服。"""

# ❌ 避免：模糊的提示
bad_prompt = "你是一个助手。"
```

### 8.3 记忆管理

```python
# ✅ 推荐：设置合理的记忆大小
memory = InMemoryMemory(
    max_size=50,  # 保留最近 50 条消息
)

# 定期清理无用记忆
if len(memory.get_messages()) > 100:
    memory.clear()

# ❌ 避免：无限制增长
bad_memory = InMemoryMemory()  # 没有大小限制
```

### 8.4 错误处理

```python
# ✅ 推荐：完善的错误处理
async def safe_agent_call(agent, msg):
    try:
        response = await agent(msg)
        return response
    except Exception as e:
        # 记录错误
        logger.error(f"智能体调用失败：{e}")
        # 返回友好错误
        return Msg("system", "抱歉，处理失败，请稍后重试。", "assistant")

# ❌ 避免：无错误处理
async def unsafe_agent_call(agent, msg):
    return await agent(msg)  # 可能抛出异常
```

---

## 九、总结

### 9.1 AgentScope 优势

| 优势 | 说明 |
|------|------|
| **开发者友好** | 5 分钟上手，简洁的 API |
| **功能全面** | 语音、工具、记忆、RL 训练等 |
| **生产就绪** | K8s 部署、可观测性、A2A 协议 |
| **中文支持** | 阿里生态，中文文档完善 |
| **生态丰富** | MCP、ModelScope 集成 |

### 9.2 适用场景

- ✅ **多 Agent 协作**：团队协作、辩论、游戏
- ✅ **语音交互**：客服、助手、教育
- ✅ **企业应用**：数据分析、代码审查、自动化
- ✅ **研究实验**：Agentic RL、新交互模式
- ✅ **教育模拟**：多 Agent 环境训练

### 9.3 学习资源

- **官方文档**：https://doc.agentscope.io/
- **GitHub**：https://github.com/agentscope-ai/agentscope
- **Discord**：https://discord.gg/agentscope
- **Tutorial**：https://doc.agentscope.io/tutorial
- **示例库**：https://github.com/agentscope-ai/agentscope/tree/main/examples

### 9.4 下一步

1. **从简单开始**：先实现单 Agent 对话
2. **添加工具**：让 Agent 能够执行操作
3. **多 Agent 协作**：尝试 MsgHub 编排
4. **部署上线**：使用 Docker/K8s 部署
5. **监控优化**：集成 Studio 可观测性

AgentScope 作为阿里巴巴开源的多 Agent 框架，提供了从开发到部署的完整解决方案。无论你是想快速原型验证，还是构建生产级应用，AgentScope 都能满足你的需求。开始构建你的多 Agent 系统吧！🚀
