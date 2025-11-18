[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **ui**

# Kimi CLI UI 模块

## 变更记录 (Changelog)

- **2025-11-18**: 初始化 UI 模块文档

## 模块职责

UI 模块负责 Kimi CLI 的用户界面实现，提供多种交互模式：

- **Shell 模式**: 交互式命令行界面，类似终端体验
- **Print 模式**: 非交互式的打印输出模式
- **ACP 协议**: Agent Client Protocol 服务器支持
- **Wire 协议**: 实验性 Wire 协议服务器支持
- **可视化渲染**: Markdown、代码高亮、进度显示

## 核心组件

### Shell 界面 (`shell/`)

提供完整的交互式 Shell 体验：

- **console.py**: Rich 控制台配置和样式
- **prompt.py**: 命令提示符和输入处理
- **setup.py**: Shell 环境初始化和设置
- **keyboard.py**: 键盘事件处理和快捷键
- **metacmd.py**: 元命令处理（如 /help, /setup）
- **debug.py**: 调试功能实现
- **replay.py**: 会话回放功能
- **update.py**: 更新检查和提示
- **visualize.py**: 可视化渲染和显示

### Print 模式 (`print/`)

- **visualize.py**: 打印模式的内容渲染

### Wire 协议 (`wire/`)

- **jsonrpc.py**: JSON-RPC 协议实现
- **README.md**: Wire 协议文档说明

### ACP 支持 (`acp/`)

- **__init__.py**: ACP 协议支持实现

## Shell 模式详解

### 控制台配置

```python
# shell/console.py
_NEUTRAL_MARKDOWN_THEME = Theme({
    "markdown.paragraph": "none",
    "markdown.code": "none",
    # ... 其他样式配置
})

console = Console(highlight=False, theme=_NEUTRAL_MARKDOWN_THEME)
```

### 提示符系统

```python
# shell/prompt.py
class KimiPrompt:
    def __init__(self, session: Session)

    def get_prompt(self) -> str:
        # 生成动态提示符，包含当前路径、模式等信息

    async def get_input(self) -> str:
        # 获取用户输入，支持多行输入和编辑
```

### 元命令处理

支持的元命令包括：

- `/help`: 显示帮助信息
- `/setup`: 设置向导
- `/exit`: 退出程序
- `/clear`: 清屏
- `/history`: 显示历史记录
- `/mode`: 切换模式

### 键盘交互

```python
# shell/keyboard.py
def setup_keybindings():
    # 设置快捷键绑定
    # Ctrl-X: 切换 Shell/Agent 模式
    # Ctrl-C: 中断当前操作
    # Tab: 自动补全
```

## Print 模式

### 渲染引擎

```python
# print/visualize.py
def render_content(content: Any, format: str = "text") -> str:
    # 根据输出格式渲染内容
    # 支持 text、stream-json 等格式
```

### 用途

- 批处理操作
- 脚本集成
- CI/CD 流水线
- 自动化工具

## 协议支持

### ACP (Agent Client Protocol)

```python
# acp/__init__.py
class ACPServer:
    async def start(self, host: str, port: int) -> None:
        # 启动 ACP 服务器

    async def handle_request(self, request: ACPRequest) -> ACPResponse:
        # 处理 ACP 请求
```

### Wire 协议

```python
# wire/jsonrpc.py
class WireServer:
    async def handle_jsonrpc(self, request: dict) -> dict:
        # 处理 JSON-RPC 请求

    async def send_status_update(self, status: StatusUpdate) -> None:
        # 发送状态更新
```

## 可视化渲染

### Markdown 渲染

- 支持 GitHub Flavored Markdown
- 代码高亮显示
- 表格和列表渲染
- 链接和图片处理

### 进度显示

```python
# 使用 Rich 的进度条
with Progress(console=console) as progress:
    task = progress.add_task("Processing...", total=100)
    # 更新进度
    progress.update(task, advance=10)
```

### 错误和警告

- 颜色编码的错误信息
- 详细的堆栈跟踪显示
- 警告和提示信息

## 关键依赖与配置

### 主要依赖

- **Rich**: 富文本终端渲染
- **prompt-toolkit**: 交互式命令行界面
- **Pillow**: 图像处理支持

### 配置选项

- **主题配置**: 可自定义的颜色主题
- **键盘绑定**: 可配置的快捷键
- **显示选项**: 详细程度控制

## 数据模型

### 状态更新

```python
class StatusUpdate:
    step: int
    total_steps: int
    message: str
    progress: float
```

### 用户输入

```python
class UserInput:
    text: str
    timestamp: datetime
    metadata: dict[str, Any]
```

## 测试与质量

### 测试覆盖

- `tests/test_metacmd.py`: 元命令测试
- UI 交互测试（需要手动测试或模拟）

### 质量保证

- **响应性**: 确保界面响应流畅
- **错误处理**: 优雅的错误显示和恢复
- **可访问性**: 支持屏幕阅读器等辅助功能

## 高级特性

### 模式切换

用户可以在不同模式间切换：

- **Agent 模式**: 与 AI 助手交互
- **Shell 模式**: 直接执行系统命令
- 快捷键: `Ctrl-X` 切换模式

### 自动补全

```python
class FileCompleter:
    def get_completions(self, document: Document) -> list[Completion]:
        # 提供文件路径自动补全
```

### 历史记录

- 命令历史存储
- 会话历史持久化
- 历史搜索功能

## 常见问题 (FAQ)

### Q: 如何自定义颜色主题？
A: 修改 `shell/console.py` 中的主题配置，或通过配置文件指定自定义主题。

### Q: Shell 模式和 Agent 模式有什么区别？
A: Shell 模式直接执行系统命令，Agent 模式通过 AI 助手处理请求。

### Q: 如何启用 Wire 协议？
A: 使用 `--wire` 参数启动 Wire 协议服务器，目前处于实验阶段。

### Q: ACP 协议支持哪些编辑器？
A: 支持任何兼容 Agent Client Protocol 的编辑器，如 Zed、VS Code（通过插件）。

## 相关文件清单

### 核心文件

- `shell/console.py` - Rich 控制台配置
- `shell/prompt.py` - 提示符和输入处理
- `shell/setup.py` - Shell 环境设置
- `shell/keyboard.py` - 键盘事件处理
- `shell/metacmd.py` - 元命令处理
- `shell/debug.py` - 调试功能
- `shell/replay.py` - 会话回放
- `shell/update.py` - 更新功能
- `shell/visualize.py` - 可视化渲染

### 协议支持

- `wire/jsonrpc.py` - JSON-RPC 实现
- `wire/README.md` - Wire 协议文档
- `acp/__init__.py` - ACP 协议支持

### 渲染组件

- `print/visualize.py` - 打印模式渲染
- `__init__.py` - UI 模块初始化

## 扩展开发

### 新的渲染器

1. 在相应目录下创建新的渲染器
2. 实现统一的渲染接口
3. 注册到主渲染系统中

### 新的协议支持

1. 在 `wire/` 或 `acp/` 目录下实现
2. 遵循对应的协议规范
3. 提供完整的错误处理

### UI 改进

- 改进响应性
- 增强可访问性
- 添加更多可视化效果
- 优化内存使用