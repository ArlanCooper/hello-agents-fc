# Hello Agents FC

一个功能强大、易于使用的AI Agent框架，支持多种Agent实现模式和工具调用能力。

## 📋 目录

- [特性](#特性)
- [快速开始](#快速开始)
- [安装](#安装)
- [核心概念](#核心概念)
- [Agent类型](#agent类型)
- [工具系统](#工具系统)
- [LLM提供商支持](#llm提供商支持)
- [使用示例](#使用示例)
- [项目结构](#项目结构)
- [配置说明](#配置说明)
- [贡献指南](#贡献指南)

## ✨ 特性

- **多种Agent实现**：支持SimpleAgent、FunctionCallAgent、ReActAgent、PlanSolveAgent、ReflectionAgent等多种Agent模式
- **统一LLM接口**：兼容OpenAI API，支持多种LLM提供商（OpenAI、DeepSeek、Qwen、ModelScope、Kimi、Zhipu、Ollama、vLLM等）
- **强大的工具系统**：支持工具注册、管理和执行，内置工具展开机制
- **函数调用支持**：原生支持OpenAI函数调用（Function Calling）范式
- **流式响应**：默认支持流式输出，提供更好的用户体验
- **灵活配置**：支持环境变量和代码配置两种方式
- **类型安全**：基于Pydantic的类型验证和配置管理

## 🚀 快速开始

### 基本使用

```python
from hello_agents_fc.core.llm import HelloAgentsLLM
from hello_agents_fc.agents.simple_agent import SimpleAgent

# 初始化LLM（使用环境变量配置）
llm = HelloAgentsLLM(
    provider="openai",  # 或 "deepseek", "qwen" 等
    model="gpt-3.5-turbo"
)

# 创建Agent
agent = SimpleAgent(
    name="助手",
    llm=llm,
    system_prompt="你是一个有用的AI助手。"
)

# 运行Agent
response = agent.run("你好，请介绍一下你自己")
print(response)
```

### 使用工具

```python
from hello_agents_fc.tools.registry import ToolRegistry
from hello_agents_fc.tools.builtin.calculator import CalculatorTool

# 创建工具注册表
tool_registry = ToolRegistry()

# 注册工具
calculator = CalculatorTool()
tool_registry.register_tool(calculator)

# 创建带工具的Agent
agent = SimpleAgent(
    name="计算助手",
    llm=llm,
    tool_registry=tool_registry,
    enable_tool_calling=True
)

# Agent会自动调用工具
response = agent.run("请计算 123 * 456 的结果")
print(response)
```

## 📦 安装

### 环境要求

- Python 3.8+
- pip

### 安装步骤

1. 克隆项目：
```bash
git clone <repository-url>
cd hello-agents-fc
```

2. 安装依赖：
```bash
pip install -r requirements.txt
```

3. 配置环境变量（可选）：
```bash
# 创建 .env 文件
export OPENAI_API_KEY="your-api-key"
export LLM_MODEL_ID="gpt-3.5-turbo"
export LLM_BASE_URL="https://api.openai.com/v1"
```

## 🧠 核心概念

### Agent

Agent是框架的核心抽象，代表一个能够理解用户输入、调用工具、生成响应的智能体。所有Agent都继承自`Agent`基类，必须实现`run`方法。

### LLM

`HelloAgentsLLM`提供了统一的LLM接口，支持多种提供商。它会自动检测配置并选择合适的API端点。

### Tool

工具是Agent可以调用的外部功能。工具可以是：
- **普通工具**：单一功能的工具
- **可展开工具**：可以展开为多个子工具的工具（使用`@tool_action`装饰器）

### ToolRegistry

工具注册表管理所有可用工具，提供注册、查询、执行等功能。

## 🤖 Agent类型

### SimpleAgent

最简单的Agent实现，支持可选的工具调用。工具调用通过文本标记`[TOOL_CALL:tool_name:parameters]`实现。

**适用场景**：简单的对话任务，需要基础工具支持

### FunctionCallAgent

基于OpenAI原生函数调用机制的Agent，使用标准的function calling API。

**适用场景**：需要精确工具调用、与OpenAI生态兼容的场景

### ReActAgent

ReAct（Reasoning and Acting）Agent，结合推理和行动的智能体。

**适用场景**：需要多步推理、复杂问题解决的任务

### PlanSolveAgent

规划-解决模式的Agent，先制定计划再执行。

**适用场景**：需要结构化规划的任务

### ReflectionAgent

具备反思能力的Agent，可以评估和改进自己的回答。

**适用场景**：需要高质量输出的任务

### ToolAwareAgent

工具感知Agent，专门优化了工具调用能力。

**适用场景**：工具密集型任务

## 🛠️ 工具系统

### 创建自定义工具

```python
from hello_agents_fc.tools.base import Tool, ToolParameter

class MyTool(Tool):
    def __init__(self):
        super().__init__(
            name="my_tool",
            description="我的自定义工具"
        )
    
    def get_parameters(self):
        return [
            ToolParameter(
                name="input",
                type="string",
                description="输入文本",
                required=True
            )
        ]
    
    def run(self, parameters):
        input_text = parameters.get("input", "")
        # 执行工具逻辑
        return f"处理结果: {input_text}"
```

### 可展开工具

```python
from hello_agents_fc.tools.base import Tool, tool_action

class ExpandableTool(Tool):
    def __init__(self):
        super().__init__(
            name="expandable_tool",
            description="可展开的工具",
            expandable=True  # 标记为可展开
        )
    
    @tool_action("action1", "执行动作1")
    def _action1(self, param: str) -> str:
        """执行动作1
        
        Args:
            param: 参数
        """
        return f"动作1结果: {param}"
    
    @tool_action("action2", "执行动作2")
    def _action2(self, value: int) -> str:
        """执行动作2
        
        Args:
            value: 数值
        """
        return f"动作2结果: {value}"
```

### 注册函数作为工具

```python
def my_function(input_text: str) -> str:
    return f"处理: {input_text}"

tool_registry.register_function(
    name="my_function",
    description="我的函数工具",
    func=my_function
)
```

## 🌐 LLM提供商支持

框架支持以下LLM提供商：

| 提供商 | 环境变量 | 默认模型 | 说明 |
|--------|---------|---------|------|
| OpenAI | `OPENAI_API_KEY` | `gpt-3.5-turbo` | 官方OpenAI API |
| DeepSeek | `DEEPSEEK_API_KEY` | `deepseek-chat` | DeepSeek API |
| Qwen | `DASHSCOPE_API_KEY` | `qwen-plus` | 阿里云通义千问 |
| ModelScope | `MODELSCOPE_API_KEY` | `Qwen/Qwen2.5-72B-Instruct` | 魔搭社区 |
| Kimi | `KIMI_API_KEY` | `moonshot-v1-8k` | 月之暗面Kimi |
| Zhipu | `ZHIPU_API_KEY` | `glm-4` | 智谱AI |
| Ollama | `OLLAMA_HOST` | `llama3.2` | 本地Ollama服务 |
| vLLM | `VLLM_HOST` | `meta-llama/Llama-2-7b-chat-hf` | vLLM服务 |
| Custom | `LLM_API_KEY`, `LLM_BASE_URL` | - | 自定义OpenAI兼容服务 |

### 统一配置方式

也可以使用统一的`LLM_*`环境变量：

```bash
export LLM_API_KEY="your-api-key"
export LLM_BASE_URL="https://api.example.com/v1"
export LLM_MODEL_ID="your-model"
export LLM_TIMEOUT="60"
```

框架会自动检测提供商，或通过`provider`参数指定。

## 💡 使用示例

### 示例1：基础对话

```python
from hello_agents_fc.core.llm import HelloAgentsLLM
from hello_agents_fc.agents.simple_agent import SimpleAgent

llm = HelloAgentsLLM(provider="openai")
agent = SimpleAgent(name="助手", llm=llm)

response = agent.run("什么是人工智能？")
print(response)
```

### 示例2：使用函数调用Agent

```python
from hello_agents_fc.agents.function_call_agent import FunctionCallAgent
from hello_agents_fc.tools.builtin.calculator import CalculatorTool

llm = HelloAgentsLLM(provider="openai")
tool_registry = ToolRegistry()
tool_registry.register_tool(CalculatorTool())

agent = FunctionCallAgent(
    name="计算助手",
    llm=llm,
    tool_registry=tool_registry
)

response = agent.run("计算 (123 + 456) * 789 的结果")
print(response)
```

### 示例3：流式响应

```python
agent = SimpleAgent(name="助手", llm=llm)

for chunk in agent.stream_run("请写一首关于春天的诗"):
    print(chunk, end="", flush=True)
```

### 示例4：ReAct Agent

```python
from hello_agents_fc.agents.react_agent import ReActAgent

llm = HelloAgentsLLM(provider="openai")
tool_registry = ToolRegistry()
# ... 注册工具

agent = ReActAgent(
    name="研究助手",
    llm=llm,
    tool_registry=tool_registry
)

response = agent.run("请研究一下Python的最新特性")
print(response)
```

## 📁 项目结构

```
hello-agents-fc/
├── core/                    # 核心模块
│   ├── agent.py            # Agent基类
│   ├── llm.py              # LLM统一接口
│   ├── config.py           # 配置管理
│   ├── message.py          # 消息类型
│   ├── exceptions.py       # 异常定义
│   └── database_config.py  # 数据库配置
├── agents/                  # Agent实现
│   ├── simple_agent.py     # 简单Agent
│   ├── function_call_agent.py  # 函数调用Agent
│   ├── react_agent.py      # ReAct Agent
│   ├── plan_solve_agent.py # 规划解决Agent
│   ├── reflection_agent.py # 反思Agent
│   └── tool_aware_agent.py # 工具感知Agent
├── tools/                   # 工具系统
│   ├── base.py             # 工具基类
│   ├── registry.py         # 工具注册表
│   ├── chain.py            # 工具链
│   ├── async_executor.py   # 异步执行器
│   └── builtin/            # 内置工具
│       ├── calculator.py  # 计算器工具
│       └── bfcl_evaluation_tool.py  # 评估工具
├── tests/                   # 测试目录
└── README.md               # 项目文档
```

## ⚙️ 配置说明

### 环境变量配置

框架支持通过环境变量配置LLM：

```bash
# 特定提供商配置
export OPENAI_API_KEY="sk-..."
export DEEPSEEK_API_KEY="sk-..."
export DASHSCOPE_API_KEY="sk-..."

# 或使用统一配置
export LLM_API_KEY="your-key"
export LLM_BASE_URL="https://api.example.com/v1"
export LLM_MODEL_ID="model-name"
export LLM_TIMEOUT="60"

# 系统配置
export DEBUG="false"
export LOG_LEVEL="INFO"
export TEMPERATURE="0.7"
export MAX_TOKENS="2000"
```

### 代码配置

```python
from hello_agents_fc.core.config import Config
from hello_agents_fc.core.llm import HelloAgentsLLM

# 创建配置
config = Config(
    default_model="gpt-4",
    temperature=0.8,
    max_tokens=2000,
    debug=True
)

# 使用配置
llm = HelloAgentsLLM(
    model="gpt-4",
    temperature=0.8,
    max_tokens=2000
)
```

## 🤝 贡献指南

欢迎贡献！请遵循以下步骤：

1. Fork 项目
2. 创建特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

## 📄 许可证

[在此添加许可证信息]

## 🙏 致谢

感谢所有为这个项目做出贡献的开发者！

---

如有问题或建议，请提交 Issue 或联系维护者。

