+++
title = "AI Agent Skill 开发完全指南 2026：从设计到实战"
date = "2026-03-18T22:00:00+08:00"

[taxonomies]
tags = ["AI Agent", "Skill 开发", "Claude Skills", "MCP", "工具设计", "最佳实践"]

[extra]
summary = "全面介绍 AI Agent Skill 开发，包含 Skill 定义、设计原则、结构规范、开发流程、测试方法，以及 Claude Skills、MCP Tools、LangChain Tools 等多种框架的 Skill 实现，附带丰富的实战案例和模板。"
author = "博主"
+++

在 AI Agent 生态系统中，**Skill（技能）** 是定义 Agent 能力和行为的核心单元。一个设计良好的 Skill 可以让 AI Agent 高效、可靠地完成特定任务，而糟糕的 Skill 设计则会导致 Agent 行为不可预测、效率低下。

本文将深入介绍 AI Agent Skill 的完整开发流程，从概念设计到实现测试，涵盖 Claude Skills、MCP Tools、LangChain Tools 等多种框架的 Skill 开发实践。

---

## 一、什么是 Skill？

### 1.1 Skill 的定义

**Skill（技能）** 是 AI Agent 执行特定任务的能力单元，它定义了：
- **触发条件**：什么情况下激活这个技能
- **执行逻辑**：如何完成任务
- **输出格式**：返回什么结果
- **边界约束**：什么能做，什么不能做

### 1.2 Skill vs Tool vs MCP

这三个概念经常被混淆，但它们有明确的区别：

| 概念 | 定义 | 位置 | 示例 |
|------|------|------|------|
| **Skill** | 程序性智能（如何思考） | Agent 内部 | "创建技术文档"的思维流程 |
| **Tool** | 执行能力（能做什么） | Agent 内部 | 调用函数、执行代码 |
| **MCP** | 外部连接协议 | Agent 边界 | 连接 Jira、Confluence 的协议 |

```
┌─────────────────────────────────────────────────────────┐
│                      AI Agent                            │
│  ┌─────────────────────────────────────────────────┐    │
│  │                    Skills                        │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐      │    │
│  │  │ 创建文档  │  │ 代码审查  │  │ 数据分析  │      │    │
│  │  └──────────┘  └──────────┘  └──────────┘      │    │
│  │                                                  │    │
│  │  ┌──────────────────────────────────────────┐   │    │
│  │  │               Tools                       │   │    │
│  │  │  [函数调用] [代码执行] [API 请求]          │   │    │
│  │  └──────────────────────────────────────────┘   │    │
│  └─────────────────────────────────────────────────┘    │
│                          ↕ MCP Protocol                  │
│  ┌─────────────────────────────────────────────────┐    │
│  │              外部系统 (Jira, GitHub...)          │    │
│  └─────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────┘
```

**关键区别：**
- **Skill** = 知道**如何做**（程序性知识）
- **Tool** = 知道**能做什么**（执行能力）
- **MCP** = 知道**如何连接**（通信协议）

### 1.3 Skill 的价值

```
没有 Skill 的 Agent:
用户："创建技术文档"
Agent: "好的，我来写..."
→ 随机生成，质量不稳定，格式不统一

有 Skill 的 Agent:
用户："创建技术文档"
Agent: "好的，使用 create-tech-spec skill..."
→ 按标准流程，提取需求 → 结构化章节 → 论证决策 → 评估风险
→ 输出一致、高质量
```

---

## 二、Skill 设计原则

### 2.1 单一职责原则

```markdown
# ✅ 好的设计
- `create-tech-spec`: 创建技术规格文档
- `review-code`: 代码审查
- `analyze-data`: 数据分析

# ❌ 糟糕的设计
- `do-everything`: 什么都做
- `handle-all-tasks`: 处理所有任务
```

**为什么重要？**
- 单一职责的 Skill 更容易测试和维护
- Agent 更容易选择正确的 Skill
- 减少上下文窗口占用

### 2.2 明确边界原则

```markdown
# ✅ 明确的边界
SKILL: create-tech-spec
触发：用户请求创建技术文档
输入：需求描述、项目上下文
输出：结构化的技术规格文档
不做：代码实现、部署配置

# ❌ 模糊的边界
SKILL: help-with-project
触发：不知道
输入：随便
输出：随便
```

### 2.3 渐进披露原则

```markdown
# ✅ 渐进披露
1. 首先：理解任务目标
2. 然后：收集必要信息
3. 接着：执行核心逻辑
4. 最后：验证和输出

# ❌ 一次性披露
- 把所有步骤、规则、例外都放在一个长提示中
- Agent 难以理解和执行
```

### 2.4 可测试原则

```markdown
# ✅ 可测试的 Skill
触发条件：明确（包含"创建技术文档"）
执行步骤：可验证（检查是否包含所有章节）
输出格式：可验证（检查是否符合模板）

# ❌ 不可测试的 Skill
触发条件：模糊（"帮助项目"）
执行步骤：不可验证（"尽力而为"）
输出格式：不可验证（"好的结果"）
```

---

## 三、Skill 结构规范

### 3.1 标准 Skill 结构

```markdown
# Skill 名称
简短描述 Skill 的用途

## 触发条件
什么情况下激活这个 Skill

## 输入要求
- 必需参数
- 可选参数
- 参数验证规则

## 执行流程
1. 第一步：做什么
2. 第二步：做什么
3. 第三步：做什么

## 输出格式
- 输出结构
- 格式要求
- 质量标准

## 边界约束
- 能做什么
- 不能做什么
- 何时转交其他 Skill

## 错误处理
- 常见错误
- 处理策略
- 降级方案

## 示例
### 示例 1：标准场景
输入：...
输出：...

### 示例 2：边界场景
输入：...
输出：...
```

### 3.2 Claude Skills 格式

```markdown
---
name: create-tech-spec
version: 1.0.0
description: 创建技术规格文档
author: your-name
license: MIT
triggers:
  - "创建技术文档"
  - "写技术规格"
  - "technical specification"
inputs:
  required:
    - name: requirements
      type: string
      description: 需求描述
    - name: project_context
      type: string
      description: 项目上下文
  optional:
    - name: template
      type: string
      default: "standard"
      description: 文档模板
outputs:
  type: markdown
  structure:
    - section: 概述
    - section: 架构设计
    - section: 技术决策
    - section: 风险评估
---

# create-tech-spec Skill

## 目标
创建结构清晰、论证充分的技术规格文档。

## 执行流程

### 1. 需求分析
- 提取功能需求
- 识别非功能需求
- 明确约束条件

### 2. 架构设计
- 系统组件
- 数据流
- 接口定义

### 3. 技术决策
- 技术选型
- 决策理由
- 替代方案

### 4. 风险评估
- 技术风险
- 缓解措施
- 应急预案

## 输出模板

# [项目名称] 技术规格文档

## 1. 概述
[项目背景和目标]

## 2. 架构设计
[系统架构图和说明]

## 3. 技术决策
| 决策 | 选项 | 选择 | 理由 |
|------|------|------|------|
| ... | ... | ... | ... |

## 4. 风险评估
| 风险 | 概率 | 影响 | 缓解措施 |
|------|------|------|----------|
| ... | ... | ... | ... |

## 示例

### 输入
```
需求：创建一个用户管理系统
项目：电商平台
```

### 输出
[完整的技术规格文档]
```

### 3.3 MCP Tool 格式

```python
"""
MCP Tool - 天气查询
"""

from fastmcp import FastMCP

mcp = FastMCP("weather-tools")

@mcp.tool()
async def get_weather(
    city: str,
    include_forecast: bool = False
) -> str:
    """
    获取指定城市的天气信息。
    
    触发条件：用户询问天气
    输入：
        - city: 城市名称（必需）
        - include_forecast: 是否包含预报（可选，默认 False）
    输出：格式化的天气信息
    边界：仅支持国内主要城市
    
    Args:
        city: 城市名称
        include_forecast: 是否包含天气预报
        
    Returns:
        天气信息字符串
        
    Raises:
        ValueError: 城市不支持时
    """
    # 验证输入
    if not city or len(city) < 2:
        raise ValueError("城市名称无效")
    
    # 执行逻辑
    weather_data = await fetch_weather(city)
    
    # 格式化输出
    result = format_weather_output(weather_data, include_forecast)
    
    return result
```

### 3.4 LangChain Tool 格式

```python
"""
LangChain Tool - 数据分析
"""

from langchain.tools import tool
from pydantic import BaseModel, Field

class DataAnalysisInput(BaseModel):
    """数据分析输入模式"""
    dataset: str = Field(
        ...,
        description="数据集路径或 URL",
        min_length=1
    )
    analysis_type: str = Field(
        ...,
        description="分析类型",
        enum=["descriptive", "diagnostic", "predictive"]
    )
    output_format: str = Field(
        "markdown",
        description="输出格式",
        enum=["markdown", "json", "html"]
    )

@tool(args_schema=DataAnalysisInput)
def analyze_data(
    dataset: str,
    analysis_type: str,
    output_format: str = "markdown"
) -> str:
    """
    执行数据分析并生成报告。
    
    用途：分析数据集并生成洞察报告
    触发：用户请求数据分析
    输入：数据集路径、分析类型、输出格式
    输出：结构化的分析报告
    
    Args:
        dataset: 数据集路径
        analysis_type: 分析类型
        output_format: 输出格式
        
    Returns:
        分析报告字符串
    """
    # 1. 加载数据
    data = load_dataset(dataset)
    
    # 2. 执行分析
    if analysis_type == "descriptive":
        insights = descriptive_analysis(data)
    elif analysis_type == "diagnostic":
        insights = diagnostic_analysis(data)
    else:
        insights = predictive_analysis(data)
    
    # 3. 生成报告
    report = generate_report(insights, output_format)
    
    return report
```

---

## 四、Skill 开发流程

### 4.1 需求分析

**步骤 1：明确目标**
```markdown
问题清单：
- 这个 Skill 解决什么问题？
- 目标用户是谁？
- 成功标准是什么？
- 有哪些约束条件？
```

**步骤 2：场景分析**
```markdown
场景分类：
1. 标准场景（80% 使用情况）
2. 边界场景（15% 使用情况）
3. 异常场景（5% 使用情况）

为每个场景定义：
- 输入特征
- 期望输出
- 成功指标
```

### 4.2 设计阶段

**步骤 1：定义触发器**
```markdown
触发器类型：
1. 关键词触发："创建文档"、"分析数据"
2. 意图触发：用户表达特定意图
3. 上下文触发：基于对话上下文

最佳实践：
- 触发器要具体，避免误触发
- 支持多种表达方式
- 考虑同义词和变体
```

**步骤 2：设计执行流程**
```markdown
流程图示例：

开始
  ↓
验证输入 → [无效] → 返回错误
  ↓ [有效]
收集上下文
  ↓
执行核心逻辑
  ↓
验证输出 → [不合格] → 重新执行
  ↓ [合格]
格式化输出
  ↓
结束
```

**步骤 3：定义输入输出**
```markdown
输入规范：
- 必需参数：[名称、类型、验证规则]
- 可选参数：[名称、类型、默认值]
- 参数依赖：[参数间关系]

输出规范：
- 输出结构：[章节、字段、格式]
- 质量标准：[完整性、准确性]
- 错误格式：[错误码、消息、建议]
```

### 4.3 实现阶段

**Python 实现示例**

```python
"""
Skill: create-tech-spec
版本：1.0.0
"""

from typing import Optional, List
from pydantic import BaseModel, Field, validator
from enum import Enum


# ============== 数据模型 ==============

class TechSpecSection(str, Enum):
    """技术文档章节"""
    OVERVIEW = "overview"
    ARCHITECTURE = "architecture"
    DECISIONS = "decisions"
    RISKS = "risks"


class TechSpecInput(BaseModel):
    """技术规格输入模型"""
    
    requirements: str = Field(
        ...,
        description="需求描述",
        min_length=10,
        max_length=10000
    )
    
    project_context: str = Field(
        ...,
        description="项目上下文",
        min_length=10
    )
    
    template: str = Field(
        default="standard",
        description="文档模板",
        enum=["standard", "minimal", "detailed"]
    )
    
    @validator('requirements')
    def validate_requirements(cls, v):
        if not v.strip():
            raise ValueError("需求描述不能为空")
        return v.strip()


class TechSpecOutput(BaseModel):
    """技术规格输出模型"""
    
    title: str
    sections: dict
    decisions: List[dict]
    risks: List[dict]
    markdown: str


# ============== Skill 实现 ==============

class CreateTechSpecSkill:
    """创建技术规格文档 Skill"""
    
    name = "create-tech-spec"
    version = "1.0.0"
    description = "创建结构化的技术规格文档"
    
    def __init__(self):
        self.template = self._load_template()
    
    def _load_template(self) -> dict:
        """加载文档模板"""
        return {
            "standard": {
                "sections": ["概述", "架构", "决策", "风险"],
                "min_decisions": 3,
                "min_risks": 2
            },
            "minimal": {
                "sections": ["概述", "决策"],
                "min_decisions": 1,
                "min_risks": 0
            },
            "detailed": {
                "sections": ["概述", "架构", "决策", "风险", "附录"],
                "min_decisions": 5,
                "min_risks": 3
            }
        }
    
    def execute(self, input_data: TechSpecInput) -> TechSpecOutput:
        """
        执行 Skill
        
        流程：
        1. 需求分析
        2. 架构设计
        3. 技术决策
        4. 风险评估
        5. 生成文档
        """
        
        # 1. 需求分析
        requirements = self._analyze_requirements(input_data.requirements)
        
        # 2. 架构设计
        architecture = self._design_architecture(requirements, input_data.project_context)
        
        # 3. 技术决策
        decisions = self._make_decisions(architecture, requirements)
        
        # 4. 风险评估
        risks = self._assess_risks(decisions)
        
        # 5. 生成文档
        markdown = self._generate_markdown(
            title=requirements["title"],
            sections={
                "概述": requirements["overview"],
                "架构": architecture,
                "决策": decisions,
                "风险": risks
            },
            template=input_data.template
        )
        
        return TechSpecOutput(
            title=requirements["title"],
            sections=requirements,
            decisions=decisions,
            risks=risks,
            markdown=markdown
        )
    
    def _analyze_requirements(self, requirements: str) -> dict:
        """分析需求"""
        # 实现需求分析逻辑
        return {
            "title": "项目技术规格",
            "overview": "项目概述...",
            "functional": [...],
            "non_functional": [...]
        }
    
    def _design_architecture(self, requirements: dict, context: str) -> str:
        """设计架构"""
        # 实现架构设计逻辑
        return "系统架构描述..."
    
    def _make_decisions(self, architecture: str, requirements: dict) -> List[dict]:
        """制定技术决策"""
        # 实现决策逻辑
        return [
            {
                "decision": "技术选型",
                "options": ["选项 A", "选项 B"],
                "choice": "选项 A",
                "rationale": "选择理由..."
            }
        ]
    
    def _assess_risks(self, decisions: List[dict]) -> List[dict]:
        """评估风险"""
        # 实现风险评估逻辑
        return [
            {
                "risk": "技术风险",
                "probability": "中",
                "impact": "高",
                "mitigation": "缓解措施..."
            }
        ]
    
    def _generate_markdown(
        self,
        title: str,
        sections: dict,
        template: str
    ) -> str:
        """生成 Markdown 文档"""
        template_config = self.template[template]
        
        markdown = f"# {title}\n\n"
        
        for section_name in template_config["sections"]:
            if section_name in sections:
                markdown += f"## {section_name}\n\n"
                markdown += sections[section_name]
                markdown += "\n\n"
        
        return markdown


# ============== 使用示例 ==============

if __name__ == "__main__":
    skill = CreateTechSpecSkill()
    
    input_data = TechSpecInput(
        requirements="创建一个用户管理系统，支持注册、登录、权限管理",
        project_context="电商平台，预计日活 10 万",
        template="standard"
    )
    
    output = skill.execute(input_data)
    print(output.markdown)
```

### 4.4 测试阶段

**单元测试**

```python
import pytest
from skill import CreateTechSpecSkill, TechSpecInput


class TestCreateTechSpecSkill:
    """Skill 单元测试"""
    
    @pytest.fixture
    def skill(self):
        return CreateTechSpecSkill()
    
    def test_valid_input(self, skill):
        """测试有效输入"""
        input_data = TechSpecInput(
            requirements="创建用户管理系统",
            project_context="电商平台",
            template="standard"
        )
        
        output = skill.execute(input_data)
        
        assert output.title is not None
        assert len(output.decisions) >= 3
        assert len(output.risks) >= 2
        assert "## 概述" in output.markdown
    
    def test_invalid_requirements(self, skill):
        """测试无效需求"""
        with pytest.raises(ValueError):
            TechSpecInput(
                requirements="",  # 空需求
                project_context="测试"
            )
    
    def test_minimal_template(self, skill):
        """测试最小模板"""
        input_data = TechSpecInput(
            requirements="简单需求",
            project_context="测试",
            template="minimal"
        )
        
        output = skill.execute(input_data)
        
        assert "## 概述" in output.markdown
        assert "## 风险" not in output.markdown  # 最小模板不包含风险
    
    def test_output_format(self, skill):
        """测试输出格式"""
        input_data = TechSpecInput(
            requirements="测试需求",
            project_context="测试项目"
        )
        
        output = skill.execute(input_data)
        
        # 验证 Markdown 格式
        assert output.markdown.startswith("#")
        assert "## " in output.markdown
        assert "| " in output.markdown  # 包含表格


# 集成测试
class TestSkillIntegration:
    """Skill 集成测试"""
    
    def test_end_to_end(self):
        """端到端测试"""
        skill = CreateTechSpecSkill()
        
        # 模拟真实使用场景
        input_data = TechSpecInput(
            requirements="创建 REST API，支持用户 CRUD 操作",
            project_context="SaaS 平台，多租户架构",
            template="detailed"
        )
        
        output = skill.execute(input_data)
        
        # 验证完整输出
        assert len(output.markdown) > 1000
        assert len(output.decisions) >= 5
        assert len(output.risks) >= 3
```

**测试覆盖率要求**

```markdown
覆盖率目标：
- 行覆盖率：≥ 80%
- 分支覆盖率：≥ 70%
- 关键路径：100%

测试类型：
1. 单元测试：验证单个函数
2. 集成测试：验证模块间交互
3. 端到端测试：验证完整流程
4. 边界测试：验证边界条件
5. 异常测试：验证错误处理
```

---

## 五、高级 Skill 模式

### 5.1 组合模式

```python
"""
Skill 组合：将多个简单 Skill 组合成复杂 Skill
"""

from typing import List


class SkillComposer:
    """Skill 组合器"""
    
    def __init__(self):
        self.skills = {}
    
    def register(self, name: str, skill):
        """注册 Skill"""
        self.skills[name] = skill
    
    def compose(self, skill_names: List[str]) -> "ComposedSkill":
        """组合多个 Skill"""
        return ComposedSkill([self.skills[name] for name in skill_names])


class ComposedSkill:
    """组合 Skill"""
    
    def __init__(self, skills: List):
        self.skills = skills
    
    def execute(self, input_data):
        """
        执行组合 Skill
        
        流程：
        1. 按顺序执行每个 Skill
        2. 传递上一个 Skill 的输出
        3. 返回最终输出
        """
        current_data = input_data
        
        for skill in self.skills:
            current_data = skill.execute(current_data)
        
        return current_data


# 使用示例
composer = SkillComposer()

# 注册基础 Skill
composer.register("analyze", DataAnalysisSkill())
composer.register("visualize", DataVisualizationSkill())
composer.register("report", ReportGenerationSkill())

# 组合成复杂 Skill
analysis_pipeline = composer.compose(["analyze", "visualize", "report"])

# 执行
result = analysis_pipeline.execute(dataset)
```

### 5.2 条件模式

```python
"""
条件 Skill：根据条件选择执行路径
"""

from enum import Enum


class SkillCondition(str, Enum):
    """Skill 条件"""
    ALWAYS = "always"
    IF_DATA_AVAILABLE = "if_data_available"
    IF_USER_CONFIRMS = "if_user_confirms"
    NEVER = "never"


class ConditionalSkill:
    """条件 Skill"""
    
    def __init__(self, skill, condition: SkillCondition):
        self.skill = skill
        self.condition = condition
    
    def should_execute(self, context: dict) -> bool:
        """判断是否执行"""
        if self.condition == SkillCondition.ALWAYS:
            return True
        elif self.condition == SkillCondition.NEVER:
            return False
        elif self.condition == SkillCondition.IF_DATA_AVAILABLE:
            return self._check_data_available(context)
        elif self.condition == SkillCondition.IF_USER_CONFIRMS:
            return self._get_user_confirmation(context)
        return False
    
    def execute(self, input_data, context: dict):
        """执行条件 Skill"""
        if not self.should_execute(context):
            return None
        
        return self.skill.execute(input_data)
    
    def _check_data_available(self, context: dict) -> bool:
        """检查数据是否可用"""
        return "data" in context
    
    def _get_user_confirmation(self, context: dict) -> bool:
        """获取用户确认"""
        # 实现用户确认逻辑
        return context.get("user_confirmed", False)


# 使用示例
conditional_risk_assessment = ConditionalSkill(
    skill=RiskAssessmentSkill(),
    condition=SkillCondition.IF_DATA_AVAILABLE
)

# 只有在数据可用时才执行风险评估
result = conditional_risk_assessment.execute(data, context)
```

### 5.3 降级模式

```python
"""
降级 Skill：在主 Skill 失败时使用备用方案
"""

import logging
from typing import Optional


class FallbackSkill:
    """降级 Skill"""
    
    def __init__(self, primary_skill, fallback_skill: Optional = None):
        self.primary_skill = primary_skill
        self.fallback_skill = fallback_skill
        self.max_retries = 3
    
    def execute(self, input_data, **kwargs):
        """
        执行降级 Skill
        
        流程：
        1. 尝试执行主 Skill
        2. 失败时重试
        3. 仍失败则使用降级 Skill
        """
        last_error = None
        
        # 尝试主 Skill
        for attempt in range(self.max_retries):
            try:
                return self.primary_skill.execute(input_data, **kwargs)
            except Exception as e:
                last_error = e
                logging.warning(
                    f"主 Skill 执行失败 (尝试 {attempt + 1}/{self.max_retries}): {e}"
                )
        
        # 使用降级 Skill
        if self.fallback_skill:
            logging.info("使用降级 Skill")
            try:
                return self.fallback_skill.execute(input_data, **kwargs)
            except Exception as e:
                logging.error(f"降级 Skill 也失败：{e}")
                raise
        
        # 没有降级 Skill，抛出原始错误
        raise last_error


# 使用示例
# 主 Skill：高级分析（需要外部 API）
advanced_analysis = AdvancedAnalysisSkill()

# 降级 Skill：基础分析（本地计算）
basic_analysis = BasicAnalysisSkill()

# 组合
robust_analysis = FallbackSkill(
    primary_skill=advanced_analysis,
    fallback_skill=basic_analysis
)

# 即使外部 API 失败，也能返回基础分析结果
result = robust_analysis.execute(data)
```

### 5.4 缓存模式

```python
"""
缓存 Skill：缓存结果以提高性能
"""

import hashlib
import json
from datetime import datetime, timedelta
from typing import Any, Optional


class CacheEntry(BaseModel):
    """缓存条目"""
    result: Any
    created_at: datetime
    ttl: int  # 生存时间（秒）
    
    def is_expired(self) -> bool:
        return datetime.now() > self.created_at + timedelta(seconds=self.ttl)


class CachedSkill:
    """缓存 Skill"""
    
    def __init__(self, skill, cache_ttl: int = 3600):
        self.skill = skill
        self.cache_ttl = cache_ttl
        self.cache: dict = {}
    
    def _generate_cache_key(self, input_data: dict) -> str:
        """生成缓存键"""
        # 序列化输入
        input_str = json.dumps(input_data, sort_keys=True, default=str)
        # 生成哈希
        return hashlib.sha256(input_str.encode()).hexdigest()
    
    def execute(self, input_data, **kwargs):
        """
        执行缓存 Skill
        
        流程：
        1. 检查缓存
        2. 命中则返回缓存结果
        3. 未命中则执行 Skill 并缓存
        """
        # 生成缓存键
        cache_key = self._generate_cache_key(input_data)
        
        # 检查缓存
        if cache_key in self.cache:
            entry = self.cache[cache_key]
            
            if not entry.is_expired():
                logging.debug(f"缓存命中：{cache_key}")
                return entry.result
            else:
                # 清除过期缓存
                del self.cache[cache_key]
        
        # 执行 Skill
        logging.debug(f"缓存未命中，执行 Skill: {cache_key}")
        result = self.skill.execute(input_data, **kwargs)
        
        # 缓存结果
        self.cache[cache_key] = CacheEntry(
            result=result,
            created_at=datetime.now(),
            ttl=self.cache_ttl
        )
        
        return result
    
    def clear_cache(self):
        """清除所有缓存"""
        self.cache.clear()
    
    def get_cache_stats(self) -> dict:
        """获取缓存统计"""
        total = len(self.cache)
        expired = sum(1 for entry in self.cache.values() if entry.is_expired())
        
        return {
            "total_entries": total,
            "expired_entries": expired,
            "valid_entries": total - expired
        }


# 使用示例
# 原始 Skill
weather_skill = GetWeatherSkill()

# 包装为缓存 Skill（缓存 1 小时）
cached_weather_skill = CachedSkill(
    skill=weather_skill,
    cache_ttl=3600
)

# 第一次调用：执行 Skill
result1 = cached_weather_skill.execute({"city": "北京"})

# 第二次调用（相同参数）：返回缓存
result2 = cached_weather_skill.execute({"city": "北京"})

# 查看缓存统计
stats = cached_weather_skill.get_cache_stats()
print(stats)
```

---

## 六、Skill 最佳实践

### 6.1 命名规范

```markdown
# ✅ 好的命名
- `create-tech-spec`: 动词 + 名词，清晰表达功能
- `analyze-data`: 简洁明确
- `review-code-security`: 包含领域信息

# ❌ 糟糕的命名
- `do-it`: 太模糊
- `skill-1`: 无意义
- `the_thing_that_does_stuff`: 太长
```

### 6.2 文档规范

```markdown
# Skill 文档模板

## 概述
一句话描述 Skill 的用途。

## 触发条件
列出所有可能的触发方式：
- 关键词：["创建文档", "写规格"]
- 意图：用户想要创建技术文档
- 上下文：在项目讨论中

## 输入参数
| 参数 | 类型 | 必需 | 默认值 | 描述 |
|------|------|------|--------|------|
| requirements | string | 是 | - | 需求描述 |
| template | string | 否 | standard | 文档模板 |

## 输出格式
描述输出的结构和格式。

## 使用示例
### 示例 1
输入：...
输出：...

## 限制和约束
- 不支持的功能
- 已知的限制
- 性能考虑

## 错误处理
| 错误 | 原因 | 处理方案 |
|------|------|----------|
| 需求为空 | 输入验证失败 | 提示用户提供需求 |
| 模板不存在 | 配置错误 | 使用默认模板 |
```

### 6.3 性能优化

```python
# ✅ 优化实践

# 1. 懒加载
class LazySkill:
    def __init__(self):
        self._expensive_resource = None
    
    @property
    def expensive_resource(self):
        if self._expensive_resource is None:
            self._expensive_resource = self._load_resource()
        return self._expensive_resource

# 2. 批量处理
class BatchSkill:
    def execute_batch(self, items: List[dict], batch_size: int = 10):
        """批量处理，减少 API 调用"""
        results = []
        for i in range(0, len(items), batch_size):
            batch = items[i:i + batch_size]
            batch_result = self._process_batch(batch)
            results.extend(batch_result)
        return results

# 3. 异步处理
class AsyncSkill:
    async def execute(self, input_data):
        """异步执行，不阻塞"""
        result = await self._async_process(input_data)
        return result

# ❌ 避免的做法

# 1. 重复加载资源
class BadSkill:
    def method1(self):
        resource = self._load_resource()  # 每次都加载
    
    def method2(self):
        resource = self._load_resource()  # 重复加载

# 2. 同步阻塞
class BlockingSkill:
    def execute(self, input_data):
        # 阻塞 10 秒
        time.sleep(10)
        return result
```

### 6.4 安全考虑

```python
# ✅ 安全实践

# 1. 输入验证
class SecureSkill:
    def execute(self, input_data: dict):
        # 验证所有输入
        if not self._validate_input(input_data):
            raise ValueError("输入验证失败")
        
        # 清理输入
        cleaned_data = self._sanitize_input(input_data)
        
        # 执行逻辑
        return self._process(cleaned_data)
    
    def _validate_input(self, data: dict) -> bool:
        """验证输入"""
        # 检查必需字段
        required_fields = ["field1", "field2"]
        for field in required_fields:
            if field not in data:
                return False
        
        # 检查类型
        if not isinstance(data.get("field1"), str):
            return False
        
        # 检查长度
        if len(data.get("field1", "")) > 1000:
            return False
        
        return True
    
    def _sanitize_input(self, data: dict) -> dict:
        """清理输入"""
        cleaned = {}
        for key, value in data.items():
            if isinstance(value, str):
                # 移除危险字符
                cleaned[key] = self._escape_special_chars(value)
            else:
                cleaned[key] = value
        return cleaned

# 2. 权限检查
class AuthorizedSkill:
    def execute(self, input_data, user_context: dict):
        # 检查权限
        if not self._check_permission(user_context):
            raise PermissionError("无权执行此操作")
        
        return self._process(input_data)

# 3. 敏感信息处理
class SecureDataSkill:
    def execute(self, input_data):
        # 不记录敏感信息
        sensitive_fields = ["password", "api_key", "token"]
        for field in sensitive_fields:
            if field in input_data:
                input_data[field] = "***REDACTED***"
        
        # 安全处理
        return self._process(input_data)
```

---

## 七、Skill 管理

### 7.1 版本控制

```markdown
# Skill 版本规范

版本格式：MAJOR.MINOR.PATCH

## 版本号规则
- MAJOR: 不兼容的变更
- MINOR: 向后兼容的功能新增
- PATCH: 向后兼容的问题修复

## 示例
- 1.0.0: 初始发布
- 1.0.1: 修复 bug
- 1.1.0: 新增功能
- 2.0.0: 不兼容的 API 变更

## 变更日志
每个版本应包含变更日志：

# v1.1.0
## 新增
- 支持新的文档模板
- 添加风险评估矩阵

## 修复
- 修复 Markdown 格式问题
- 优化性能

## 变更
- 更新依赖版本
```

### 7.2 依赖管理

```python
# requirements.txt 示例
# Skill 依赖

# 核心依赖
pydantic>=2.0.0
fastmcp>=0.1.0

# 可选依赖
redis>=4.0.0  # 缓存支持
sqlalchemy>=2.0.0  # 数据库支持

# 开发依赖
pytest>=7.0.0
pytest-cov>=4.0.0
black>=23.0.0
```

### 7.3 注册和发现

```python
"""
Skill 注册中心
"""

from typing import Dict, List, Optional


class SkillRegistry:
    """Skill 注册中心"""
    
    def __init__(self):
        self.skills: Dict[str, dict] = {}
    
    def register(
        self,
        name: str,
        skill_class,
        version: str,
        description: str,
        tags: List[str] = None
    ):
        """注册 Skill"""
        self.skills[name] = {
            "class": skill_class,
            "version": version,
            "description": description,
            "tags": tags or [],
            "enabled": True
        }
    
    def get(self, name: str) -> Optional[object]:
        """获取 Skill 实例"""
        if name not in self.skills:
            return None
        
        skill_info = self.skills[name]
        if not skill_info["enabled"]:
            return None
        
        return skill_info["class"]()
    
    def search(self, query: str) -> List[dict]:
        """搜索 Skill"""
        results = []
        query_lower = query.lower()
        
        for name, info in self.skills.items():
            # 搜索名称、描述、标签
            if (query_lower in name.lower() or
                query_lower in info["description"].lower() or
                any(query_lower in tag.lower() for tag in info["tags"])):
                results.append({
                    "name": name,
                    "version": info["version"],
                    "description": info["description"],
                    "tags": info["tags"]
                })
        
        return results
    
    def list_all(self) -> List[dict]:
        """列出所有 Skill"""
        return [
            {
                "name": name,
                "version": info["version"],
                "description": info["description"],
                "tags": info["tags"],
                "enabled": info["enabled"]
            }
            for name, info in self.skills.items()
        ]
    
    def enable(self, name: str):
        """启用 Skill"""
        if name in self.skills:
            self.skills[name]["enabled"] = True
    
    def disable(self, name: str):
        """禁用 Skill"""
        if name in self.skills:
            self.skills[name]["enabled"] = False


# 使用示例
registry = SkillRegistry()

# 注册 Skills
registry.register(
    name="create-tech-spec",
    skill_class=CreateTechSpecSkill,
    version="1.0.0",
    description="创建技术规格文档",
    tags=["documentation", "technical"]
)

registry.register(
    name="analyze-data",
    skill_class=DataAnalysisSkill,
    version="2.0.0",
    description="数据分析",
    tags=["analysis", "data"]
)

# 搜索 Skills
results = registry.search("文档")
print(results)

# 获取 Skill 实例
skill = registry.get("create-tech-spec")
if skill:
    result = skill.execute(input_data)
```

---

## 八、实战案例

### 8.1 代码审查 Skill

```python
"""
Skill: code-review
代码审查 Skill
"""

from typing import List, Dict
from pydantic import BaseModel, Field
from enum import Enum


class Severity(str, Enum):
    """问题严重程度"""
    INFO = "info"
    WARNING = "warning"
    ERROR = "error"
    CRITICAL = "critical"


class CodeIssue(BaseModel):
    """代码问题"""
    line: int
    column: int
    severity: Severity
    message: str
    suggestion: str
    rule_id: str


class CodeReviewInput(BaseModel):
    """代码审查输入"""
    code: str = Field(..., description="待审查的代码")
    language: str = Field(default="python", description="编程语言")
    review_focus: List[str] = Field(
        default=["security", "performance"],
        description="审查重点"
    )


class CodeReviewOutput(BaseModel):
    """代码审查输出"""
    issues: List[CodeIssue]
    summary: str
    score: int  # 0-100
    approved: bool


class CodeReviewSkill:
    """代码审查 Skill"""
    
    name = "code-review"
    version = "1.0.0"
    
    # 审查规则
    RULES = {
        "security": [
            {"id": "SEC001", "pattern": "eval(", "severity": Severity.CRITICAL},
            {"id": "SEC002", "pattern": "exec(", "severity": Severity.CRITICAL},
            {"id": "SEC003", "pattern": "os.system(", "severity": Severity.ERROR},
        ],
        "performance": [
            {"id": "PERF001", "pattern": "for.*in.*range\\(len\\(", "severity": Severity.WARNING},
        ],
        "style": [
            {"id": "STYLE001", "pattern": "^\\s{4}", "severity": Severity.INFO},
        ]
    }
    
    def execute(self, input_data: CodeReviewInput) -> CodeReviewOutput:
        """执行代码审查"""
        issues = []
        
        # 1. 安全检查
        if "security" in input_data.review_focus:
            issues.extend(self._check_security(input_data.code))
        
        # 2. 性能检查
        if "performance" in input_data.review_focus:
            issues.extend(self._check_performance(input_data.code))
        
        # 3. 风格检查
        if "style" in input_data.review_focus:
            issues.extend(self._check_style(input_data.code))
        
        # 4. 计算分数
        score = self._calculate_score(issues)
        
        # 5. 生成总结
        summary = self._generate_summary(issues, score)
        
        return CodeReviewOutput(
            issues=issues,
            summary=summary,
            score=score,
            approved=score >= 80 and not any(i.severity == Severity.CRITICAL for i in issues)
        )
    
    def _check_security(self, code: str) -> List[CodeIssue]:
        """安全检查"""
        issues = []
        
        for rule in self.RULES["security"]:
            lines = code.split("\n")
            for line_num, line in enumerate(lines, 1):
                if rule["pattern"] in line:
                    issues.append(CodeIssue(
                        line=line_num,
                        column=line.index(rule["pattern"]),
                        severity=rule["severity"],
                        message=f"发现安全问题：{rule['pattern']}",
                        suggestion="使用更安全的替代方案",
                        rule_id=rule["id"]
                    ))
        
        return issues
    
    def _check_performance(self, code: str) -> List[CodeIssue]:
        """性能检查"""
        import re
        issues = []
        
        for rule in self.RULES["performance"]:
            for match in re.finditer(rule["pattern"], code):
                line_num = code[:match.start()].count("\n") + 1
                issues.append(CodeIssue(
                    line=line_num,
                    column=match.start(),
                    severity=rule["severity"],
                    message="性能问题：使用 enumerate 代替 range(len())",
                    suggestion="for i, item in enumerate(items)",
                    rule_id=rule["id"]
                ))
        
        return issues
    
    def _check_style(self, code: str) -> List[CodeIssue]:
        """风格检查"""
        issues = []
        # 实现风格检查逻辑
        return issues
    
    def _calculate_score(self, issues: List[CodeIssue]) -> int:
        """计算代码分数"""
        score = 100
        
        for issue in issues:
            if issue.severity == Severity.CRITICAL:
                score -= 20
            elif issue.severity == Severity.ERROR:
                score -= 10
            elif issue.severity == Severity.WARNING:
                score -= 5
            else:
                score -= 1
        
        return max(0, min(100, score))
    
    def _generate_summary(self, issues: List[CodeIssue], score: int) -> str:
        """生成审查总结"""
        critical = sum(1 for i in issues if i.severity == Severity.CRITICAL)
        errors = sum(1 for i in issues if i.severity == Severity.ERROR)
        warnings = sum(1 for i in issues if i.severity == Severity.WARNING)
        
        summary = f"代码审查完成，得分：{score}/100\n"
        summary += f"发现 {critical} 个严重问题，{errors} 个错误，{warnings} 个警告\n"
        
        if score >= 90:
            summary += "评价：优秀 ✅"
        elif score >= 80:
            summary += "评价：良好 ✓"
        elif score >= 60:
            summary += "评价：需要改进 ⚠️"
        else:
            summary += "评价：需要重大重构 ❌"
        
        return summary


# 使用示例
if __name__ == "__main__":
    skill = CodeReviewSkill()
    
    code = """
def process_data(items):
    for i in range(len(items)):
        print(items[i])
    
    eval(user_input)
    """
    
    input_data = CodeReviewInput(
        code=code,
        language="python",
        review_focus=["security", "performance", "style"]
    )
    
    result = skill.execute(input_data)
    
    print(f"得分：{result.score}")
    print(f"审查结果：{result.summary}")
    print(f"问题列表:")
    for issue in result.issues:
        print(f"  - 行{issue.line}: {issue.message}")
```

### 8.2 数据分析 Skill

```python
"""
Skill: data-analysis
数据分析 Skill
"""

import pandas as pd
import numpy as np
from typing import Optional, Dict, Any
from pydantic import BaseModel, Field


class DataAnalysisInput(BaseModel):
    """数据分析输入"""
    data_source: str = Field(..., description="数据源路径或 URL")
    analysis_type: str = Field(
        ...,
        enum=["descriptive", "diagnostic", "predictive"],
        description="分析类型"
    )
    target_column: Optional[str] = Field(None, description="目标列（预测分析需要）")


class DataAnalysisOutput(BaseModel):
    """数据分析输出"""
    summary: Dict[str, Any]
    insights: list
    visualizations: list
    recommendations: list


class DataAnalysisSkill:
    """数据分析 Skill"""
    
    def execute(self, input_data: DataAnalysisInput) -> DataAnalysisOutput:
        """执行数据分析"""
        # 1. 加载数据
        df = self._load_data(input_data.data_source)
        
        # 2. 执行分析
        if input_data.analysis_type == "descriptive":
            insights = self._descriptive_analysis(df)
        elif input_data.analysis_type == "diagnostic":
            insights = self._diagnostic_analysis(df)
        else:
            insights = self._predictive_analysis(df, input_data.target_column)
        
        # 3. 生成总结
        summary = self._generate_summary(df)
        
        # 4. 生成建议
        recommendations = self._generate_recommendations(insights)
        
        return DataAnalysisOutput(
            summary=summary,
            insights=insights,
            visualizations=[],
            recommendations=recommendations
        )
    
    def _load_data(self, data_source: str) -> pd.DataFrame:
        """加载数据"""
        if data_source.endswith(".csv"):
            return pd.read_csv(data_source)
        elif data_source.endswith(".parquet"):
            return pd.read_parquet(data_source)
        else:
            raise ValueError("不支持的数据格式")
    
    def _descriptive_analysis(self, df: pd.DataFrame) -> list:
        """描述性分析"""
        insights = []
        
        # 基本统计
        insights.append({
            "type": "statistics",
            "title": "基本统计",
            "content": df.describe().to_dict()
        })
        
        # 缺失值分析
        missing = df.isnull().sum()
        if missing.any():
            insights.append({
                "type": "missing_values",
                "title": "缺失值分析",
                "content": missing[missing > 0].to_dict()
            })
        
        # 相关性分析
        numeric_df = df.select_dtypes(include=[np.number])
        if len(numeric_df.columns) > 1:
            correlations = numeric_df.corr()
            insights.append({
                "type": "correlations",
                "title": "相关性分析",
                "content": correlations.to_dict()
            })
        
        return insights
    
    def _diagnostic_analysis(self, df: pd.DataFrame) -> list:
        """诊断性分析"""
        insights = []
        
        # 异常值检测
        for col in df.select_dtypes(include=[np.number]).columns:
            q1 = df[col].quantile(0.25)
            q3 = df[col].quantile(0.75)
            iqr = q3 - q1
            
            outliers = df[
                (df[col] < q1 - 1.5 * iqr) | 
                (df[col] > q3 + 1.5 * iqr)
            ]
            
            if len(outliers) > 0:
                insights.append({
                    "type": "outliers",
                    "title": f"{col} 异常值",
                    "content": {
                        "count": len(outliers),
                        "percentage": len(outliers) / len(df) * 100
                    }
                })
        
        return insights
    
    def _predictive_analysis(
        self,
        df: pd.DataFrame,
        target_column: Optional[str]
    ) -> list:
        """预测性分析"""
        insights = []
        
        if target_column and target_column in df.columns:
            # 特征重要性分析（简化版）
            numeric_df = df.select_dtypes(include=[np.number])
            if target_column in numeric_df.columns:
                correlations = numeric_df[target_column].corr(numeric_df.drop(columns=[target_column]))
                insights.append({
                    "type": "feature_importance",
                    "title": "特征重要性",
                    "content": correlations.abs().sort_values(ascending=False).to_dict()
                })
        
        return insights
    
    def _generate_summary(self, df: pd.DataFrame) -> dict:
        """生成数据总结"""
        return {
            "rows": len(df),
            "columns": len(df.columns),
            "memory_usage": df.memory_usage(deep=True).sum() / 1024 ** 2,  # MB
            "dtypes": df.dtypes.astype(str).to_dict()
        }
    
    def _generate_recommendations(self, insights: list) -> list:
        """生成建议"""
        recommendations = []
        
        for insight in insights:
            if insight["type"] == "missing_values":
                recommendations.append({
                    "priority": "high",
                    "action": "处理缺失值",
                    "details": "建议使用插值或删除缺失值"
                })
            elif insight["type"] == "outliers":
                recommendations.append({
                    "priority": "medium",
                    "action": "处理异常值",
                    "details": "建议检查异常值原因并决定处理方式"
                })
        
        return recommendations
```

---

## 九、总结

### 9.1 Skill 开发检查清单

```markdown
## 设计阶段
- [ ] 明确 Skill 的目标和边界
- [ ] 定义清晰的触发条件
- [ ] 设计输入输出格式
- [ ] 规划执行流程

## 实现阶段
- [ ] 实现输入验证
- [ ] 实现核心逻辑
- [ ] 实现错误处理
- [ ] 添加日志记录

## 测试阶段
- [ ] 编写单元测试
- [ ] 编写集成测试
- [ ] 测试边界条件
- [ ] 测试异常场景

## 部署阶段
- [ ] 编写文档
- [ ] 注册到 Skill 中心
- [ ] 配置监控
- [ ] 制定更新计划
```

### 9.2 学习资源

- **官方文档**：
  - Claude Skills: https://docs.anthropic.com/claude/docs/skills
  - MCP: https://modelcontextprotocol.io/
  - LangChain Tools: https://python.langchain.com/docs/modules/tools/

- **示例仓库**：
  - https://github.com/anthropics/skills
  - https://github.com/modelcontextprotocol/servers
  - https://github.com/HKUDS/nanobot

- **社区资源**：
  - https://skillsmp.com/
  - https://mcpmarket.com/

### 9.3 下一步

1. **从简单开始**：先实现一个简单 Skill，逐步扩展
2. **参考示例**：学习官方和社区的优秀 Skill
3. **加入社区**：参与讨论和贡献
4. **持续优化**：根据反馈不断改进 Skill

掌握 Skill 开发，你将能够构建强大、可靠的 AI Agent，让 AI 真正成为你的得力助手。开始构建你的第一个 Skill 吧！🚀
