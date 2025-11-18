[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **wire**

# Kimi CLI Wire 模块

## 变更记录 (Changelog)

- **2025-11-18**: 初始化 Wire 模块文档

## 模块职责

Wire 模块提供实验性的 Wire 协议支持，包括：

- **Wire 协议实现**: 基于 JSON-RPC 的通信协议
- **消息处理**: 协议消息的编码和解码
- **状态同步**: 实时状态更新和事件推送
- **调试支持**: 开发和调试时的实时通信

## 核心组件

### 消息系统 (`message.py`)

Wire 协议的核心消息定义：

```python
class StatusUpdate(BaseModel):
    """状态更新消息"""
    step: int
    total_steps: int
    message: str
    timestamp: datetime

class StepBegin(BaseModel):
    """步骤开始消息"""
    step_id: str
    description: str
    timestamp: datetime

class StepInterrupted(BaseModel):
    """步骤中断消息"""
    step_id: str
    reason: str
    timestamp: datetime

class CompactionBegin(BaseModel):
    """上下文压缩开始消息"""
    reason: str
    timestamp: datetime

class CompactionEnd(BaseModel):
    """上下文压缩结束消息"""
    messages_removed: int
    timestamp: datetime
```

### JSON-RPC 实现 (`ui/wire/jsonrpc.py`)

Wire 协议的 JSON-RPC 服务器实现：

```python
class WireServer:
    async def start(self, host: str, port: int) -> None:
        """启动 Wire 服务器"""

    async def handle_request(self, request: dict) -> dict:
        """处理 JSON-RPC 请求"""

    async def send_notification(self, method: str, params: dict) -> None:
        """发送通知消息"""
```

## 协议规范

### 消息格式

Wire 协议使用 JSON-RPC 2.0 格式：

```json
{
  "jsonrpc": "2.0",
  "method": "status_update",
  "params": {
    "step": 5,
    "total_steps": 100,
    "message": "Processing file..."
  },
  "id": null
}
```

### 支持的方法

- **status_update**: 状态更新
- **step_begin**: 步骤开始
- **step_interrupted**: 步骤中断
- **compaction_begin**: 压缩开始
- **compaction_end**: 压缩结束

### 事件类型

```python
class WireEvent(Enum):
    STATUS_UPDATE = "status_update"
    STEP_BEGIN = "step_begin"
    STEP_INTERRUPTED = "step_interrupted"
    COMPACTION_BEGIN = "compaction_begin"
    COMPACTION_END = "compaction_end"
    ERROR = "error"
    COMPLETED = "completed"
```

## 使用方式

### 启动 Wire 服务器

```bash
# 启动 Wire 模式
kimi --wire

# 指定端口
kimi --wire --port 8080
```

### 客户端连接

```python
import asyncio
import aiohttp

async def connect_wire_client():
    ws = aiohttp.ClientSession()
    async with ws.ws_connect("ws://localhost:8080") as conn:
        async for msg in conn:
            if msg.type == aiohttp.WSMsgType.TEXT:
                data = json.loads(msg.data)
                print(f"Received: {data}")
```

## 集成方式

### 在 Soul 引擎中集成

```python
from kimi_cli.wire.message import wire_send

class KimiSoul(Soul):
    async def step(self) -> StepResult:
        # 发送步骤开始事件
        wire_send(StepBegin(
            step_id=str(self.step_count),
            description="Executing tool call"
        ))

        try:
            # 执行步骤逻辑
            result = await self.execute_step()

            # 发送状态更新
            wire_send(StatusUpdate(
                step=self.step_count,
                total_steps=self.max_steps,
                message="Step completed successfully"
            ))

            return result

        except Exception as e:
            # 发送中断事件
            wire_send(StepInterrupted(
                step_id=str(self.step_count),
                reason=str(e)
            ))
            raise
```

### 上下文压缩集成

```python
from kimi_cli.wire.message import CompactionBegin, CompactionEnd

async def compact_context(messages: list[Message]) -> list[Message]:
    wire_send(CompactionBegin(reason="Context window limit reached"))

    compacted = await perform_compaction(messages)

    wire_send(CompactionEnd(
        messages_removed=len(messages) - len(compacted)
    ))

    return compacted
```

## 关键依赖与配置

### 主要依赖

- **asyncio**: 异步运行时支持
- **aiohttp**: HTTP 和 WebSocket 服务器
- **Pydantic**: 数据验证和序列化
- **websockets**: WebSocket 协议支持

### 配置选项

```python
# Wire 服务器配置
WIRE_CONFIG = {
    "host": "localhost",
    "port": 8080,
    "max_connections": 10,
    "heartbeat_interval": 30,
    "buffer_size": 8192
}
```

## 数据模型

### 基础消息

```python
class WireMessage(BaseModel):
    """Wire 协议基础消息"""
    timestamp: datetime
    session_id: str

    class Config:
        json_encoders = {
            datetime: lambda v: v.isoformat()
        }
```

### 错误消息

```python
class WireError(BaseModel):
    """错误消息"""
    code: str
    message: str
    details: dict[str, Any] | None = None
    timestamp: datetime
```

## 性能考虑

### 连接管理

- 连接池管理
- 自动重连机制
- 心跳检测
- 优雅关闭

### 消息缓冲

- 消息队列管理
- 批量发送优化
- 内存使用控制

### 并发处理

- 异步消息处理
- 非阻塞 I/O
- 背压控制

## 调试支持

### 日志记录

```python
import logging
wire_logger = logging.getLogger("kimi_cli.wire")

wire_logger.debug("Wire server started on %s:%d", host, port)
wire_logger.info("Client connected: %s", client_id)
```

### 开发工具

```python
def create_test_client():
    """创建测试客户端用于开发"""
    return WireTestClient("ws://localhost:8080")

async def simulate_workflow():
    """模拟工作流程用于测试"""
    client = await create_test_client()
    await client.connect()

    # 模拟状态更新
    await client.send_status_update(step=1, total_steps=5, message="Starting...")

    await client.disconnect()
```

## 测试与质量

### 测试覆盖

- `tests/test_wire_message.py`: Wire 消息测试
- 集成测试需要模拟客户端连接

### 质量保证

- **消息验证**: 严格的 JSON Schema 验证
- **错误处理**: 完整的异常处理机制
- **连接安全**: 安全的连接管理

## 安全考虑

### 连接安全

- 本地连接限制
- 可选的认证机制
- 连接数限制

### 数据安全

- 消息内容过滤
- 敏感信息保护
- 访问控制

## 常见问题 (FAQ)

### Q: Wire 协议是稳定的吗？
A: Wire 协议目前处于实验阶段，API 可能会在未来版本中发生变化。

### Q: 如何在生产环境中使用 Wire 协议？
A: 目前不建议在生产环境中使用，建议等待协议稳定后再使用。

### Q: Wire 协议和 ACP 协议有什么区别？
A: Wire 协议主要用于实时状态同步和调试，ACP 协议是标准的 Agent Client Protocol。

### Q: 如何扩展 Wire 协议支持新的事件类型？
A: 在 `message.py` 中定义新的消息类型，并在处理逻辑中添加对应的处理代码。

## 相关文件清单

### 核心文件

- `__init__.py` - Wire 模块初始化
- `message.py` - Wire 协议消息定义
- `ui/wire/jsonrpc.py` - JSON-RPC 服务器实现
- `ui/wire/README.md` - Wire 协议文档

### 测试文件

- `tests/test_wire_message.py` - 消息处理测试

## 未来发展

### 协议标准化

- 定义正式的协议规范
- 向后兼容性保证
- 版本管理机制

### 功能扩展

- 更多事件类型支持
- 双向通信支持
- 插件机制

### 性能优化

- 二进制协议支持
- 压缩传输
- 连接复用