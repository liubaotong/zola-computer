+++
title = "LangChain 完全指南 2026：架构、生态与实战代码详解"
date = "2026-03-18T12:00:00+08:00"

[taxonomies]
tags = ["LangChain", "AI", "大语言模型", "RAG", "AI Agent", "Python"]
categories = ["人工智能", "编程"]

[extra]
summary = "全面介绍 LangChain 2026 最新版本的架构设计、核心组件、生态系统，包含完整的代码示例，涵盖 Chains、Agents、RAG、LangGraph 等核心技术，帮助你快速构建生产级 AI 应用。"
author = "博主"
+++

在 AI 应用开发领域，**LangChain** 已经成为构建大语言模型（LLM）应用的事实标准框架。从简单的聊天机器人到复杂的自主 AI Agent，LangChain 提供了一套完整的工具和抽象，让开发者能够轻松连接 LLM 与外部数据源、工具和业务逻辑。

本文将深入介绍 LangChain 2026 最新版本的架构设计、核心组件、生态系统，并通过完整的代码示例，帮助你掌握构建生产级 AI 应用的关键技术。

---

## 一、什么是 LangChain？

### LangChain 简介

**LangChain** 是一个开源框架，用于简化基于大语言模型的应用开发。它于 2022 年发布，迅速成为 AI 开发领域最流行的框架之一。

**核心价值：**
- **模块化设计**：将 LLM 应用拆分为可复用的组件
- **统一接口**：为不同 LLM 提供商提供一致的 API
- **生态丰富**：100+ 集成，支持各种数据源、工具和模型
- **生产就绪**：提供可观察性、错误处理、流式响应等生产功能

### LangChain 能做什么？

| 应用场景 | 描述 | 复杂度 |
|---------|------|--------|
| **智能问答** | 基于私有数据的问答系统 | ⭐⭐ |
| **文档分析** | 自动摘要、提取关键信息 | ⭐⭐ |
| **代码助手** | 代码生成、解释、调试 | ⭐⭐ |
| **数据分析** | 自然语言查询数据库 | ⭐⭐⭐ |
| **AI Agent** | 自主执行复杂任务 | ⭐⭐⭐⭐ |
| **多 Agent 协作** | 多个 Agent 协同工作 | ⭐⭐⭐⭐⭐ |

### LangChain 版本演进

```
2022.10 - LangChain 首次发布
2023.03 - LangChain 0.1，基础组件完善
2023.10 - LangChain 0.2，架构重构
2024.06 - LangChain 0.3，引入 LangGraph
2025.01 - LangChain 1.0，生产级特性完善
2026.01 - LangChain 1.x，当前最新版本
```

> 💡 **注意**：LangChain 在 2025 年进行了重大架构调整，推荐使用 LangGraph 构建复杂的 Agent 应用。

---

## 二、LangChain 核心架构

### 2.1 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                      LangChain 应用层                        │
├─────────────────────────────────────────────────────────────┤
│  Chains  │  Agents  │  RAG  │  Graphs  │  Memory  │  Tools  │
├─────────────────────────────────────────────────────────────┤
│                    LangChain Expression Language (LCEL)      │
├─────────────────────────────────────────────────────────────┤
│  LLMs  │  Chat Models  │  Embeddings  │  Output Parsers    │
├─────────────────────────────────────────────────────────────┤
│  数据连接器  │  向量存储  │  工具集成  │  回调系统           │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 核心组件详解

#### 1. Models（模型层）

LangChain 支持多种模型类型：

```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_ollama import ChatOllama

# OpenAI GPT-4
llm = ChatOpenAI(
    model="gpt-4o",
    temperature=0.7,
    max_tokens=2000,
    api_key="your-api-key"
)

# Anthropic Claude
llm = ChatAnthropic(
    model="claude-sonnet-4-20250514",
    temperature=0.7,
    max_tokens=2000,
    api_key="your-api-key"
)

# Google Gemini
llm = ChatGoogleGenerativeAI(
    model="gemini-2.0-flash",
    temperature=0.7,
    api_key="your-api-key"
)

# Ollama（本地运行）
llm = ChatOllama(
    model="llama3.1:8b",
    temperature=0.7,
    base_url="http://localhost:11434"
)
```

#### 2. Prompts（提示词）

提示词模板让输入结构化、可复用：

```python
from langchain_core.prompts import ChatPromptTemplate, PromptTemplate

# 简单提示模板
prompt = PromptTemplate.from_template(
    "请用简洁的语言总结以下内容：\n\n{content}"
)

# 聊天提示模板（支持多轮对话）
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个专业的{role}助手，擅长{skill}。"),
    ("human", "{input}"),
    ("ai", "{response}"),
    ("human", "{followup}")
])

# 带部分变量的模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个翻译助手，将{text}翻译成{target_language}。"),
    ("human", "{input}")
]).partial(target_language="中文")
```

#### 3. Output Parsers（输出解析器）

将 LLM 的非结构化输出转换为结构化数据：

```python
from langchain_core.output_parsers import (
    StrOutputParser,
    JsonOutputParser,
    PydanticOutputParser
)
from pydantic import BaseModel, Field
from typing import List, Literal

# 字符串输出（最简单）
parser = StrOutputParser()

# JSON 输出
parser = JsonOutputParser()

# Pydantic 结构化输出（推荐）
class ArticleSummary(BaseModel):
    title: str = Field(description="文章标题")
    summary: str = Field(description="文章摘要，100 字以内")
    key_points: List[str] = Field(description="关键点列表")
    sentiment: Literal["positive", "neutral", "negative"] = Field(
        description="文章情感倾向"
    )

parser = PydanticOutputParser(pydantic_object=ArticleSummary)

# 使用结构化输出
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o")
structured_llm = llm.with_structured_output(ArticleSummary)

result = structured_llm.invoke("请总结这篇关于 AI 发展的文章...")
print(result.title)
print(result.sentiment)
```

---

## 三、LangChain Expression Language (LCEL)

LCEL 是 LangChain 的核心，提供了一种声明式的方式构建 AI 工作流。

### 3.1 基础语法

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

# 创建组件
prompt = ChatPromptTemplate.from_template("讲一个关于{topic}的笑话")
model = ChatOpenAI(model="gpt-4o")
parser = StrOutputParser()

# 使用 | 操作符连接组件（类似 Unix 管道）
chain = prompt | model | parser

# 执行链
result = chain.invoke({"topic": "程序员"})
print(result)
```

### 3.2 并行执行

```python
from langchain_core.runnables import RunnableParallel

# 并行执行多个链
parallel = RunnableParallel(
    summary=prompt1 | model | parser,
    translation=prompt2 | model | parser,
    keywords=prompt3 | model | parser
)

result = parallel.invoke({"topic": "AI 发展"})
# result = {
#     "summary": "...",
#     "translation": "...",
#     "keywords": ["AI", "发展", "..."]
# }
```

### 3.3 条件分支

```python
from langchain_core.runnables import RunnableBranch

# 根据条件执行不同的链
branch = RunnableBranch(
    (lambda x: "紧急" in x["query"], urgent_chain),
    (lambda x: "投诉" in x["query"], complaint_chain),
    default_chain  # 默认链
)

result = branch.invoke({"query": "这是一个紧急问题"})
```

### 3.4 流式响应

```python
# 流式输出（适合聊天应用）
chain = prompt | model | StrOutputParser()

for chunk in chain.stream({"topic": "人工智能"}):
    print(chunk, end="", flush=True)

# 异步流式
async for chunk in chain.astream({"topic": "人工智能"}):
    print(chunk, end="", flush=True)
```

### 3.5 批量处理

```python
# 批量处理多个输入
results = chain.batch([
    {"topic": "AI"},
    {"topic": "区块链"},
    {"topic": "量子计算"}
])

# 异步批量
results = await chain.abatch([
    {"topic": "AI"},
    {"topic": "区块链"}
])
```

---

## 四、Chains（链）

Chain 是 LangChain 中最基本的工作流单元，表示一系列按顺序执行的操作。

### 4.1 简单链示例

```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 定义 LLM
llm = ChatOpenAI(model="gpt-4o", temperature=0.7)

# 定义提示模板
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个专业的技术写作助手。"),
    ("human", "请为以下主题写一篇简短的文章：{topic}")
])

# 创建链
chain = prompt | llm | StrOutputParser()

# 执行
result = chain.invoke({"topic": "生成式 AI 的发展趋势"})
print(result)
```

### 4.2 带记忆的对话链

```python
from langchain.memory import ConversationBufferMemory
from langchain_core.runnables.history import RunnableWithMessageHistory
from langchain_core.chat_history import BaseChatMessageHistory, InMemoryChatMessageHistory

# 创建记忆存储
store = {}

def get_session_history(session_id: str) -> BaseChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

# 创建带记忆的链
prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个友好的 AI 助手，记住之前的对话内容。"),
    ("placeholder", "{chat_history}"),
    ("human", "{input}")
])

chain = prompt | llm | StrOutputParser()

# 包装为带历史记录的链
conversational_chain = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history"
)

# 使用
config = {"configurable": {"session_id": "user123"}}

response1 = conversational_chain.invoke(
    {"input": "我叫小明"},
    config=config
)

response2 = conversational_chain.invoke(
    {"input": "我叫什么名字？"},
    config=config
)
# AI 会记得你叫小明
```

---

## 五、RAG（检索增强生成）

RAG 是 LangChain 最重要的应用场景之一，让 LLM 能够访问私有数据。

### 5.1 RAG 架构

```
用户问题 → 检索器 → 相关文档 → 提示词 → LLM → 回答
              ↓
          向量数据库
```

### 5.2 完整 RAG 实现

```python
from langchain_community.document_loaders import (
    DirectoryLoader,
    PyPDFLoader,
    WebBaseLoader
)
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings, ChatOpenAI
from langchain_community.vectorstores import Chroma
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 1. 加载文档
loader = DirectoryLoader(
    "./documents/",
    glob="**/*.pdf",
    loader_cls=PyPDFLoader
)
documents = loader.load()

# 2. 分割文档
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=800,
    chunk_overlap=100,
    length_function=len
)
chunks = text_splitter.split_documents(documents)

# 3. 创建向量存储
embeddings = OpenAIEmbeddings(model="text-embedding-3-large")
vectorstore = Chroma.from_documents(
    documents=chunks,
    embedding=embeddings,
    persist_directory="./chroma_db"
)

# 4. 创建检索器
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}  # 返回最相关的 4 个文档
)

# 5. 创建 RAG 链
template = """基于以下上下文信息回答问题。如果上下文中没有答案，请说"我不知道"。

上下文：
{context}

问题：{question}

回答："""

prompt = ChatPromptTemplate.from_template(template)

model = ChatOpenAI(model="gpt-4o", temperature=0)

rag_chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)

# 6. 使用 RAG
answer = rag_chain.invoke("公司的年假政策是什么？")
print(answer)
```

### 5.3 高级 RAG 技巧

#### 查询重写（Query Rewriting）

```python
# 在检索前优化查询
query_rewrite_prompt = ChatPromptTemplate.from_messages([
    ("system", "你是一个查询优化专家。将用户的查询改写为更适合向量检索的形式。"),
    ("human", "原始查询：{query}")
])

query_rewriter = query_rewrite_prompt | llm | StrOutputParser()

# 完整的 RAG 链（带查询重写）
rag_chain_with_rewrite = (
    {"query": RunnablePassthrough()}
    | {"rewritten_query": query_rewriter, "original_query": RunnablePassthrough()}
    | {"context": lambda x: retriever.invoke(x["rewritten_query"]), 
       "question": lambda x: x["original_query"]}
    | prompt
    | model
    | StrOutputParser()
)
```

#### 文档重排序（Re-ranking）

```python
from langchain.retrievers import ContextualCompressionRetriever
from langchain.retrievers.document_compressors import FlashrankRerank

# 使用 FlashRank 进行重排序
compressor = FlashrankRerank()
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=retriever
)

# 使用重排序后的检索器
rag_chain = (
    {"context": compression_retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
```

---

## 六、Tools（工具）

工具让 LLM 能够执行具体操作，如搜索、计算、API 调用等。

### 6.1 创建工具

```python
from langchain.tools import tool
from langchain_community.tools import DuckDuckGoSearchRun
import requests

# 使用装饰器创建工具
@tool
def search_web(query: str) -> str:
    """搜索互联网获取最新信息。
    
    Args:
        query: 搜索关键词
        
    Returns:
        搜索结果摘要
    """
    search = DuckDuckGoSearchRun()
    return search.run(query)

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
def get_weather(city: str) -> str:
    """获取城市天气信息。
    
    Args:
        city: 城市名称
        
    Returns:
        天气信息
    """
    api_key = "your-api-key"
    url = f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"
    response = requests.get(url)
    data = response.json()
    
    if data.get("weather"):
        temp = data["main"]["temp"] - 273.15  # 转换为摄氏度
        desc = data["weather"][0]["description"]
        return f"{city}的天气：{desc}，温度：{temp:.1f}°C"
    return "无法获取天气信息"

# 工具列表
tools = [search_web, calculate, get_weather]
```

### 6.2 使用工具

```python
from langchain.agents import create_tool_calling_agent, AgentExecutor

# 创建 Agent 提示
prompt = ChatPromptTemplate.from_messages([
    ("system", """你是一个智能助手，可以使用工具来帮助用户。
    
可用工具：
- search_web: 搜索互联网
- calculate: 计算数学表达式
- get_weather: 查询天气

请根据用户需求选择合适的工具。"""),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

# 创建 Agent
agent = create_tool_calling_agent(llm, tools, prompt)

# 创建执行器
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=10,
    handle_parsing_errors=True
)

# 使用 Agent
result = agent_executor.invoke({
    "input": "北京今天的天气怎么样？然后计算一下 25 乘以 35 等于多少"
})
print(result["output"])
```

---

## 七、AI Agents（智能体）

Agent 是 LangChain 最强大的功能，能够自主规划并执行复杂任务。

### 7.1 ReAct Agent（经典模式）

```python
from langchain.agents import create_react_agent, AgentExecutor
from langchain_core.prompts import PromptTemplate

# ReAct 提示模板
react_template = """Answer the following questions as best you can. You have access to the following tools:

{tools}

Use the following format:

Question: the input question you must answer
Thought: you should always think about what to do
Action: the action to take, should be one of [{tool_names}]
Action Input: the input to the action
Observation: the result of the action
... (this Thought/Action/Action Input/Observation can repeat N times)
Thought: I now know the final answer
Final Answer: the final answer to the original input question

Begin!

Question: {input}
Thought:{agent_scratchpad}"""

prompt = PromptTemplate.from_template(react_template)

# 创建 ReAct Agent
agent = create_react_agent(llm, tools, prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=10
)

# 执行复杂任务
result = agent_executor.invoke({
    "input": """研究一下 2025 年 AI 领域的重大突破，
    找出 3 个最重要的进展，并总结它们的影响。"""
})
print(result["output"])
```

### 7.2 LangGraph（推荐的生产级方案）

LangGraph 是 LangChain 2025 年推出的新一代 Agent 框架，基于图结构，支持更复杂的工作流。

```python
from langgraph.graph import StateGraph, END
from langgraph.checkpoint.memory import MemorySaver
from typing import TypedDict, Annotated, Sequence, List
from langchain_core.messages import BaseMessage, HumanMessage, AIMessage
import operator

# 定义状态
class AgentState(TypedDict):
    messages: Annotated[Sequence[BaseMessage], operator.add]
    research_topic: str
    sources: List[str]
    summary: str
    verified: bool

# 定义节点函数
def research_node(state: AgentState):
    """研究节点：搜索信息"""
    topic = state["research_topic"]
    # 使用搜索工具
    search_result = search_web.invoke(f"{topic} 2025 最新进展")
    return {
        "sources": [search_result],
        "messages": [AIMessage(content=f"已收集关于{topic}的信息")]
    }

def analyze_node(state: AgentState):
    """分析节点：总结信息"""
    sources = state["sources"]
    analyze_prompt = ChatPromptTemplate.from_messages([
        ("system", "你是分析专家，总结以下信息。"),
        ("human", "信息：{sources}")
    ])
    chain = analyze_prompt | llm | StrOutputParser()
    summary = chain.invoke({"sources": "\n".join(sources)})
    return {
        "summary": summary,
        "messages": [AIMessage(content="已完成分析")]
    }

def verify_node(state: AgentState):
    """验证节点：检查质量"""
    summary = state["summary"]
    # 简单验证逻辑
    verified = len(summary) > 100 and "错误" not in summary
    return {
        "verified": verified,
        "messages": [AIMessage(content=f"验证{'通过' if verified else '失败'}")]
    }

# 构建图
workflow = StateGraph(AgentState)

# 添加节点
workflow.add_node("research", research_node)
workflow.add_node("analyze", analyze_node)
workflow.add_node("verify", verify_node)

# 定义边
workflow.set_entry_point("research")
workflow.add_edge("research", "analyze")
workflow.add_edge("analyze", "verify")

# 条件边（验证失败则重新研究）
workflow.add_conditional_edges(
    "verify",
    lambda state: "end" if state["verified"] else "retry",
    {
        "end": END,
        "retry": "research"
    }
)

# 添加记忆保存
memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

# 执行
config = {"configurable": {"thread_id": "research_001"}}
result = app.invoke({
    "research_topic": "AI Agent 框架",
    "sources": [],
    "summary": "",
    "verified": False,
    "messages": [HumanMessage(content="请研究 AI Agent 框架的最新发展")]
}, config=config)

print(result["summary"])
```

### 7.3 多 Agent 协作

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, List

class MultiAgentState(TypedDict):
    task: str
    research_result: str
    code_result: str
    final_report: str

# 研究员 Agent
def researcher_agent(state: MultiAgentState):
    """负责信息收集"""
    task = state["task"]
    result = search_web.invoke(f"{task} 技术细节")
    return {"research_result": result}

# 工程师 Agent
def engineer_agent(state: MultiAgentState):
    """负责代码实现"""
    research = state["research_result"]
    code_prompt = ChatPromptTemplate.from_messages([
        ("system", "你是资深工程师，根据以下需求写代码。"),
        ("human", "需求：{research}")
    ])
    code = (code_prompt | llm | StrOutputParser()).invoke({"research": research})
    return {"code_result": code}

# 文档工程师 Agent
def writer_agent(state: MultiAgentState):
    """负责文档编写"""
    research = state["research_result"]
    code = state["code_result"]
    write_prompt = ChatPromptTemplate.from_messages([
        ("system", "你是技术文档专家，编写完整的技术文档。"),
        ("human", "研究内容：{research}\n代码实现：{code}")
    ])
    report = (write_prompt | llm | StrOutputParser()).invoke({
        "research": research,
        "code": code
    })
    return {"final_report": report}

# 构建多 Agent 工作流
workflow = StateGraph(MultiAgentState)
workflow.add_node("researcher", researcher_agent)
workflow.add_node("engineer", engineer_agent)
workflow.add_node("writer", writer_agent)

workflow.set_entry_point("researcher")
workflow.add_edge("researcher", "engineer")
workflow.add_edge("engineer", "writer")
workflow.add_edge("writer", END)

app = workflow.compile()

# 执行
result = app.invoke({
    "task": "用 Python 实现一个简单的 RAG 系统",
    "research_result": "",
    "code_result": "",
    "final_report": ""
})

print("=== 最终报告 ===")
print(result["final_report"])
```

---

## 八、Memory（记忆）

记忆让 AI 应用能够记住历史对话，提供更连贯的体验。

### 8.1 对话缓冲记忆

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# 添加对话
memory.save_context(
    {"input": "你好，我叫小明"},
    {"output": "你好小明，很高兴认识你！"}
)

memory.save_context(
    {"input": "我喜欢编程"},
    {"output": "编程是一项很棒的技能！"}
)

# 获取历史
history = memory.load_memory_variables({})
print(history["chat_history"])
```

### 8.2 摘要记忆

```python
from langchain.memory import ConversationSummaryMemory

summary_memory = ConversationSummaryMemory(
    llm=llm,
    memory_key="summary"
)

# 长对话会自动被总结
summary_memory.save_context(
    {"input": "今天天气不错"},
    {"output": "是的，适合户外活动"}
)

summary = summary_memory.load_memory_variables({})
print(summary["summary"])
```

### 8.3 向量存储记忆

```python
from langchain.memory import VectorStoreRetrieverMemory
from langchain_community.vectorstores import Chroma

# 创建向量存储记忆
embeddings = OpenAIEmbeddings()
vectorstore = Chroma(embedding_function=embeddings)
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

memory = VectorStoreRetrieverMemory(retriever=retriever)

# 保存记忆
memory.save_context(
    {"input": "我喜欢 Python 和 Rust"},
    {"output": "这两门语言各有优势"}
)

# 检索相关记忆
relevant = memory.load_memory_variables({"input": "编程语言"})
print(relevant)
```

---

## 九、生产级最佳实践

### 9.1 错误处理

```python
from functools import wraps
import time

def retry(max_attempts=3, delay=2):
    """重试装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    time.sleep(delay * (2 ** attempt))
        return wrapper
    return decorator

@retry(max_attempts=3)
def invoke_llm_safely(chain, input_data):
    """安全调用 LLM"""
    return chain.invoke(input_data)
```

### 9.2 速率限制

```python
from collections import deque
import threading

class RateLimiter:
    """API 速率限制器"""
    
    def __init__(self, max_calls=10, time_window=60):
        self.max_calls = max_calls
        self.time_window = time_window
        self.calls = deque()
        self.lock = threading.Lock()
    
    def wait_if_needed(self):
        """等待直到可以发起请求"""
        with self.lock:
            while True:
                now = time.time()
                # 移除过期调用
                while self.calls and self.calls[0] < now - self.time_window:
                    self.calls.popleft()
                
                if len(self.calls) < self.max_calls:
                    self.calls.append(now)
                    return
                time.sleep(1)

# 使用
limiter = RateLimiter(max_calls=10, time_window=60)

def execute_with_rate_limit(chain, input_data):
    limiter.wait_if_needed()
    return chain.invoke(input_data)
```

### 9.3 日志和监控

```python
import logging
from langchain.callbacks import StdOutCallbackHandler

# 配置日志
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

# 使用回调
handler = StdOutCallbackHandler()
chain = prompt | llm | StrOutputParser()
result = chain.invoke(
    {"topic": "AI"},
    config={"callbacks": [handler]}
)
```

### 9.4 配置管理

```python
# .env 文件
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
DATABASE_URL=postgresql://...
CHROMA_PERSIST_DIR=./chroma_db

# config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    openai_api_key: str
    anthropic_api_key: str
    database_url: str
    chroma_persist_dir: str = "./chroma_db"
    llm_model: str = "gpt-4o"
    temperature: float = 0.7
    
    class Config:
        env_file = ".env"

settings = Settings()
```

---

## 十、LangChain 生态系统

### 10.1 核心包

| 包名 | 用途 |
|------|------|
| `langchain-core` | 核心抽象和接口 |
| `langchain` | 主框架 |
| `langchain-community` | 社区贡献的集成 |
| `langchain-openai` | OpenAI 集成 |
| `langchain-anthropic` | Anthropic 集成 |
| `langchain-google-genai` | Google Gemini 集成 |
| `langchain-ollama` | Ollama 本地模型集成 |
| `langgraph` | Agent 图工作流 |

### 10.2 相关工具

| 工具 | 描述 |
|------|------|
| **LangSmith** | 可观察性和调试平台 |
| **LangServe** | 部署 LangChain 应用为 REST API |
| **LangGraph Studio** | LangGraph 可视化开发工具 |
| **LlamaIndex** | 专注 RAG 的替代框架 |

### 10.3 向量数据库集成

```python
# Chroma（推荐用于开发）
from langchain_community.vectorstores import Chroma

# Pinecone（生产级）
from langchain_pinecone import PineconeVectorStore

# Qdrant（开源生产级）
from langchain_qdrant import QdrantVectorStore

# Weaviate
from langchain_weaviate import WeaviateVectorStore

# Milvus
from langchain_milvus import Milvus
```

---

## 十一、完整项目示例：智能客服助手

```python
"""
智能客服助手 - 完整的 LangChain 应用示例
"""

from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_community.vectorstores import Chroma
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_community.document_loaders import DirectoryLoader
from langchain.tools import tool
from langgraph.graph import StateGraph, END
from typing import TypedDict, List
from pydantic import BaseModel
import os

# 配置
os.environ["OPENAI_API_KEY"] = "your-api-key"

# 初始化 LLM
llm = ChatOpenAI(model="gpt-4o", temperature=0.7)

# 1. 加载知识库
def load_knowledge_base():
    loader = DirectoryLoader("./docs/", glob="**/*.md")
    documents = loader.load()
    
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=800,
        chunk_overlap=100
    )
    chunks = splitter.split_documents(documents)
    
    embeddings = OpenAIEmbeddings()
    vectorstore = Chroma.from_documents(
        documents=chunks,
        embedding=embeddings,
        persist_directory="./kb_chroma"
    )
    
    return vectorstore.as_retriever(search_kwargs={"k": 3})

retriever = load_knowledge_base()

# 2. 定义工具
@tool
def search_knowledge_base(query: str) -> str:
    """搜索知识库获取产品信息和常见问题解答。"""
    docs = retriever.invoke(query)
    return "\n\n".join([doc.page_content for doc in docs])

@tool
def create_ticket(issue: str, priority: str = "medium") -> str:
    """创建客服工单。优先级：low, medium, high, urgent"""
    # 模拟工单创建
    ticket_id = f"TKT-{os.urandom(4).hex()}"
    return f"工单已创建：{ticket_id}，优先级：{priority}"

tools = [search_knowledge_base, create_ticket]

# 3. 定义状态
class CustomerServiceState(TypedDict):
    customer_message: str
    intent: str
    knowledge_result: str
    ticket_id: str
    response: str

# 4. 定义节点
def classify_intent(state: CustomerServiceState):
    """分类客户意图"""
    classify_prompt = ChatPromptTemplate.from_messages([
        ("system", "分类客户意图为：product_question, technical_issue, billing, complaint, other"),
        ("human", "{message}")
    ])
    chain = classify_prompt | llm | StrOutputParser()
    intent = chain.invoke({"message": state["customer_message"]})
    return {"intent": intent}

def search_knowledge(state: CustomerServiceState):
    """搜索知识库"""
    if state["intent"] in ["product_question", "technical_issue"]:
        result = search_knowledge_base.invoke(state["customer_message"])
        return {"knowledge_result": result}
    return {"knowledge_result": ""}

def generate_response(state: CustomerServiceState):
    """生成回复"""
    response_prompt = ChatPromptTemplate.from_messages([
        ("system", """你是专业的客服助手。根据以下信息回复客户：
- 如果知识库有答案，详细解答
- 如果需要，创建工单
- 语气友好、专业"""),
        ("human", "客户消息：{message}\n意图：{intent}\n知识库：{knowledge}")
    ])
    chain = response_prompt | llm | StrOutputParser()
    response = chain.invoke({
        "message": state["customer_message"],
        "intent": state["intent"],
        "knowledge": state["knowledge_result"]
    })
    return {"response": response}

def create_ticket_if_needed(state: CustomerServiceState):
    """必要时创建工单"""
    if state["intent"] in ["technical_issue", "complaint"]:
        ticket = create_ticket.invoke({
            "issue": state["customer_message"],
            "priority": "high" if state["intent"] == "complaint" else "medium"
        })
        return {"ticket_id": ticket}
    return {"ticket_id": ""}

# 5. 构建工作流
workflow = StateGraph(CustomerServiceState)
workflow.add_node("classify", classify_intent)
workflow.add_node("search", search_knowledge)
workflow.add_node("respond", generate_response)
workflow.add_node("ticket", create_ticket_if_needed)

workflow.set_entry_point("classify")
workflow.add_edge("classify", "search")
workflow.add_edge("search", "respond")
workflow.add_edge("respond", "ticket")
workflow.add_edge("ticket", END)

app = workflow.compile()

# 6. 使用示例
def customer_service(message: str):
    result = app.invoke({
        "customer_message": message,
        "intent": "",
        "knowledge_result": "",
        "ticket_id": "",
        "response": ""
    })
    
    print(f"回复：{result['response']}")
    if result['ticket_id']:
        print(f"工单：{result['ticket_id']}")
    return result

# 测试
customer_service("你们的产品支持 API 集成吗？")
customer_service("我的账户无法登录，一直报错")
```

---

## 十二、总结

LangChain 已经发展成为一个完整的 AI 应用开发生态系统。从 2022 年的简单框架，到 2026 年的生产级平台，LangChain 提供了：

**核心优势：**
- ✅ 统一的 LLM 接口
- ✅ 强大的 RAG 能力
- ✅ 灵活的 Agent 架构
- ✅ 丰富的工具集成
- ✅ 生产级特性（LangGraph、LangSmith）

**学习路径建议：**

1. **入门**：掌握 LLM、Prompt、Output Parser
2. **进阶**：学习 Chains、LCEL、Memory
3. **高级**：深入 RAG、Tools、Agents
4. **生产**：使用 LangGraph 构建复杂工作流

**2026 年最佳实践：**
- 使用 LangGraph 构建 Agent（而非传统 AgentExecutor）
- 采用 LCEL 声明式语法
- 实现完善的错误处理和监控
- 使用 LangSmith 进行调试和监控

**参考资源：**
- 官方文档：https://python.langchain.com/
- LangGraph 文档：https://langchain-ai.github.io/langgraph/
- GitHub：https://github.com/langchain-ai/langchain
- LangSmith：https://smith.langchain.com/

掌握 LangChain，你将能够构建从简单聊天机器人到复杂自主 Agent 的各种 AI 应用。开始动手实践吧！🚀
