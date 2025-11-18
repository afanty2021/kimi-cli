[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **agents**

# Kimi CLI Agent 配置模块

## 变更记录 (Changelog)

- **2025-11-18 10:54**: 新增 Agent 配置模块文档
- **2025-11-18**: 初始化并分析 Agent 配置结构

## 模块职责

Agent 配置模块负责管理和定义 AI 代理的行为规范：

- **Agent 规范定义**: 定义 AI 代理的系统提示词和行为规则
- **工具配置管理**: 管理每个 Agent 可用的工具集
- **配置文件解析**: 解析和验证 YAML 格式的 Agent 配置
- **子 Agent 支持**: 支持定义和调用子 Agent
- **动态加载**: 运行时动态加载和切换 Agent 配置

## 配置文件结构

### 默认 Agent 配置

位于 `src/kimi_cli/agents/default/` 目录：

- **`agent.yaml`**: 主 Agent 配置文件
- **`sub.yaml`**: 子 Agent 配置文件
- **`system.md`**: 系统提示词模板

### 配置文件详解

#### agent.yaml

```yaml
version: 1
agent:
  name: ""
  system_prompt_path: ./system.md
  system_prompt_args:
    ROLE_ADDITIONAL: ""
  tools:
    - "kimi_cli.tools.task:Task"
    - "kimi_cli.tools.todo:SetTodoList"
    - "kimi_cli.tools.bash:Bash"
    - "kimi_cli.tools.file:ReadFile"
    - "kimi_cli.tools.file:Glob"
    - "kimi_cli.tools.file:Grep"
    - "kimi_cli.tools.file:WriteFile"
    - "kimi_cli.tools.file:StrReplaceFile"
    - "kimi_cli.tools.web:SearchWeb"
    - "kimi_cli.tools.web:FetchURL"
```

#### 工具分类

当前启用的工具按功能分类：

**文件操作**:
- `ReadFile`: 读取文件内容
- `WriteFile`: 写入文件内容
- `StrReplaceFile`: 字符串替换
- `Glob`: 文件模式匹配
- `Grep`: 文本搜索

**系统交互**:
- `Bash`: 执行 Shell 命令
- `Task`: 子任务管理

**网络功能**:
- `SearchWeb`: Web 搜索
- `FetchURL`: 获取 URL 内容

**任务管理**:
- `SetTodoList`: 待办事项管理

**可选工具** (默认禁用):
- `SendDMail`: 邮件发送
- `Think`: 思考过程记录
- `PatchFile`: 补丁文件应用

## 系统提示词

### system.md 模板

包含 Agent 的核心行为指导：

- 角色定义和能力描述
- 工具使用规范
- 输出格式要求
- 错误处理策略
- 安全约束和限制

## 核心接口

### Agent 规范处理

```python
# agentspec.py 中的核心类
class AgentSpec:
    def load_from_file(cls, path: Path) -> "AgentSpec"

    def validate(self) -> None:
        # 验证配置文件格式和内容

    def get_tool_list(self) -> list[str]:
        # 获取启用的工具列表

    def get_system_prompt(self, **kwargs) -> str:
        # 获取格式化的系统提示词
```

### 配置加载机制

- **路径解析**: 自动解析相对路径引用
- **参数替换**: 支持系统提示词中的变量替换
- **继承机制**: 支持配置继承和覆盖
- **验证检查**: 严格的配置验证和错误报告

## 关键依赖与配置

### 主要依赖

- **PyYAML**: YAML 配置文件解析
- **Jinja2**: 模板渲染（用于系统提示词）
- **pathlib**: 路径处理和验证

### 配置验证

- **Schema 验证**: 严格的配置格式检查
- **工具存在性**: 验证指定的工具类是否存在
- **路径有效性**: 检查引用文件和路径的有效性
- **权限检查**: 验证工具使用权限

## 数据模型

### Agent 配置模型

```python
@dataclass
class AgentConfig:
    name: str
    version: int
    system_prompt_path: Path
    system_prompt_args: dict[str, str]
    tools: list[str]
    description: str | None = None
    metadata: dict[str, Any] | None = None
```

### 工具规范

```python
@dataclass
class ToolSpec:
    name: str
    module_path: str
    class_name: str
    enabled: bool = True
    config: dict[str, Any] | None = None
```

## 测试与质量

### 测试覆盖

- `tests/test_default_agent.py`: 默认 Agent 配置测试
- `tests/test_agent_spec.py`: Agent 规范解析测试
- `tests/test_load_agent.py`: Agent 加载功能测试
- `tests/test_load_agents_md.py`: Agent 文档加载测试

### 质量保证

- **配置验证**: 完整的配置文件验证机制
- **错误处理**: 详细的错误信息和恢复建议
- **向后兼容**: 支持旧版本配置文件的迁移

## 高级特性

### 子 Agent 系统

通过 `sub.yaml` 定义子 Agent：

```yaml
# 子 Agent 用于专门任务
sub_agents:
  - name: "file_analyzer"
    description: "专门用于文件分析"
    tools: ["ReadFile", "Grep", "Glob"]
    system_prompt: "./file_analyzer.md"
```

### 动态配置

- **运行时切换**: 支持运行时切换 Agent 配置
- **条件启用**: 基于环境条件启用/禁用工具
- **参数化配置**: 支持通过参数动态调整配置

### 配置继承

```yaml
# 继承基础配置
extends: "./base.yaml"

# 覆盖特定配置
agent:
  name: "specialized_agent"
  tools:
    - "kimi_cli.tools.special:SpecialTool"
```

## 常见问题 (FAQ)

### Q: 如何添加新的工具到 Agent？
A: 在 `agent.yaml` 的 `tools` 列表中添加工具路径，格式为 `"module.path:ClassName"`。

### Q: 如何自定义系统提示词？
A: 修改 `system.md` 文件，或创建新的系统提示词文件并更新 `system_prompt_path`。

### Q: 如何创建专门的子 Agent？
A: 在 `sub.yaml` 中定义子 Agent 配置，通过工具和行为限制来实现专门化。

### Q: 配置文件验证失败怎么办？
A: 检查 YAML 语法、工具路径格式、文件引用路径等，使用详细错误信息定位问题。

### Q: 如何禁用某些工具？
A: 从 `tools` 列表中移除相应工具，或设置 `enabled: false`。

## 相关文件清单

### 核心文件

- `agentspec.py` - Agent 规范处理核心类
- `__init__.py` - 模块初始化

### 配置目录

- `default/agent.yaml` - 默认 Agent 配置
- `default/sub.yaml` - 子 Agent 配置
- `default/system.md` - 系统提示词模板

### 提示词目录

- `../prompts/init.md` - 初始化提示词
- `../prompts/compact.md` - 压缩提示词

## 扩展开发

### 新 Agent 配置

1. 创建新的配置目录
2. 编写 Agent YAML 配置文件
3. 创建系统提示词文件
4. 编写配置验证测试
5. 更新文档和使用示例

### 工具集成

1. 在相应工具模块中实现新工具
2. 在 Agent 配置中添加工具引用
3. 编写工具测试和文档
4. 更新系统提示词（如需要）

### 配置模板

- 支持配置模板系统
- 提供常用场景的配置模板
- 支持配置生成和向导

## 最佳实践

### 配置管理

- **版本控制**: 对配置文件进行版本控制
- **环境分离**: 不同环境使用不同配置
- **文档同步**: 保持配置与文档同步更新

### 工具选择

- **最小权限**: 只启用必要的工具
- **功能聚焦**: 根据用途选择合适的工具集
- **性能考虑**: 避免启用过多影响性能的工具

### 系统提示词

- **清晰明确**: 明确指定 Agent 的角色和限制
- **上下文感知**: 根据可用工具调整提示词
- **安全约束**: 包含必要的安全和隐私约束