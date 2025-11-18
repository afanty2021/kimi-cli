[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **utils**

# Kimi CLI Utils 模块

## 变更记录 (Changelog)

- **2025-11-18**: 初始化 Utils 模块文档

## 模块职责

Utils 模块提供 Kimi CLI 的通用工具函数库，包括：

- **日志系统**: 统一的日志记录和管理
- **Rich 组件**: Rich 库的扩展组件
- **文件路径**: 路径操作和规范化
- **字符串处理**: 文本处理和格式化
- **终端操作**: 终端环境检测和操作
- **系统信号**: 信号处理和优雅退出
- **HTTP 客户端**: 异步 HTTP 请求封装
- **剪贴板**: 剪贴板操作支持
- **变更日志**: 变更日志处理
- **PyInstaller**: 打包相关工具

## 核心组件

### 日志系统 (`logging.py`)

统一的日志记录系统：

```python
# 配置日志系统
logger.add(
    log_file,
    level="INFO",
    rotation="06:00",
    retention="10 days",
    format="{time} | {level} | {name}:{function}:{line} - {message}"
)

# 支持结构化日志
logger.info("Operation completed", extra={
    "operation": "file_read",
    "duration": 0.123,
    "file_path": "/path/to/file"
})
```

### Rich 组件 (`rich/`)

Rich 库的扩展组件：

- **columns.py**: 多列布局组件
- **markdown.py**: Markdown 渲染增强
- **__init__.py**: Rich 组件初始化

### 文件路径 (`path.py`)

文件路径操作工具：

```python
def normalize_path(path: Path | str) -> Path:
    # 规范化文件路径
    # 处理相对路径、符号链接等

def get_relative_path(path: Path, base: Path) -> Path:
    # 获取相对路径

def ensure_directory(path: Path) -> Path:
    # 确保目录存在
```

### 字符串处理 (`string.py`)

文本处理和格式化工具：

```python
def shorten_middle(text: str, width: int = 50) -> str:
    # 从中间缩短文本，保留首尾
    # 示例: "very long text..." -> "very...text"

def truncate_lines(text: str, max_lines: int) -> str:
    # 截断文本行数

def format_size(size: int) -> str:
    # 格式化文件大小显示
```

### 终端操作 (`term.py`)

终端环境检测和操作：

```python
def is_terminal() -> bool:
    # 检测是否在终端环境中运行

def get_terminal_size() -> tuple[int, int]:
    # 获取终端尺寸

def supports_color() -> bool:
    # 检测终端是否支持颜色
```

### 系统信号 (`signals.py`)

信号处理和优雅退出：

```python
def setup_signal_handlers():
    # 设置信号处理器
    # 处理 SIGINT, SIGTERM 等

def graceful_shutdown():
    # 优雅关闭应用程序
    # 保存状态，清理资源
```

### HTTP 客户端 (`aiohttp.py`)

异步 HTTP 请求封装：

```python
async def fetch_url(
    url: str,
    headers: dict[str, str] | None = None,
    timeout: int = 30
) -> tuple[str, dict[str, str]]:
    # 获取 URL 内容
    # 返回 (content, headers)

async def download_file(
    url: str,
    output_path: Path,
    chunk_size: int = 8192
) -> Path:
    # 下载文件
    # 支持进度显示
```

### 剪贴板 (`clipboard.py`)

剪贴板操作支持：

```python
def copy_to_clipboard(text: str) -> bool:
    # 复制文本到剪贴板
    # 跨平台支持

def paste_from_clipboard() -> str | None:
    # 从剪贴板粘贴文本
```

### 变更日志 (`changelog.py`)

变更日志处理工具：

```python
def parse_changelog(content: str) -> list[ChangeLogEntry]:
    # 解析变更日志格式

def format_changelog_entry(entry: ChangeLogEntry) -> str:
    # 格式化变更日志条目
```

### PyInstaller 工具 (`pyinstaller.py`)

打包相关工具：

```python
def get_resource_path(relative_path: str) -> Path:
    # 获取打包后的资源路径
    # 处理开发环境和打包环境的差异

def is_bundled() -> bool:
    # 检测是否为打包版本
```

### 消息工具 (`message.py`)

消息处理和格式化：

```python
def format_tool_result(result: ToolResult) -> str:
    # 格式化工具执行结果

def format_error(error: Exception) -> str:
    # 格式化错误信息

def extract_key_info(text: str, max_length: int = 100) -> str:
    # 提取关键信息
```

## 关键依赖与配置

### 主要依赖

- **loguru**: 结构化日志记录
- **Rich**: 终端富文本渲染
- **aiohttp**: 异步 HTTP 客户端
- **Pillow**: 图像处理（剪贴板支持）
- **PyInstaller**: 应用程序打包

### 配置选项

```python
# 日志配置
LOG_CONFIG = {
    "level": "INFO",
    "rotation": "06:00",
    "retention": "10 days",
    "format": "{time} | {level} | {name}:{function}:{line} - {message}"
}

# HTTP 配置
HTTP_CONFIG = {
    "timeout": 30,
    "max_redirects": 5,
    "user_agent": "Kimi-CLI/0.54"
}
```

## 数据模型

### 变更日志条目

```python
class ChangeLogEntry(BaseModel):
    version: str
    date: datetime
    changes: list[str]
    type: Literal["feature", "fix", "breaking", "doc"]
```

### HTTP 响应

```python
class HTTPResponse(BaseModel):
    status_code: int
    headers: dict[str, str]
    content: str
    url: str
```

## 性能优化

### 异步操作

- 所有 I/O 操作都使用 async/await
- HTTP 客户端支持连接池
- 文件操作支持异步流

### 内存管理

- 流式处理大文件
- 及时释放资源
- 避免内存泄漏

### 缓存机制

```python
# 简单的内存缓存
_cache: dict[str, Any] = {}

def cached_result(key: str, func: Callable[[], T]) -> T:
    if key not in _cache:
        _cache[key] = func()
    return _cache[key]
```

## 测试与质量

### 测试覆盖

- `tests/test_utils_path.py`: 路径操作测试
- `tests/test_pyinstaller_utils.py`: PyInstaller 工具测试
- `tests/test_changelog.py`: 变更日志测试
- `tests/test_message_utils.py`: 消息工具测试

### 质量保证

- **类型安全**: 完整的类型注解
- **错误处理**: 全面的异常处理
- **跨平台**: 支持 Windows、macOS、Linux

## 常见问题 (FAQ)

### Q: 如何配置日志级别？
A: 使用 `--debug` 参数启用调试级别日志，或修改日志配置。

### Q: 剪贴板功能在哪些平台支持？
A: 支持 Windows、macOS 和 Linux（需要 xclip/xsel）。

### Q: 如何处理大文件下载？
A: 使用流式下载，支持断点续传和进度显示。

### Q: 打包后的资源路径如何处理？
A: 使用 `get_resource_path()` 函数自动处理开发环境和打包环境的路径差异。

## 相关文件清单

### 核心工具

- `__init__.py` - 工具模块初始化
- `logging.py` - 日志系统
- `path.py` - 文件路径操作
- `string.py` - 字符串处理
- `term.py` - 终端操作
- `signals.py` - 信号处理
- `message.py` - 消息处理
- `aiohttp.py` - HTTP 客户端
- `clipboard.py` - 剪贴板操作
- `changelog.py` - 变更日志处理
- `pyinstaller.py` - 打包工具

### Rich 组件

- `rich/__init__.py` - Rich 组件初始化
- `rich/columns.py` - 多列布局
- `rich/markdown.py` - Markdown 渲染

### 示例文件

- `rich/markdown_sample.md` - Markdown 示例
- `rich/markdown_sample_short.md` - 短示例

## 扩展开发

### 新工具添加

1. 在相应类别下创建新工具文件
2. 实现工具函数
3. 添加类型注解和文档
4. 编写单元测试
5. 更新模块导出

### 性能优化

- 使用缓存机制
- 优化异步操作
- 减少内存占用
- 提高响应速度

### 平台适配

- 添加新平台支持
- 处理平台差异
- 测试跨平台兼容性