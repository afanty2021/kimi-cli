[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **tools**

# Kimi CLI 工具模块

## 变更记录 (Changelog)

- **2025-11-18 11:17**: 增量更新 - 完善 SetTodoList Markdown 渲染功能文档
- **2025-11-18**: 初始化工具模块文档

## 模块职责

工具模块是 Kimi CLI 的核心功能组件，提供了丰富的操作工具集：

- 文件系统操作（读写、搜索、替换）
- Shell 命令执行
- Web 搜索和内容获取
- 任务管理和子代理调用
- 思考过程记录
- **✨ 增强待办事项管理**（支持 Markdown 样式渲染）
- MCP (Model Context Protocol) 集成

## 工具分类

### 文件操作工具 (`file/`)

- **ReadFile**: 读取文件内容，支持分页读取
- **WriteFile**: 写入文件内容，支持覆盖和创建
- **StrReplaceFile**: 字符串替换文件内容
- **PatchFile**: 应用补丁文件
- **Glob**: 文件模式匹配搜索
- **Grep**: 文本内容搜索

### Shell 工具 (`bash/`)

- **Bash**: 执行 Shell 命令
- **CMD**: 命令执行工具（Bash 的别名）

### Web 工具 (`web/`)

- **SearchWeb**: Web 搜索功能
- **FetchURL**: 获取 URL 内容

### 任务管理工具 (`task/`)

- **Task**: 创建和管理子代理任务

### 思考工具 (`think/`)

- **Think**: 记录和展示思考过程

### 待办工具 (`todo/`)

- **SetTodoList**: **管理待办事项列表（支持 Markdown 样式渲染）**
  - **新功能**: 支持 Markdown 样式渲染，提供更好的视觉效果
  - **渲染规则**:
    - 完成状态: `~~任务标题~~ [Done]` （删除线）
    - 进行中状态: `**任务标题** [In Progress]` （粗体）
    - 待办状态: `任务标题 [Pending]` （普通文本）
  - **适用场景**: 多步骤任务分解、项目进度跟踪、复杂工作流管理

### MCP 工具

- **MCP**: Model Context Protocol 集成支持

### 邮件工具 (`dmail/`)

- **SendDMail**: 发送邮件功能

## 工具注册机制

### 动态加载

工具通过动态导入机制加载：

```python
# tools/__init__.py
def load_tools():
    # 动态发现和加载工具模块
    # 支持 SkipThisTool 异常跳过特定工具
```

### 工具规范

每个工具遵循统一的规范：

- **工具描述**: Markdown 格式的工具文档
- **参数验证**: 使用 Pydantic 进行参数验证
- **错误处理**: 统一的错误处理机制
- **结果返回**: 标准化的结果格式

## 核心接口

### 工具基类

所有工具都基于统一的接口规范：

```python
# 工具签名规范
def tool_name(
    param1: type,
    param2: type,
    # ... 其他参数
) -> ToolResult:
    # 工具实现
```

### 工具参数提取

```python
def extract_key_argument(
    json_content: str | streamingjson.Lexer,
    tool_name: str
) -> str | None:
    # 从 JSON 内容中提取关键参数用于显示
```

## 关键依赖与配置

### 主要依赖

- **streamingjson**: 流式 JSON 解析
- **ripgrepy**: ripgrep Python 绑定，用于文件搜索
- **trafilatura**: Web 内容提取
- **aiohttp**: 异步 HTTP 客户端
- **fastmcp**: MCP 协议实现

### 外部工具集成

- **ripgrep**: 高性能文本搜索
- **patch**: 补丁工具集成
- **浏览器引擎**: Web 内容获取

## 数据模型

### 工具结果

```python
class ToolResult:
    # 标准化的工具执行结果格式
    success: bool
    data: Any
    error: str | None
```

### SetTodoList 数据模型

```python
class Todo(BaseModel):
    title: str = Field(description="The title of the todo", min_length=1)
    status: Literal["Pending", "In Progress", "Done"] = Field(description="The status of the todo")

class SetTodoListResult(ToolOk):
    brief: str  # Markdown 格式的渲染输出
```

### 工具配置

- **默认超时**: 工具执行超时配置
- **重试策略**: 失败重试机制
- **安全限制**: 文件访问和命令执行限制

## 测试与质量

### 测试覆盖

每个工具模块都有对应的测试：

- `tests/test_bash.py`: Shell 工具测试
- `tests/test_glob.py`: 文件搜索测试
- `tests/test_grep.py`: 文本搜索测试
- `tests/test_read_file.py`: 文件读取测试
- `tests/test_write_file.py`: 文件写入测试
- `tests/test_str_replace_file.py`: 字符串替换测试
- `tests/test_patch_file.py`: 补丁应用测试
- `tests/test_fetch_url.py`: URL 获取测试

### SetTodoList 特定测试

- `tests/test_tool_schemas.py`: 工具模式验证
- `tests/test_tool_descriptions.py`: 工具描述验证
- `tests/test_default_agent.py`: 集成测试

### 安全考虑

- **路径验证**: 文件操作路径安全检查
- **命令限制**: Shell 命令执行白名单
- **资源限制**: 文件大小和操作频率限制

## 工具使用示例

### 文件操作

```python
# 读取文件
result = await ReadFile(path="/path/to/file.txt")

# 搜索文件
result = await Glob(pattern="**/*.py")

# 文本搜索
result = await Grep(pattern="TODO", path="src/")
```

### Shell 操作

```python
# 执行命令
result = await Bash(command="ls -la", work_dir="/tmp")
```

### Web 操作

```python
# 搜索 Web
result = await SearchWeb(query="Python async programming")

# 获取 URL 内容
result = await FetchURL(url="https://example.com")
```

### ✨ SetTodoList 增强使用

```python
# 创建带状态的待办列表
result = await SetTodoList(todos=[
    Todo(title="设计 API 架构", status="Done"),
    Todo(title="实现核心功能", status="In Progress"),
    Todo(title="编写测试用例", status="Pending"),
    Todo(title="部署到生产环境", status="Pending")
])

# 输出效果（Markdown 渲染）:
# - ~~设计 API 架构~~ [Done]
# - **实现核心功能** [In Progress]
# - 编写测试用例 [Pending]
# - 部署到生产环境 [Pending]
```

## 常见问题 (FAQ)

### Q: 如何添加自定义工具？
A: 在 `tools/` 目录下创建新的工具模块，实现标准接口，工具会被自动发现和加载。

### Q: 工具执行失败如何处理？
A: 工具返回包含错误信息的 `ToolResult`，上层系统会根据错误类型进行重试或报告。

### Q: 如何限制工具的访问权限？
A: 通过路径验证、命令白名单和资源限制等机制确保工具安全执行。

### Q: MCP 工具如何配置？
A: 通过 CLI 参数 `--mcp-config-file` 或 `--mcp-config` 加载 MCP 服务器配置。

### Q: SetTodoList 的 Markdown 样式如何工作？
A: SetTodoList 会根据任务状态自动应用 Markdown 样式：
- Done: 删除线效果
- In Progress: 粗体强调
- Pending: 普通文本
这些样式在支持 Markdown 的环境中会自动渲染。

## 相关文件清单

### 核心文件

- `__init__.py` - 工具注册和管理
- `utils.py` - 工具通用功能
- `test.py` - 测试工具
- `mcp.py` - MCP 协议集成

### 工具模块

- `bash/__init__.py` - Shell 命令工具
- `file/` - 文件操作工具集
- `web/` - Web 操作工具集
- `task/__init__.py` - 任务管理工具
- `think/__init__.py` - 思考工具
- `todo/__init__.py` - **✨ 增强待办工具**
- `dmail/__init__.py` - 邮件工具

### 工具文档

每个工具都有对应的 Markdown 文档：
- `bash/bash.md`, `bash/cmd.md`
- `file/glob.md`, `file/grep.md`, `file/read.md`, `file/write.md` 等
- `web/fetch.md`, `web/search.md`
- `task/task.md`, `think/think.md`
- `todo/set_todo_list.md` - **待办工具使用指南**

## 扩展开发

### 新工具开发

1. 在 `tools/` 下创建新目录
2. 实现工具函数，遵循参数和返回值规范
3. 创建对应的 Markdown 文档
4. 编写单元测试
5. 更新工具注册逻辑（如需要）

### 工具配置

- 支持通过配置文件自定义工具行为
- 支持环境变量配置
- 支持运行时参数调整

### SetTodoList 扩展思路

- 可以添加更多状态类型（如 "Blocked", "Review"）
- 可以支持优先级设置
- 可以集成日期和时间跟踪
- 可以支持子任务嵌套