[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **wire**

# Kimi CLI Wire 模块

## 变更记录 (Changelog)

- **2026-01-28**: 重大更新 - 顶层模块提升、Wire 服务器、文件后端支持
- **2026-01-20**: 更新至 v0.80 - 初始化方法、外部工具调用、审批响应
- **2025-11-18**: 初始化 Wire 模块文档

## 模块职责

Wire 模块提供 Wire 协议支持，包括：

- **Wire 协议**: 基于 JSON-RPC 的通信协议
- **消息系统**: 单生产者多消费者（SPMC）消息队列
- **服务器**: Wire 协议服务器实现
- **文件后端**: 消息持久化到文件
- **外部工具**: 支持外部工具调用
- **审批请求**: 支持操作审批流程

## 入口与启动

### 主要文件

- **`wire/__init__.py`**: Wire 核心实现（SPMC 消息队列）
- **`wire/server.py`**: Wire 服务器（JSON-RPC）
- **`wire/types.py`**: 消息类型定义
- **`wire/protocol.py`**: 协议常量和版本
- **`wire/file.py`**: 文件后端支持
- **`wire/serde.py`**: 消息序列化
- **`wire/jsonrpc.py`**: JSON-RPC 消息适配

### 架构定位

Wire 模块已从 `ui/wire/` 提升为顶层模块 `wire/`，独立于 UI 系统。

## 对外接口

### Wire 消息队列

```python
class Wire:
    """SPMC 消息通道

    特性:
    - 原始消息队列（未合并）
    - 合并消息队列（合并可合并消息）
    - 文件后端支持（持久化）
    - 广播机制（多订阅者）
    """

    def __init__(self, *, file_backend: WireFile | None = None):
        """创建 Wire 实例

        Args:
            file_backend: 可选的文件后端用于持久化
        """

    @property
    def soul_side(self) -> WireSoulSide:
        """Soul 侧接口（发送消息）"""

    def ui_side(self, *, merge: bool) -> WireUISide:
        """UI 侧接口（接收消息）

        Args:
            merge: 是否合并消息
        """

    def shutdown(self) -> None:
        """关闭 Wire"""

    async def join(self) -> None:
        """等待文件后端刷新完成"""
```

### Wire 服务器

```python
class WireServer:
    """Wire 协议服务器（基于 JSON-RPC）

    功能:
    - stdio 通信
    - 初始化握手
    - 外部工具调用
    - 审批请求处理
    - 优雅关闭
    """

    async def serve(self) -> None:
        """启动 Wire 服务器"""

    async def _handle_initialize(
        self, msg: JSONRPCInitializeMessage
    ) -> JSONRPCSuccessResponse | JSONRPCErrorResponse:
        """处理初始化请求

        功能:
        - 交换客户端/服务器信息
        - 注册外部工具
        - 广告斜杠命令
        """

    async def _handle_prompt(
        self, msg: JSONRPCPromptMessage
    ) -> JSONRPCSuccessResponse | JSONRPCErrorResponse:
        """处理提示请求

        执行 Agent 轮次并流式返回消息
        """

    async def _handle_cancel(
        self, msg: JSONRPCCancelMessage
    ) -> JSONRPCSuccessResponse | JSONRPCErrorResponse:
        """处理取消请求"""
```

### JSON-RPC 消息

```python
# 初始化消息
class JSONRPCInitializeMessage(BaseModel):
    method: Literal["initialize"]
    params: InitializeParams
    id: str

    class InitializeParams(BaseModel):
        client: ClientInfo | None
        external_tools: list[ToolSchema]

# 提示消息
class JSONRPCPromptMessage(BaseModel):
    method: Literal["prompt"]
    params: PromptParams
    id: str

    class PromptParams(BaseModel):
        user_input: str

# 取消消息
class JSONRPCCancelMessage(BaseModel):
    method: Literal["cancel"]
    id: str

# 事件消息
class JSONRPCEventMessage(BaseModel):
    method: Literal["event"]
    params: WireMessage
```

## 关键依赖与配置

### 主要依赖

- **asyncio**: 异步运行时
- **acp**: stdio 流处理
- **Pydantic**: 数据验证
- **kosong**: 消息类型和工具

### 消息类型

```python
# Wire 消息基类
class WireMessage(BaseModel):
    """Wire 协议消息基类"""

# 具体消息类型
class TurnBegin(BaseModel):
    """标记每次 agent 轮次的开始"""

class StatusUpdate(BaseModel):
    """状态更新消息"""
    step: int
    snapshot: StatusSnapshot
    token_usage: dict | None
    message_id: str | None

class ApprovalRequest(BaseModel):
    """操作确认请求"""
    id: str
    title: str
    description: str
    display: list[DisplayBlock] | None

class ToolCallRequest(BaseModel):
    """外部工具调用请求"""
    id: str
    name: str
    arguments: dict[str, Any]

class ContentPart(BaseModel):
    """内容部分（支持合并）"""
    role: Literal["user", "assistant", "system", "tool"]
    content: str | list[TextBlock | ImageBlock]
    mergeable: bool = True
```

### 协议版本

```python
WIRE_PROTOCOL_VERSION = "0.2.0"
```

## 数据模型

### Wire 消息队列

```python
class WireSoulSide:
    """Soul 侧接口

    功能:
    - 发送原始消息
    - 合并可合并消息
    - 刷新合并缓冲区
    """

    def send(self, msg: WireMessage) -> None:
        """发送消息"""

    def flush(self) -> None:
        """刷新合并缓冲区"""

class WireUISide:
    """UI 侧接口

    功能:
    - 接收消息（原始或合并）
    - 异步队列操作
    """

    async def receive(self) -> WireMessage:
        """接收下一条消息"""
```

### 文件后端

```python
class WireFile:
    """文件后端

    功能:
    - 消息持久化
    - 追加写入
    - 读取历史
    """

    async def append_message(self, msg: WireMessage) -> None:
        """追加消息到文件"""

    async def read_messages(self) -> list[WireMessage]:
        """读取所有消息"""
```

### 广播队列

```python
class BroadcastQueue(Generic[T]):
    """广播队列

    特性:
    - 单生产者
    - 多消费者
    - 发布/订阅模式
    - 优雅关闭
    """

    def subscribe(self) -> Queue[T]:
        """订阅队列，返回新的消费者队列"""

    def publish_nowait(self, item: T) -> None:
        """发布消息到所有订阅者"""

    def shutdown(self) -> None:
        """关闭队列"""
```

## 测试与质量

### 测试覆盖

- 单元测试：消息队列测试
- 集成测试：Wire 服务器测试
- 协议测试：JSON-RPC 兼容性测试

### 质量保证

- **类型安全**: 完整的类型注解
- **错误处理**: 完整的异常处理
- **协议兼容**: JSON-RPC 2.0 兼容
- **优雅关闭**: 正确的资源清理

## 常见问题 (FAQ)

### Q: Wire 协议和 ACP 协议有什么区别？
A: Wire 协议用于实时状态同步和调试，ACP 协议是标准的 Agent Client Protocol。

### Q: 如何启动 Wire 服务器？
A: 使用 `kimi --wire` 命令启动 Wire 服务器。

### Q: Wire 消息会持久化吗？
A: 默认不持久化，可以传入 `file_backend` 参数启用文件持久化。

### Q: 如何订阅 Wire 消息？
A: 使用 `wire.ui_side(merge=True/False)` 创建订阅者。

### Q: 外部工具如何调用？
A: 通过初始化时注册外部工具，服务器会在工具调用时发送 `ToolCallRequest`。

### Q: 审批请求如何处理？
A: 服务器发送 `ApprovalRequest`，客户端返回 `ApprovalResponse`。

### Q: 消息合并是如何工作的？
A: 可合并的消息（如 `ContentPart`）会自动合并，减少消息数量。

### Q: Wire 协议稳定吗？
A: Wire 协议仍在活跃开发中，API 可能会变化。

## 相关文件清单

### 核心文件

- `__init__.py` - Wire 核心实现
- `server.py` - Wire 服务器
- `types.py` - 消息类型定义
- `protocol.py` - 协议常量
- `file.py` - 文件后端
- `serde.py` - 序列化
- `jsonrpc.py` - JSON-RPC 适配

### 测试文件

- `tests/test_wire.py` - Wire 测试
- `tests/test_wire_server.py` - 服务器测试

## 架构集成

### 与 Soul 集成

```python
# 在 Soul 运行时
wire = Wire(file_backend=wire_file)
async with wire.soul_side as soul_wire:
    await run_soul(
        soul,
        user_input,
        wire.soul_side,
        cancel_event,
        wire_file,
    )
```

### 与 UI 集成

```python
# UI 订阅消息
wire_ui = wire.ui_side(merge=True)
while True:
    msg = await wire_ui.receive()
    match msg:
        case StatusUpdate():
            # 更新状态
        case ApprovalRequest():
            # 请求审批
        case ToolCallRequest():
            # 调用工具
```

### 与服务器集成

```python
# Wire 服务器
server = WireServer(soul)
await server.serve()

# 处理请求
await server._handle_initialize(msg)
await server._handle_prompt(msg)
await server._handle_cancel(msg)
```

## 未来发展

### 功能扩展

- 二进制协议支持
- 压缩传输
- 连接复用
- 更多事件类型

### 性能优化

- 消息批量处理
- 背压控制
- 内存优化
- 延迟降低

### 稳定性改进

- 协议版本管理
- 向后兼容性
- 错误恢复
- 连接重建
