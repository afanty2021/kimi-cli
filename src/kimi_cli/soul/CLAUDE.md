[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **soul**

# Kimi CLI Soul 模块

## 变更记录 (Changelog)

- **2025-11-18 10:54**: 更新文档，添加 DMail 和 DenwaRenji 功能说明
- **2025-11-18**: 初始化 Soul 模块文档

## 模块职责

Soul 模块是 Kimi CLI 的 AI 推理引擎，负责：

- LLM 模型集成和通信
- Agent 规范加载和执行
- 工具调用协调和管理
- 对话上下文维护
- 消息处理和转换
- 运行时状态管理
- 思考过程控制
- 邮件发送功能（DMail）
- 检查点管理（DenwaRenji）

## 核心组件

### KimiSoul (`kimisoul.py`)

主要的推理引擎实现：

```python
class KimiSoul(Soul):
    async def step(self) -> StepResult
    async def run(self) -> bool
    async def send_message(self, message: Message) -> None
    async def receive_message(self) -> Message
```

### Agent 系统 (`agent.py`)

- **Agent**: Agent 规范定义和加载
- **load_agent()**: 动态加载 Agent 配置
- **DEFAULT_AGENT_FILE**: 默认 Agent 文件路径

### 运行时管理 (`runtime.py`)

- **Runtime**: 运行时环境管理
- **工具集管理**: 工具注册和调用
- **状态跟踪**: 执行状态和进度管理

### 上下文管理 (`context.py`)

- **Context**: 对话上下文维护
- **消息历史**: 消息存储和检索
- **上下文窗口**: 上下文长度管理

### 消息处理 (`message.py`)

- **消息转换**: LLM 消息格式转换
- **工具结果处理**: 工具执行结果格式化
- **系统消息**: 系统提示词管理

### 工具集管理 (`toolset.py`)

- **Toolset**: 工具集合管理
- **工具调用**: 工具执行和结果处理
- **工具发现**: 动态工具加载

### 邮件系统 (`denwarenji.py`)

- **DenwaRenji**: 电动悟吉系统，管理邮件发送和检查点
- **DMail**: 邮件数据模型
- **检查点管理**: 支持时间点标记和回滚机制

### 其他组件

- **approval.py**: 操作确认机制
- **compaction.py**: 上下文压缩

## 对外接口

### Soul 接口

```python
class Soul(ABC):
    @abstractmethod
    async def step(self) -> StepResult

    @abstractmethod
    async def send_message(self, message: Message) -> None

    @abstractmethod
    async def receive_message(self) -> Message
```

### KimiSoul 实现

```python
class KimiSoul(Soul):
    def __init__(
        self,
        session: Session,
        runtime: Runtime,
        agent: Agent,
        llm_provider: ChatProvider,
        thinking: bool = False,
        denwa_renji: DenwaRenji | None = None,  # 新增邮件系统支持
    )

    async def step(self) -> StepResult
    async def run(self, max_steps: int) -> bool
    async def handle_tool_call(self, tool_call: ToolCall) -> ToolResult
    async def process_assistant_message(self, message: Message) -> None
```

### DMail 邮件系统

```python
class DMail(BaseModel):
    message: str = Field(description="The message to send.")
    checkpoint_id: int = Field(description="The checkpoint to send the message back to.", ge=0)

class DenwaRenji:
    def __init__(self):
        self._pending_dmail: DMail | None = None
        self._n_checkpoints: int = 0

    def send_dmail(self, dmail: DMail):
        """Send a D-Mail. Intended to be called by the SendDMail tool."""

    def create_checkpoint(self) -> int:
        """Create a new checkpoint and return its ID."""

    def get_pending_dmail(self) -> DMail | None:
        """Get the pending D-Mail if any."""
```

### 异常定义

```python
class SoulException(Exception): pass
class LLMNotSet(SoulException): pass
class LLMNotSupported(SoulException): pass
class MaxStepsReached(SoulException): pass

class DenwaRenjiError(Exception): pass
```

## 关键依赖与配置

### 主要依赖

- **Kosong**: LLM 抽象层和聊天提供者
- **Tenacity**: 重试机制
- **Pydantic**: 数据验证
- **asyncio**: 异步运行时

### LLM 集成

```python
# 支持的 LLM 功能
class ModelCapability(Enum):
    THINKING = "thinking"  # 思考模式支持
    TOOL_CALLING = "tool_calling"  # 工具调用支持
    STREAMING = "streaming"  # 流式响应支持
```

### Agent 配置

Agent 规范支持 YAML 格式：

```yaml
# agent.yaml 示例
name: "Default Agent"
description: "Kimi CLI 默认代理"
model: "moonshot-v1-8k"
tools:
  - ReadFile
  - WriteFile
  - Bash
  - SendDMail  # 新增邮件工具
  # ... 其他工具
system_prompt: |
  系统提示词内容...
```

## 数据模型

### 消息模型

```python
# 基于 Kosong 消息模型
class Message:
    role: Literal["system", "user", "assistant", "tool"]
    content: list[ContentPart]

class ContentPart:
    type: Literal["text", "image", "tool_call", "tool_result"]
    # ... 其他字段
```

### 工具调用模型

```python
class ToolCall:
    id: str
    name: str
    arguments: dict[str, Any]

class ToolResult:
    tool_call_id: str
    content: str
    is_error: bool = False
```

### 邮件数据模型

```python
class DMail(BaseModel):
    message: str = Field(description="The message to send.")
    checkpoint_id: int = Field(description="The checkpoint to send the message back to.", ge=0)
    # TODO: allow restoring filesystem state to the checkpoint
```

### 状态快照

```python
class StatusSnapshot:
    step_count: int
    last_message: Message | None
    current_tools: list[str]
    thinking_enabled: bool
    pending_dmail: DMail | None  # 新增待处理邮件状态
    checkpoint_count: int  # 新增检查点计数
```

## 执行流程

### 主循环

```python
async def run(self, max_steps: int = 100) -> bool:
    while step_count < max_steps:
        # 1. 获取 LLM 响应
        response = await self.llm_provider.chat_completion(messages)

        # 2. 处理助手消息
        await self.process_assistant_message(response.message)

        # 3. 执行工具调用
        for tool_call in response.tool_calls:
            result = await self.handle_tool_call(tool_call)
            # 处理工具结果

        # 4. 更新状态
        step_count += 1
```

### 工具调用流程

```python
async def handle_tool_call(self, tool_call: ToolCall) -> ToolResult:
    # 1. 参数验证
    # 2. 工具查找
    # 3. 执行确认（如需要）
    # 4. 工具执行
    # 5. 结果格式化
    # 6. 错误处理

    # 特别处理邮件工具
    if tool_call.name == "SendDMail":
        return await self._handle_dmail_tool(tool_call)
```

### DMail 处理流程

```python
async def _handle_dmail_tool(self, tool_call: ToolCall) -> ToolResult:
    try:
        # 1. 解析邮件参数
        dmail_data = tool_call.arguments
        dmail = DMail(**dmail_data)

        # 2. 验证检查点 ID
        if dmail.checkpoint_id >= self.denwa_renji._n_checkpoints:
            raise DenwaRenjiError("Invalid checkpoint ID")

        # 3. 发送邮件
        self.denwa_renji.send_dmail(dmail)

        # 4. 返回成功结果
        return ToolResult(tool_call_id=tool_call.id, content="DMail sent successfully")

    except Exception as e:
        return ToolResult(tool_call_id=tool_call.id, content=f"Error: {str(e)}", is_error=True)
```

## 测试与质量

### 测试覆盖

- `tests/test_soul_message.py`: 消息处理测试
- `tests/test_default_agent.py`: 默认 Agent 测试
- `tests/test_load_agent.py`: Agent 加载测试
- `tests/test_load_agents_md.py`: Agent 文档加载测试
- `tests/test_task_subagents.py`: 子任务和邮件功能测试

### 质量保证

- **异步测试**: 全面支持异步测试
- **错误处理**: 完整的异常处理机制
- **重试机制**: 使用 Tenacity 进行重试
- **类型安全**: 完整的类型注解

## 高级特性

### 思考模式

```python
class ThinkingEffort(Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

# 在 KimiSoul 中启用思考模式
if self.thinking:
    request.thinking_effort = ThinkingEffort.MEDIUM
```

### 上下文压缩

```python
class SimpleCompaction:
    async def compact(self, messages: list[Message]) -> list[Message]:
        # 上下文压缩逻辑
        # 保留重要消息，压缩或删除次要消息
```

### DMail 时间旅行系统

DenwaRenji 系统实现了类似 Steins;Gate 的 D-Mail 功能：

```python
# 检查点创建
checkpoint_id = denwa_renji.create_checkpoint()

# 邮件发送到过去的时间点
dmail = DMail(
    message="重要信息，发送到过去的检查点",
    checkpoint_id=checkpoint_id
)
denwa_renji.send_dmail(dmail)
```

### Wire 协议支持

```python
def wire_send(message: Any) -> None:
    # Wire 协议消息发送
    # 用于实时状态更新和调试
```

## 常见问题 (FAQ)

### Q: 如何配置自定义 Agent？
A: 使用 `--agent-file` 参数指定自定义 Agent YAML 文件，或修改默认 Agent 配置。

### Q: 思考模式如何启用？
A: 使用 `--thinking` 参数启用，或在配置中设置默认启用。

### Q: 工具调用失败如何处理？
A: KimiSoul 会自动重试失败的工具调用，超过重试次数后报告错误。

### Q: 如何扩展新的 LLM 提供商？
A: 继承 Kosong 的 ChatProvider 接口，实现具体的 LLM 调用逻辑。

### Q: DMail 功能如何使用？
A: 在 Agent 配置中启用 SendDMail 工具，然后可以通过工具调用发送邮件到指定的检查点。

### Q: 检查点系统如何工作？
A: 通过 DenwaRenji.create_checkpoint() 创建检查点，邮件可以发送到任意已创建的检查点。

## 相关文件清单

### 核心文件

- `kimisoul.py` - 主要推理引擎实现
- `agent.py` - Agent 规范加载和管理
- `runtime.py` - 运行时环境管理
- `context.py` - 对话上下文维护
- `message.py` - 消息处理和转换
- `toolset.py` - 工具集管理
- `approval.py` - 操作确认机制
- `compaction.py` - 上下文压缩
- `denwarenji.py` - 电动悟吉邮件和检查点系统

### 工具集成

- `../tools/dmail/__init__.py` - SendDMail 工具实现
- `../tools/task/__init__.py` - 任务管理工具（支持子任务调用）

### Agent 配置

- `../agents/default/agent.yaml` - 默认 Agent 配置（已启用 SendDMail）
- `../agents/default/sub.yaml` - 子 Agent 配置
- `../agents/default/system.md` - 系统提示词

## 扩展开发

### 新功能开发

1. 在对应模块中实现新功能
2. 更新接口定义（如需要）
3. 编写单元测试
4. 更新文档

### 邮件系统扩展

- **附件支持**: 扩展 DMail 模型支持文件附件
- **多收件人**: 支持同时发送给多个检查点
- **邮件历史**: 记录和查询已发送的邮件
- **权限控制**: 添加邮件发送权限验证

### 检查点系统增强

- **状态快照**: 保存完整的系统状态
- **文件系统回滚**: 实现文件系统的状态回滚
- **分支时间线**: 支持多个并行的时间线
- **检查点清理**: 自动清理过期的检查点

### 性能优化

- **工具调用并行化**: 支持并行工具调用
- **上下文优化**: 更智能的上下文压缩
- **缓存机制**: 工具结果缓存
- **流式处理**: 改进流式响应处理