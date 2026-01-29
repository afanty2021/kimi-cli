[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **skill**

# Kimi CLI Skill 模块

## 变更记录 (Changelog)

- **2026-01-28**: 重大更新 - 目录结构重构、Flow Skills 支持（Mermaid/D2）
- **2026-01-21**: 更新至 v0.81 - Flow skills 类型、嵌入式流程图
- **2026-01-04**: 更新至 v0.72 - 技能系统增强、目录候选扩展

## 模块职责

Skill 模块提供技能发现、加载和管理功能，负责：

- **技能发现**: 从多个目录发现技能（内置、用户、项目）
- **SKILL.md 解析**: 解析 frontmatter 和技能内容
- **Flow Skills**: 支持嵌入式流程图（Mermaid/D2）
- **技能索引**: 按名称索引技能，支持模糊匹配
- **分层加载**: 内置 → 用户 → 项目的分层加载机制

## 入口与启动

### 主要文件

- **`skill/__init__.py`**: 技能发现和加载核心逻辑
- **`skill/flow/__init__.py`**: Flow 基础数据结构
- **`skill/flow/mermaid.py`**: Mermaid 流程图解析
- **`skill/flow/d2.py`**: D2 流程图解析

### 目录结构

```
src/kimi_cli/skill/
├── __init__.py         # 技能发现和加载
├── flow/
│   ├── __init__.py     # Flow 基础类型
│   ├── mermaid.py      # Mermaid 解析器
│   └── d2.py           # D2 解析器
└── skills/             # 内置技能目录
    └── kimi-cli-help/  # 内置帮助技能
```

## 对外接口

### 技能目录

```python
# 用户级技能目录（优先级顺序）
~/.config/agents/skills/
~/.agents/skills/
~/.kimi/skills/
~/.claude/skills/
~/.codex/skills/

# 项目级技能目录（优先级顺序）
./.agents/skills/
./.kimi/skills/
./.claude/skills/
./.codex/skills/

# 内置技能目录
src/kimi_cli/skills/
```

### 核心函数

#### 技能发现

```python
async def discover_skills(skills_dir: KaosPath) -> list[Skill]:
    """发现指定目录中的所有技能

    Args:
        skills_dir: Kaos 路径到技能目录

    Returns:
        技能列表，按名称排序
    """

async def discover_skills_from_roots(
    skills_dirs: Iterable[KaosPath]
) -> list[Skill]:
    """从多个目录根发现技能

    后面的目录会覆盖前面的同名技能
    """

async def resolve_skills_roots(
    work_dir: KaosPath,
    *,
    skills_dir_override: KaosPath | None = None,
) -> list[KaosPath]:
    """解析分层技能根目录（优先级顺序）

    优先级: 内置 → 用户 → 项目
    """
```

#### 技能解析

```python
def parse_skill_text(
    content: str, *, dir_path: KaosPath
) -> Skill:
    """解析 SKILL.md 内容

    提取 frontmatter 中的:
    - name: 技能名称
    - description: 技能描述
    - type: 技能类型（standard/flow）

    对于 flow 类型，解析 Mermaid/D2 流程图
    """

async def read_skill_text(skill: Skill) -> str | None:
    """读取技能的 SKILL.md 内容"""
```

#### Flow 解析

```python
def parse_mermaid_flowchart(code: str) -> Flow:
    """解析 Mermaid 流程图

    支持的节点类型:
    - begin: 开始节点
    - end: 结束节点
    - task: 任务节点
    - decision: 决策节点
    """

def parse_d2_flowchart(code: str) -> Flow:
    """解析 D2 流程图

    语法与 Mermaid 类似
    """
```

### 数据模型

```python
class Skill(BaseModel):
    """技能信息"""
    name: str
    description: str
    type: SkillType = "standard"  # "standard" 或 "flow"
    dir: KaosPath
    flow: Flow | None = None  # Flow Skills 的流程图

    @property
    def skill_md_file(self) -> KaosPath:
        """SKILL.md 文件路径"""

class Flow:
    """流程图数据结构"""
    nodes: dict[str, FlowNode]
    outgoing: dict[str, list[FlowEdge]]
    begin_id: str
    end_id: str

class FlowNode:
    """流程图节点"""
    id: str
    label: str | list[ContentPart]
    kind: FlowNodeKind  # "begin", "end", "task", "decision"

class FlowEdge:
    """流程图边"""
    src: str
    dst: str
    label: str | None  # 决策节点的分支标签
```

## 关键依赖与配置

### 主要依赖

- **kaos**: 文件系统抽象
- **Pydantic**: 数据验证
- **loguru**: 日志记录

### SKILL.md 格式

#### 标准技能

```markdown
---
name: My Skill
description: 技能描述
---

# 技能指令

这里是技能的具体内容，AI 会按需加载这些指令。
```

#### Flow Skills

```markdown
---
name: My Flow Skill
description: 使用流程图定义工作流
type: flow
---

# 工作流程

```mermaid
begin --> task1
task1 --> decision1
decision1 -->|是| task2
decision1 -->|否| end
task2 --> end
```

## 任务说明

### 任务 1
...

### 决策 1
- **是**: 继续执行任务 2
- **否**: 结束流程

### 任务 2
...
```

### Flow Skills 语法

#### Mermaid 格式

```mermaid
begin --> task1
task1 --> decision1
decision1 -->|条件 A| task2
decision1 -->|条件 B| task3
task2 --> end
task3 --> end
```

#### D2 格式

```d2
begin -> task1
task1 -> decision1
decision1 -> task2: 条件 A
decision1 -> task3: 条件 B
task2 -> end
task3 -> end
```

## 数据模型

### 技能类型

```python
SkillType = Literal["standard", "flow"]
```

### 节点类型

```python
FlowNodeKind = Literal["begin", "end", "task", "decision"]
```

### 流程验证

```python
def validate_flow(
    nodes: dict[str, FlowNode],
    outgoing: dict[str, list[FlowEdge]],
) -> tuple[str, str]:
    """验证流程图

    检查:
    - 恰好一个 begin 节点
    - 恰好一个 end 节点
    - end 节点可达
    - 决策节点的边都有标签
    - 决策节点的边标签唯一
    """
```

## 测试与质量

### 测试覆盖

- 单元测试：技能发现和解析
- 集成测试：Flow 执行测试
- 格式验证：Mermaid/D2 语法验证

### 质量保证

- **类型安全**: 完整的类型注解
- **错误处理**: 友好的错误提示
- **格式验证**: 严格的流程图验证
- **向后兼容**: 兼容旧版本技能格式

## 常见问题 (FAQ)

### Q: 标准技能和 Flow Skills 有什么区别？
A: 标准技能是纯文本指令，Flow Skills 包含流程图定义工作流程和决策逻辑。

### Q: 如何调用 Flow Skills？
A: 使用 `/flow:<name>` 或 `/skill:<name>` 斜杠命令都可以。

### Q: Flow Skills 支持哪些格式？
A: 目前支持 Mermaid 和 D2 两种流程图格式。

### Q: 技能的优先级是什么？
A: 内置 → 用户 → 项目。后面的会覆盖前面的同名技能。

### Q: 如何自定义技能目录？
A: 使用 `--skills-dir` 选项指定自定义目录。

### Q: Flow Skills 如何定义条件分支？
A: 在决策节点的边上使用 `|条件|` 语法定义分支条件。

### Q: 技能目录支持哪些位置？
A: 支持用户级（`~/`）和项目级（`./`）多个目录。

### Q: 内置技能有哪些？
A: 目前有 `kimi-cli-help` 内置帮助技能。

## 相关文件清单

### 核心文件

- `__init__.py` - 技能发现和加载
- `flow/__init__.py` - Flow 基础类型
- `flow/mermaid.py` - Mermaid 解析器
- `flow/d2.py` - D2 解析器

### 内置技能

- `skills/kimi-cli-help/SKILL.md` - 内置帮助技能

### 测试文件

- `tests/test_skill.py` - 技能系统测试
- `tests/test_flow.py` - Flow 解析测试

## 架构集成

### 与 Shell 集成

```python
# 用户输入 /skill:my-skill
# 查找技能
# 读取 SKILL.md 内容
# 作为系统消息插入
```

### 与 Soul 集成

```python
# Flow Skills 执行
# 1. 解析流程图
# 2. 从 begin 节点开始
# 3. 执行任务节点
# 4. 处理决策节点
# 5. 到达 end 节点
```

### 与 KimiCLI 集成

```python
# 初始化时发现技能
skills = await discover_skills_from_roots(roots)
skills_index = index_skills(skills)

# 运行时按需加载
skill = skills_index.get(normalize_skill_name(name))
content = await read_skill_text(skill)
```

## 未来发展

### 功能扩展

- 更多流程图格式支持
- 技能依赖管理
- 技能版本控制
- 技能市场

### 用户体验改进

- 技能搜索和推荐
- 技能预览功能
- 技能编辑器
- 技能分享机制
