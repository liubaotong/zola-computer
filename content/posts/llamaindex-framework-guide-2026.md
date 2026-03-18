+++
title = "LlamaIndex 开发框架完全指南 2026：RAG 应用从入门到实战"
date = "2026-03-18T17:00:00+08:00"

[taxonomies]
tags = ["LlamaIndex", "RAG", "AI", "向量数据库", "Python", "知识库"]

[extra]
summary = "全面介绍 LlamaIndex RAG 开发框架，包含核心概念、数据加载、索引构建、查询引擎、高级 RAG 技巧、Agent 集成等完整知识，附带丰富的实战代码示例和生产级部署方案。"
author = "博主"
+++

在构建基于大语言模型的应用时，**LlamaIndex** 已成为连接私有数据与 LLM 的首选框架。作为一个专注于数据索引和检索的工具，LlamaIndex 让开发者能够轻松构建 Retrieval-Augmented Generation (RAG) 应用，使 LLM 能够访问和理解你的私有数据。

本文将深入介绍 LlamaIndex 的核心概念、使用方法和最佳实践，通过完整的代码示例，帮助你掌握构建生产级 RAG 应用的关键技术。

---

## 一、什么是 LlamaIndex？

### LlamaIndex 简介

**LlamaIndex**（原名 GPT Index）是一个开源的数据框架，用于构建 LLM 应用，特别是 RAG（检索增强生成）系统。它提供了从数据加载、索引、检索到生成的完整工具链。

**核心理念**：让 LLM 能够访问和理解你的私有数据，而不仅仅依赖训练数据。

**官方网站**：https://www.llamaindex.ai/

**GitHub**：https://github.com/run-llama/llama_index（40K+ Stars）

**文档**：https://docs.llamaindex.ai/

### RAG 基础概念

RAG（Retrieval-Augmented Generation）检索增强生成的核心流程：

```
用户问题 → 检索相关文档 → 构建提示词 → LLM 生成回答
              ↓
          向量数据库
```

**为什么需要 RAG？**

| 问题 | 传统 LLM | RAG + LLM |
|------|---------|----------|
| **数据时效性** | 知识截止训练日期 | 可访问最新数据 |
| **私有数据** | 无法访问 | 可索引和检索 |
| **幻觉问题** | 容易编造信息 | 基于检索内容回答 |
| **可追溯性** | 难以溯源 | 可引用来源文档 |
| **成本** | 需要微调 | 无需微调 |

### LlamaIndex 核心组件

```
┌─────────────────────────────────────────────────────────┐
│                    LlamaIndex 应用层                      │
├─────────────────────────────────────────────────────────┤
│  Chat Engines  │  Query Engines  │  Agents  │  Workflows │
├─────────────────────────────────────────────────────────┤
│  Retrievers  │  Response Synthesizers  │  Postprocessors │
├─────────────────────────────────────────────────────────┤
│  Indices  │  Vector Stores  │  Embeddings  │  LLMs      │
├─────────────────────────────────────────────────────────┤
│  Data Connectors (LlamaHub)  │  Document Parsers         │
└─────────────────────────────────────────────────────────┘
```

### LlamaIndex vs LangChain

| 特性 | LlamaIndex | LangChain |
|------|-----------|----------|
| **核心定位** | 数据索引和检索 | 通用 LLM 应用编排 |
| **RAG 支持** | 深度优化，开箱即用 | 需要手动配置 |
| **数据连接器** | 200+ (LlamaHub) | 100+ |
| **学习曲线** | 较低，专注 RAG | 较高，功能广泛 |
| **适用场景** | 知识库、问答系统 | 多样化 LLM 应用 |

---

## 二、快速开始

### 2.1 环境要求

- **Python**: 3.9 - 3.13
- **操作系统**: Linux / macOS / Windows
- **内存**: 最低 2GB，推荐 4GB+

### 2.2 安装 LlamaIndex

```bash
# 创建虚拟环境
python -m venv llamaindex-env
source llamaindex-env/bin/activate  # Windows: llamaindex-env\Scripts\activate

# 安装核心包
pip install llama-index

# 安装 OpenAI 集成
pip install llama-index-llms-openai
pip install llama-index-embeddings-openai

# 安装文件读取支持
pip install llama-index-readers-file

# 安装 ChromaDB 向量存储
pip install llama-index-vector-stores-chroma

# 安装其他常用组件
pip install python-dotenv chromadb pypdf
```

### 2.3 配置 API 密钥

创建 `.env` 文件：

```bash
# .env
OPENAI_API_KEY=sk-your-openai-api-key
```

在代码中加载：

```python
from dotenv import load_dotenv
load_dotenv()
```

### 2.4 第一个 RAG 应用（50 行代码）

```python
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, Settings
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding

# 配置模型
Settings.llm = OpenAI(model="gpt-4o-mini", temperature=0.1)
Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-small")

# 加载文档
documents = SimpleDirectoryReader("./data").load_data()

# 创建索引
index = VectorStoreIndex.from_documents(documents)

# 创建查询引擎
query_engine = index.as_query_engine(similarity_top_k=3)

# 查询
response = query_engine.query("你的问题是什么？")
print(response)
```

---

## 三、核心组件详解

### 3.1 数据加载（Data Loaders）

LlamaIndex 支持 200+ 数据源。

#### 简单目录读取

```python
from llama_index.core import SimpleDirectoryReader

# 加载目录下所有支持的文件
documents = SimpleDirectoryReader("./data").load_data()

# 加载特定类型文件
documents = SimpleDirectoryReader(
    "./data",
    required_exts=[".pdf", ".txt", ".md"]
).load_data()

# 递归加载子目录
documents = SimpleDirectoryReader(
    "./data",
    recursive=True
).load_data()

print(f"加载了 {len(documents)} 个文档")
```

#### 专用读取器

```python
from llama_index.readers.file import PDFReader, DocxReader, CSVReader
from llama_index.readers.web import SimpleWebPageReader
from llama_index.readers.database import DatabaseReader

# PDF 读取
pdf_reader = PDFReader()
pdf_docs = pdf_reader.load_data(file_path="./data/report.pdf")

# Word 文档
docx_reader = DocxReader()
docx_docs = docx_reader.load_data(file_path="./data/document.docx")

# 网页读取
web_reader = SimpleWebPageReader(html_to_text=True)
web_docs = web_reader.load_data(urls=["https://example.com/page1"])

# 数据库读取
db_reader = DatabaseReader(
    scheme="postgresql",
    host="localhost",
    port=5432,
    user="user",
    password="password",
    dbname="mydb"
)
db_docs = db_reader.load_data(query="SELECT * FROM articles")
```

#### LlamaHub 社区连接器

```python
# 安装特定连接器
pip install llama-index-readers-wikipedia
pip install llama-index-readers-google
pip install llama-index-readers-notion

# Wikipedia 读取
from llama_index.readers.wikipedia import WikipediaReader
wiki_docs = WikipediaReader().load_data(pages=["Python", "人工智能"])

# Google Drive 读取
from llama_index.readers.google import GoogleDocsReader
gdoc_ids = ["doc_id_1", "doc_id_2"]
gdocs = GoogleDocsReader().load_data(document_ids=gdoc_ids)

# Notion 读取
from llama_index.readers.notion import NotionPageReader
notion_docs = NotionPageReader(integration_token="your-token").load_data(
    page_ids=["page_id_1", "page_id_2"]
)
```

### 3.2 文档分块（Text Splitting）

合理的分块策略对检索质量至关重要。

#### SentenceSplitter（推荐）

```python
from llama_index.core.node_parser import SentenceSplitter

# 配置分块参数
parser = SentenceSplitter(
    chunk_size=1024,      # 每块 token 数
    chunk_overlap=200,    # 块间重叠 token 数
    separator="\n",       # 主要分隔符
    paragraph_separator="\n\n\n",  # 段落分隔符
    secondary_chunking_regex="[^,.;。！？]+[,.;。！？]?"  # 次要分块（按句子）
)

nodes = parser.get_nodes_from_documents(documents)
print(f"分成了 {len(nodes)} 个节点")
```

#### 其他分块策略

```python
from llama_index.core.node_parser import (
    TokenTextSplitter,      # 按 token 分块
    SemanticSplitterNodeParser,  # 语义分块
    MarkdownNodeParser,    # Markdown 专用
    HTMLNodeParser,        # HTML 专用
    CodeSplitter           # 代码专用
)

# Token 分块（精确控制）
token_parser = TokenTextSplitter(
    chunk_size=512,
    chunk_overlap=50
)

# 语义分块（更智能）
from llama_index.embeddings.openai import OpenAIEmbedding
semantic_parser = SemanticSplitterNodeParser(
    buffer_size=1,
    breakpoint_percentile_threshold=95,
    embed_model=OpenAIEmbedding()
)

# Markdown 分块（保留结构）
md_parser = MarkdownNodeParser(
    include_metadata=True
)

# 代码分块（保留语法结构）
code_parser = CodeSplitter(
    language="python",
    chunk_lines=50,
    chunk_lines_overlap=10
)
```

#### 分块策略对比

| 策略 | 适用场景 | 优点 | 缺点 |
|------|---------|------|------|
| **SentenceSplitter** | 通用文档 | 保持句子完整性 | 可能切断语义 |
| **TokenTextSplitter** | 精确控制 | 大小一致 | 忽略语义 |
| **SemanticSplitter** | 长文档 | 语义连贯 | 计算成本高 |
| **MarkdownNodeParser** | 技术文档 | 保留结构 | 仅适用 Markdown |

### 3.3 向量化（Embeddings）

#### 配置嵌入模型

```python
from llama_index.embeddings.openai import OpenAIEmbedding
from llama_index.core import Settings

# 设置全局嵌入模型
Settings.embed_model = OpenAIEmbedding(
    model="text-embedding-3-small",  # 性价比高
    # model="text-embedding-3-large",  # 质量更好
    embed_batch_size=100  # 批量处理
)

# 或使用本地模型（Ollama）
from llama_index.embeddings.ollama import OllamaEmbedding
Settings.embed_model = OllamaEmbedding(
    model_name="mxbai-embed-large",
    base_url="http://localhost:11434"
)
```

#### 主流嵌入模型对比

| 模型 | 维度 | 成本 | 质量 | 适用场景 |
|------|------|------|------|---------|
| **text-embedding-3-small** | 1536 | $ | 好 | 通用 |
| **text-embedding-3-large** | 3072 | $$ | 最好 | 高质量需求 |
| **mxbai-embed-large** | 1024 | 免费 | 好 | 本地部署 |
| **bge-large-zh** | 1024 | 免费 | 好（中文） | 中文场景 |

### 3.4 索引构建（Indexing）

#### VectorStoreIndex（最常用）

```python
from llama_index.core import VectorStoreIndex

# 从文档创建索引
index = VectorStoreIndex.from_documents(
    documents,
    show_progress=True  # 显示进度条
)

# 从节点创建索引
nodes = parser.get_nodes_from_documents(documents)
index = VectorStoreIndex(nodes)

# 查询
query_engine = index.as_query_engine()
response = query_engine.query("问题")
```

#### 持久化存储

```python
from llama_index.core import StorageContext, persist_dir
from llama_index.core import VectorStoreIndex

# 保存到磁盘
index.storage_context.persist(persist_dir="./storage")

# 从磁盘加载
from llama_index.core import load_index_from_storage
storage_context = StorageContext.from_defaults(persist_dir="./storage")
index = load_index_from_storage(storage_context)

# 避免重复嵌入
import os
if os.path.exists("./storage"):
    index = load_index_from_storage(storage_context)
    print("加载已有索引")
else:
    index = VectorStoreIndex.from_documents(documents)
    index.storage_context.persist(persist_dir="./storage")
    print("创建新索引")
```

### 3.5 向量数据库集成

#### ChromaDB（本地推荐）

```python
from llama_index.core import StorageContext
from llama_index.vector_stores.chroma import ChromaVectorStore
import chromadb

# 初始化 ChromaDB
chroma_client = chromadb.PersistentClient(path="./chroma_db")
chroma_collection = chroma_client.get_or_create_collection("my_docs")

# 创建向量存储
vector_store = ChromaVectorStore(chroma_collection=chroma_collection)
storage_context = StorageContext.from_defaults(vector_store=vector_store)

# 创建索引
index = VectorStoreIndex.from_documents(
    documents,
    storage_context=storage_context
)

# 从现有向量存储加载
index = VectorStoreIndex.from_vector_store(
    vector_store=vector_store
)
```

#### 其他向量数据库

```python
# Pinecone（云端）
from llama_index.vector_stores.pinecone import PineconeVectorStore
import pinecone

pinecone.init(api_key="your-key", environment="us-west1-gcp")
index = PineconeVectorStore(index_name="my-index")

# Qdrant（开源）
from llama_index.vector_stores.qdrant import QdrantVectorStore
from qdrant_client import QdrantClient

client = QdrantClient(url="http://localhost:6333")
vector_store = QdrantVectorStore(client=client, collection_name="my_docs")

# Milvus
from llama_index.vector_stores.milvus import MilvusVectorStore

vector_store = MilvusVectorStore(
    uri="./milvus_demo.db",
    token="",
    dim=1536
)

# Weaviate
from llama_index.vector_stores.weaviate import WeaviateVectorStore
import weaviate

client = weaviate.Client("http://localhost:8080")
vector_store = WeaviateVectorStore(weaviate_client=client)
```

---

## 四、查询引擎（Query Engines）

### 4.1 基础查询

```python
from llama_index.core import VectorStoreIndex

# 创建索引
index = VectorStoreIndex.from_documents(documents)

# 基础查询引擎
query_engine = index.as_query_engine(
    similarity_top_k=5,  # 返回最相关的 5 个文档
    verbose=True  # 显示详细信息
)

response = query_engine.query("你的问题")
print(response)
print(response.source_nodes)  # 查看来源
```

### 4.2 响应合成模式

```python
from llama_index.core.response_synthesizers import get_response_synthesizer, ResponseMode

# 1. default - 默认模式（适合简单查询）
query_engine = index.as_query_engine(
    response_mode="default",
    similarity_top_k=3
)

# 2. compact - 压缩模式（节省 token）
query_engine = index.as_query_engine(
    response_mode="compact",
    similarity_top_k=5
)

# 3. tree_summarize - 树形总结（适合长文档）
query_engine = index.as_query_engine(
    response_mode="tree_summarize",
    similarity_top_k=10
)

# 4. accumulate - 累积模式（保留所有信息）
query_engine = index.as_query_engine(
    response_mode="accumulate",
    similarity_top_k=5
)

# 5. compact_accumulate - 压缩累积（平衡）
query_engine = index.as_query_engine(
    response_mode="compact_accumulate",
    similarity_top_k=5
)
```

#### 响应模式对比

| 模式 | 描述 | 适用场景 | 成本 |
|------|------|---------|------|
| **default** | 直接使用最相关文档 | 简单查询 | $ |
| **compact** | 压缩文档到最小提示 | 成本敏感 | $ |
| **tree_summarize** | 递归总结文档树 | 长文档总结 | $$$ |
| **accumulate** | 分别处理每个文档后合并 | 需要完整信息 | $$ |
| **compact_accumulate** | 压缩后累积 | 平衡场景 | $$ |

### 4.3 自定义查询引擎

```python
from llama_index.core.query_engine import RetrieverQueryEngine
from llama_index.core.retrievers import VectorIndexRetriever
from llama_index.core.response_synthesizers import get_response_synthesizer
from llama_index.core.postprocessor import SimilarityPostprocessor

# 自定义检索器
retriever = VectorIndexRetriever(
    index=index,
    similarity_top_k=5,
    vector_store_query_mode="default"  # 或 "hybrid"
)

# 响应合成器
response_synthesizer = get_response_synthesizer(
    response_mode="tree_summarize",
    streaming=True  # 启用流式输出
)

# 后处理器
node_postprocessors = [
    SimilarityPostprocessor(similarity_cutoff=0.7)  # 相似度阈值
]

# 组装查询引擎
query_engine = RetrieverQueryEngine(
    retriever=retriever,
    response_synthesizer=response_synthesizer,
    node_postprocessors=node_postprocessors
)

response = query_engine.query("你的问题")
```

### 4.4 流式查询

```python
# 启用流式输出
query_engine = index.as_query_engine(streaming=True)

# 流式响应
response = query_engine.query("你的问题")
for chunk in response.response_gen:
    print(chunk, end="", flush=True)

# 异步流式
async def query_stream():
    response = await query_engine.aquery("你的问题")
    async for chunk in response.async_response_gen():
        print(chunk, end="", flush=True)
```

---

## 五、高级 RAG 技巧

### 5.1 混合检索（Hybrid Search）

结合稠密检索（语义）和稀疏检索（关键词）。

```python
from llama_index.core import VectorStoreIndex
from llama_index.core.retrievers import RecursiveRetriever
from llama_index.retrievers.bm25 import BM25Retriever
from llama_index.core.retrievers import QueryFusionRetriever

# 创建向量检索器
vector_retriever = index.as_retriever(similarity_top_k=3)

# 创建 BM25 检索器（关键词）
bm25_retriever = BM25Retriever.from_defaults(
    docstore=index.docstore,
    similarity_top_k=3
)

# 融合检索器
retriever = QueryFusionRetriever(
    retrievers=[vector_retriever, bm25_retriever],
    similarity_top_k=5,
    num_queries=1,
    mode="reciprocal_rerank",  # 倒数排名融合
    verbose=True
)

query_engine = RetrieverQueryEngine.from_args(
    retriever=retriever,
    llm=Settings.llm
)

response = query_engine.query("你的问题")
```

### 5.2 元数据过滤

```python
from llama_index.core import VectorStoreIndex, Document
from llama_index.core.vector_stores import MetadataFilter, MetadataFilters

# 创建带元数据的文档
documents = [
    Document(
        text="文档内容 1",
        metadata={"category": "技术", "year": 2024, "author": "张三"}
    ),
    Document(
        text="文档内容 2",
        metadata={"category": "产品", "year": 2025, "author": "李四"}
    )
]

index = VectorStoreIndex.from_documents(documents)

# 创建过滤器
filters = MetadataFilters(
    filters=[
        MetadataFilter(key="category", value="技术"),
        MetadataFilter(key="year", value=2024, op=">")  # 大于 2024
    ]
)

# 带过滤的查询
query_engine = index.as_query_engine(filters=filters)
response = query_engine.query("技术问题")
```

### 5.3 查询重写（Query Rewriting）

```python
from llama_index.core.query_transform import StepDecomposeQueryTransform
from llama_index.core import QueryBundle

# 查询重写（将复杂问题分解）
query_transform = StepDecomposeQueryTransform(
    llm=Settings.llm,
    verbose=True
)

# 原始查询
query_bundle = QueryBundle("LlamaIndex 和 LangChain 有什么区别？各自的优缺点是什么？")

# 重写查询
new_query = query_transform.run(query_bundle)
print(new_query)

# 使用重写后的查询
query_engine = index.as_query_engine()
response = query_engine.query(new_query)
```

### 5.4 文档重排序（Reranking）

```python
from llama_index.postprocessor.cohere_rerank import CohereRerank
from llama_index.core.query_engine import RetrieverQueryEngine

# 创建检索器
retriever = index.as_retriever(similarity_top_k=10)

# 添加重排序
node_postprocessors = [
    CohereRerank(
        api_key="your-cohere-key",
        top_n=5  # 重排序后保留 5 个
    )
]

query_engine = RetrieverQueryEngine(
    retriever=retriever,
    node_postprocessors=node_postprocessors
)

response = query_engine.query("你的问题")
```

### 5.5 多索引检索

```python
from llama_index.core import VectorStoreIndex
from llama_index.core.retrievers import RecursiveRetriever
from llama_index.core.query_engine import RetrieverQueryEngine

# 创建多个索引
index1 = VectorStoreIndex.from_documents(tech_docs)
index2 = VectorStoreIndex.from_documents(product_docs)
index3 = VectorStoreIndex.from_documents(hr_docs)

# 创建多个检索器
retriever1 = index1.as_retriever()
retriever2 = index2.as_retriever()
retriever3 = index3.as_retriever()

# 递归检索器（自动路由）
retriever = RecursiveRetriever(
    root_id="root",
    retriever_dict={
        "tech": retriever1,
        "product": retriever2,
        "hr": retriever3
    }
)

query_engine = RetrieverQueryEngine.from_args(retriever)
response = query_engine.query("技术问题")
```

---

## 六、Chat Engines（对话引擎）

### 6.1 基础对话

```python
from llama_index.core.memory import ChatMemoryBuffer
from llama_index.core.chat_engine import SimpleChatEngine

# 简单对话引擎（无检索）
chat_engine = SimpleChatEngine.from_defaults(
    llm=Settings.llm,
    system_prompt="你是一个友好的 AI 助手"
)

# 多轮对话
response = chat_engine.chat("你好")
print(response)

response = chat_engine.chat("我叫小明，记住我的名字")
print(response)

response = chat_engine.chat("我叫什么名字？")  # 会记得
print(response)
```

### 6.2 上下文对话

```python
from llama_index.core.chat_engine import ContextChatEngine
from llama_index.core.memory import ChatMemoryBuffer

# 带上下文的对话引擎（基于检索）
chat_engine = ContextChatEngine.from_defaults(
    index=index,
    llm=Settings.llm,
    system_prompt="基于检索到的内容回答用户问题",
    memory=ChatMemoryBuffer.from_defaults(token_limit=4000)
)

# 对话
response = chat_engine.chat("文档中提到了哪些关键技术？")
print(response)

response = chat_engine.chat("详细解释一下第一个技术")
print(response)
```

### 6.3 条件对话

```python
from llama_index.core.chat_engine import CondenseQuestionChatEngine

# 压缩问题对话引擎（优化多轮对话）
chat_engine = CondenseQuestionChatEngine.from_defaults(
    query_engine=index.as_query_engine(),
    llm=Settings.llm,
    verbose=True
)

# 第一轮
response = chat_engine.chat("LlamaIndex 的主要功能是什么？")
print(response)

# 第二轮（会自动压缩上下文）
response = chat_engine.chat("如何使用它构建 RAG 系统？")
print(response)
```

---

## 七、Agent 集成

### 7.1 FunctionAgent（函数调用 Agent）

```python
from llama_index.core.agent.workflow import FunctionAgent
from llama_index.core.tools import FunctionTool
from llama_index.core import Settings

# 定义工具函数
def multiply(a: float, b: float) -> float:
    """计算两个数的乘积"""
    return a * b

def add(a: float, b: float) -> float:
    """计算两个数的和"""
    return a + b

# 创建工具
multiply_tool = FunctionTool.from_defaults(fn=multiply)
add_tool = FunctionTool.from_defaults(fn=add)

# 创建 Agent
agent = FunctionAgent(
    name="计算器助手",
    system_prompt="你是一个数学助手，可以使用工具进行计算",
    tools=[multiply_tool, add_tool],
    llm=Settings.llm
)

# 运行 Agent
from llama_index.core.agent.workflow import WorkflowRunner

runner = WorkflowRunner()
result = await runner.run(agent, "计算 25 乘以 35，然后加上 100")
print(result)
```

### 7.2 RAG + Agent

```python
from llama_index.core.tools import QueryEngineTool
from llama_index.core.agent.workflow import FunctionAgent

# 创建查询引擎工具
query_engine_tool = QueryEngineTool.from_defaults(
    query_engine=index.as_query_engine(),
    name="knowledge_base",
    description="搜索知识库获取相关信息"
)

# 创建带 RAG 能力的 Agent
agent = FunctionAgent(
    name="知识助手",
    system_prompt="""你是一个知识渊博的助手。
    当用户询问文档相关内容时，使用 knowledge_base 工具搜索。
    如果不知道答案，诚实地告诉用户。""",
    tools=[query_engine_tool],
    llm=Settings.llm
)

# 运行
runner = WorkflowRunner()
result = await runner.run(agent, "文档中关于 RAG 的最佳实践有哪些？")
print(result)
```

### 7.3 多 Agent 协作

```python
from llama_index.core.agent.workflow import AgentWorkflow, FunctionAgent

# 创建专业 Agent
researcher = FunctionAgent(
    name="研究员",
    system_prompt="你负责收集和整理信息",
    tools=[query_engine_tool],
    llm=Settings.llm
)

analyst = FunctionAgent(
    name="分析师",
    system_prompt="你负责分析信息并提供洞察",
    tools=[],
    llm=Settings.llm
)

writer = FunctionAgent(
    name="作家",
    system_prompt="你负责撰写清晰的报告",
    tools=[],
    llm=Settings.llm
)

# 创建工作流
workflow = AgentWorkflow(
    agents=[researcher, analyst, writer],
    root_agent="researcher"  # 从研究员开始
)

# 运行工作流
runner = WorkflowRunner()
result = await runner.run(
    workflow,
    "研究 LlamaIndex 的应用场景并生成报告"
)
print(result)
```

---

## 八、LlamaCloud（企业级服务）

### 8.1 使用 LlamaCloud

```python
from llama_index.cloud import LlamaCloudIndex

# 连接到 LlamaCloud
index = LlamaCloudIndex(
    name="my-index",
    organization_id="your-org-id",
    token="your-cloud-token"
)

# 查询
query_engine = index.as_query_engine()
response = query_engine.query("你的问题")
```

### 8.2 LlamaParse（高级文档解析）

```python
from llama_parse import LlamaParse

# 解析复杂文档（表格、图表等）
parser = LlamaParse(
    api_key="your-llama-parse-key",
    result_type="markdown",  # 或 "text"
    verbose=True
)

# 解析 PDF
documents = parser.load_data("./complex_document.pdf")

# 解析多个文件
documents = parser.load_data([
    "./doc1.pdf",
    "./doc2.pdf"
])
```

---

## 九、实战项目

### 9.1 企业知识库问答系统

```python
"""
企业知识库问答系统 - 完整的 RAG 应用
"""

import os
from pathlib import Path
from llama_index.core import (
    VectorStoreIndex,
    SimpleDirectoryReader,
    StorageContext,
    Settings,
    load_index_from_storage
)
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding
from llama_index.vector_stores.chroma import ChromaVectorStore
from llama_index.core.query_engine import RetrieverQueryEngine
from llama_index.core.retrievers import VectorIndexRetriever
from llama_index.core.postprocessor import SimilarityPostprocessor
from llama_index.core.response_synthesizers import get_response_synthesizer
import chromadb
from dotenv import load_dotenv

load_dotenv()

class KnowledgeBaseQA:
    """知识库问答系统"""
    
    def __init__(self, data_dir: str, persist_dir: str = "./storage"):
        self.data_dir = data_dir
        self.persist_dir = persist_dir
        
        # 配置模型
        Settings.llm = OpenAI(model="gpt-4o", temperature=0.1)
        Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-small")
        
        # 加载或创建索引
        self.index = self._load_or_create_index()
        
        # 创建查询引擎
        self.query_engine = self._create_query_engine()
    
    def _load_or_create_index(self):
        """加载或创建索引"""
        if os.path.exists(self.persist_dir):
            print("加载已有索引...")
            storage_context = StorageContext.from_defaults(
                persist_dir=self.persist_dir
            )
            return load_index_from_storage(storage_context)
        else:
            print("创建新索引...")
            documents = SimpleDirectoryReader(self.data_dir).load_data()
            print(f"加载了 {len(documents)} 个文档")
            
            # 使用 ChromaDB 持久化
            chroma_client = chromadb.PersistentClient(path="./chroma_db")
            chroma_collection = chroma_client.get_or_create_collection("knowledge")
            vector_store = ChromaVectorStore(chroma_collection=chroma_collection)
            storage_context = StorageContext.from_defaults(vector_store=vector_store)
            
            index = VectorStoreIndex.from_documents(
                documents,
                storage_context=storage_context,
                show_progress=True
            )
            
            # 保存索引
            index.storage_context.persist(persist_dir=self.persist_dir)
            return index
    
    def _create_query_engine(self):
        """创建查询引擎"""
        # 检索器
        retriever = VectorIndexRetriever(
            index=self.index,
            similarity_top_k=5
        )
        
        # 后处理器
        node_postprocessors = [
            SimilarityPostprocessor(similarity_cutoff=0.7)
        ]
        
        # 响应合成器
        response_synthesizer = get_response_synthesizer(
            response_mode="tree_summarize",
            streaming=True
        )
        
        return RetrieverQueryEngine(
            retriever=retriever,
            node_postprocessors=node_postprocessors,
            response_synthesizer=response_synthesizer
        )
    
    def query(self, question: str, stream: bool = False):
        """查询知识库"""
        response = self.query_engine.query(question)
        
        if stream:
            for chunk in response.response_gen:
                print(chunk, end="", flush=True)
            print()
        else:
            print(response)
        
        # 显示来源
        print("\n来源文档:")
        for i, node in enumerate(response.source_nodes, 1):
            print(f"{i}. {node.metadata.get('file_name', 'Unknown')}")
        
        return response
    
    def chat(self):
        """交互式对话"""
        print("知识库问答系统已启动（输入 'quit' 退出）")
        
        while True:
            question = input("\n问题：").strip()
            if question.lower() in ['quit', 'exit', 'q']:
                break
            
            self.query(question, stream=True)

if __name__ == "__main__":
    # 初始化系统
    kb = KnowledgeBaseQA(data_dir="./data/docs")
    
    # 交互式对话
    kb.chat()
```

### 9.2 多文档类型 RAG 系统

```python
"""
多文档类型 RAG 系统 - 支持 PDF、Word、网页等
"""

from llama_index.core import VectorStoreIndex, Settings
from llama_index.readers.file import PDFReader, DocxReader
from llama_index.readers.web import SimpleWebPageReader
from llama_index.core.node_parser import MarkdownNodeParser, SentenceSplitter
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding

class MultiSourceRAG:
    """多源 RAG 系统"""
    
    def __init__(self):
        # 配置模型
        Settings.llm = OpenAI(model="gpt-4o", temperature=0.1)
        Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-large")
        
        self.documents = []
    
    def load_pdfs(self, pdf_dir: str):
        """加载 PDF 文档"""
        reader = PDFReader()
        docs = reader.load_data(Path(pdf_dir))
        self.documents.extend(docs)
        print(f"加载了 {len(docs)} 个 PDF 文档")
    
    def load_word_docs(self, docx_dir: str):
        """加载 Word 文档"""
        reader = DocxReader()
        docs = reader.load_data(Path(docx_dir))
        self.documents.extend(docs)
        print(f"加载了 {len(docs)} 个 Word 文档")
    
    def load_web_pages(self, urls: list):
        """加载网页"""
        reader = SimpleWebPageReader(html_to_text=True)
        docs = reader.load_data(urls)
        self.documents.extend(docs)
        print(f"加载了 {len(docs)} 个网页")
    
    def build_index(self):
        """构建索引"""
        # 使用不同的分块器处理不同类型的文档
        all_nodes = []
        
        for doc in self.documents:
            if doc.metadata.get('file_type') == 'pdf':
                # PDF 使用句子分块
                parser = SentenceSplitter(chunk_size=1024, chunk_overlap=200)
            else:
                # 其他使用 Markdown 分块
                parser = MarkdownNodeParser()
            
            nodes = parser.get_nodes_from_documents([doc])
            all_nodes.extend(nodes)
        
        print(f"创建了 {len(all_nodes)} 个节点")
        
        # 创建索引
        self.index = VectorStoreIndex(all_nodes, show_progress=True)
        return self.index
    
    def query(self, question: str):
        """查询"""
        query_engine = self.index.as_query_engine(
            similarity_top_k=5,
            response_mode="tree_summarize"
        )
        
        response = query_engine.query(question)
        print(response)
        
        # 显示来源类型
        print("\n来源:")
        for node in response.source_nodes:
            source_type = node.metadata.get('file_type', 'Unknown')
            source_name = node.metadata.get('file_name', 'Web Page')
            print(f"- [{source_type}] {source_name}")
        
        return response

if __name__ == "__main__":
    rag = MultiSourceRAG()
    
    # 加载多种数据源
    rag.load_pdfs("./data/pdfs")
    rag.load_word_docs("./data/docs")
    rag.load_web_pages(["https://example.com/doc1", "https://example.com/doc2"])
    
    # 构建索引
    rag.build_index()
    
    # 查询
    rag.query("文档中提到了哪些关键技术？")
```

### 9.3 生产级 RAG 系统（带评估）

```python
"""
生产级 RAG 系统 - 包含评估和监控
"""

import time
from typing import List, Dict
from llama_index.core import VectorStoreIndex, Settings
from llama_index.core.evaluation import (
    FaithfulnessEvaluator,
    RelevancyEvaluator,
    AnswerRelevancyEvaluator
)
from llama_index.llms.openai import OpenAI
from llama_index.embeddings.openai import OpenAIEmbedding

class ProductionRAG:
    """生产级 RAG 系统"""
    
    def __init__(self, index: VectorStoreIndex):
        self.index = index
        Settings.llm = OpenAI(model="gpt-4o", temperature=0.1)
        
        # 创建评估器
        self.faithfulness_eval = FaithfulnessEvaluator(llm=Settings.llm)
        self.relevancy_eval = RelevancyEvaluator(llm=Settings.llm)
        
        # 性能指标
        self.metrics: List[Dict] = []
    
    def query_with_metrics(self, question: str) -> Dict:
        """带性能指标的查询"""
        start_time = time.time()
        
        # 查询
        query_engine = self.index.as_query_engine(
            similarity_top_k=5,
            response_mode="tree_summarize"
        )
        response = query_engine.query(question)
        
        # 计算指标
        query_time = time.time() - start_time
        retrieved_docs = len(response.source_nodes)
        
        # 评估答案质量
        faithfulness = self.faithfulness_eval.evaluate_response(
            query=question,
            response=response
        )
        
        relevancy = self.relevancy_eval.evaluate_response(
            query=question,
            response=response
        )
        
        # 记录指标
        metrics = {
            "question": question,
            "query_time": query_time,
            "retrieved_docs": retrieved_docs,
            "faithfulness_score": faithfulness.score,
            "relevancy_score": relevancy.score,
            "answer_length": len(response.response)
        }
        
        self.metrics.append(metrics)
        
        return {
            "response": response,
            "metrics": metrics
        }
    
    def get_performance_report(self) -> Dict:
        """生成性能报告"""
        if not self.metrics:
            return {"error": "No queries yet"}
        
        avg_query_time = sum(m["query_time"] for m in self.metrics) / len(self.metrics)
        avg_faithfulness = sum(m["faithfulness_score"] for m in self.metrics) / len(self.metrics)
        avg_relevancy = sum(m["relevancy_score"] for m in self.metrics) / len(self.metrics)
        
        return {
            "total_queries": len(self.metrics),
            "avg_query_time": f"{avg_query_time:.2f}s",
            "avg_faithfulness": f"{avg_faithfulness:.2f}",
            "avg_relevancy": f"{avg_relevancy:.2f}",
            "queries": self.metrics
        }
    
    def optimize_based_on_feedback(self, feedback_threshold: float = 0.7):
        """基于反馈优化"""
        low_quality_queries = [
            m for m in self.metrics 
            if m["faithfulness_score"] < feedback_threshold 
            or m["relevancy_score"] < feedback_threshold
        ]
        
        if low_quality_queries:
            print(f"发现 {len(low_quality_queries)} 个低质量回答")
            print("建议优化：")
            print("1. 增加 similarity_top_k")
            print("2. 调整 chunk_size")
            print("3. 使用更好的嵌入模型")
            print("4. 添加重排序")
        
        return low_quality_queries

# 使用示例
if __name__ == "__main__":
    # 创建索引
    documents = SimpleDirectoryReader("./data").load_data()
    index = VectorStoreIndex.from_documents(documents)
    
    # 创建生产系统
    rag = ProductionRAG(index)
    
    # 测试查询
    questions = [
        "什么是 RAG？",
        "LlamaIndex 的主要功能是什么？",
        "如何优化检索质量？"
    ]
    
    for q in questions:
        result = rag.query_with_metrics(q)
        print(f"\n问题：{q}")
        print(f"答案：{result['response']}")
        print(f"指标：{result['metrics']}")
    
    # 性能报告
    report = rag.get_performance_report()
    print("\n性能报告:")
    print(report)
```

---

## 十、最佳实践

### 10.1 分块策略优化

```python
# ✅ 推荐：根据文档类型选择分块器
if doc_type == "code":
    parser = CodeSplitter(language="python")
elif doc_type == "markdown":
    parser = MarkdownNodeParser()
else:
    parser = SentenceSplitter(chunk_size=1024, chunk_overlap=200)

# ✅ 推荐：测试不同 chunk_size
for chunk_size in [512, 1024, 2048]:
    parser = SentenceSplitter(chunk_size=chunk_size, chunk_overlap=200)
    # 测试检索质量

# ❌ 避免：固定使用单一分块策略
```

### 10.2 检索质量优化

```python
# ✅ 推荐：混合检索
retriever = QueryFusionRetriever([vector_retriever, bm25_retriever])

# ✅ 推荐：添加重排序
node_postprocessors = [CohereRerank(top_n=5)]

# ✅ 推荐：元数据过滤
filters = MetadataFilters(filters=[MetadataFilter(key="year", value=2024, op=">")])

# ❌ 避免：仅使用基础向量检索
```

### 10.3 成本控制

```python
# ✅ 推荐：使用较小的嵌入模型
Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-small")

# ✅ 推荐：限制检索文档数
query_engine = index.as_query_engine(similarity_top_k=3)

# ✅ 推荐：使用 compact 响应模式
query_engine = index.as_query_engine(response_mode="compact")

# ✅ 推荐：缓存索引
index.storage_context.persist(persist_dir="./storage")
```

### 10.4 监控和评估

```python
# ✅ 推荐：定期评估
evaluator = FaithfulnessEvaluator()
results = evaluator.evaluate_response(query, response)

# ✅ 推荐：记录查询日志
import logging
logging.info(f"Query: {question}, Time: {query_time}")

# ✅ 推荐：A/B 测试不同配置
```

---

## 十一、总结

LlamaIndex 作为专注于 RAG 的框架，提供了从数据加载到查询生成的完整工具链。

**核心优势：**
- ✅ **专注 RAG**：深度优化的检索和生成流程
- ✅ **丰富的连接器**：200+ 数据源支持
- ✅ **灵活的索引**：多种索引和向量数据库支持
- ✅ **高级特性**：混合检索、重排序、元数据过滤
- ✅ **Agent 集成**：无缝集成 Agent 工作流
- ✅ **企业级服务**：LlamaCloud、LlamaParse

**学习路径建议：**

1. **入门**：安装 → 第一个 RAG → 理解索引和查询
2. **进阶**：数据加载 → 分块策略 → 向量数据库
3. **高级**：混合检索 → 重排序 → 多索引
4. **生产**：评估 → 监控 → 性能优化

**参考资源：**
- 官方文档：https://docs.llamaindex.ai/
- GitHub：https://github.com/run-llama/llama_index
- LlamaHub：https://llamahub.ai/
- Discord 社区：https://discord.gg/dGcwcsnxhU

开始使用 LlamaIndex 构建你的 RAG 应用吧！🦙
