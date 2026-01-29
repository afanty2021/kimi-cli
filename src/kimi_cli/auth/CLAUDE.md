[根目录](../../../CLAUDE.md) > [src](../../) > [kimi_cli](../) > **auth**

# Kimi CLI Auth 模块

## 变更记录 (Changelog)

- **2026-01-28**: 初始化 Auth 模块文档 - OAuth 授权、平台管理、令牌刷新

## 模块职责

Auth 模块提供 OAuth 认证管理功能，负责：

- **OAuth 授权流程**: 设备代码授权流程实现
- **令牌管理**: 访问令牌和刷新令牌的安全存储
- **平台管理**: 多平台支持（Kimi Code、Moonshot AI）
- **自动刷新**: 令牌过期自动刷新机制
- **CLI 集成**: 与 Shell、CLI 子命令和斜杠命令集成

## 入口与启动

### 主要文件

- **`oauth.py`**: OAuth 授权流程核心实现
- **`platforms.py`**: 平台配置和模型管理
- **`__init__.py`**: 模块初始化，导出平台常量

### 核心常量

```python
KIMI_CODE_PLATFORM_ID = "kimi-code"
KIMI_CODE_CLIENT_ID = "17e5f671-d194-4dfb-9706-5516cb48c098"
KIMI_CODE_OAUTH_KEY = "oauth/kimi-code"
DEFAULT_OAUTH_HOST = "https://auth.kimi.com"
KEYRING_SERVICE = "kimi-code"
REFRESH_INTERVAL_SECONDS = 60
REFRESH_THRESHOLD_SECONDS = 300
```

## 对外接口

### CLI 子命令 (v1.0+)

```bash
# 登录 Kimi 账户
kimi login [--json]

# 登出 Kimi 账户
kimi logout [--json]
```

### 斜杠命令 (v1.0+)

```bash
# 在 Shell 模式下登录
/login

# 在 Shell 模式下登出
/logout
```

### 核心函数

#### OAuth 授权流程

```python
async def login_kimi_code(
    config: Config,
    *, open_browser: bool = True
) -> AsyncIterator[OAuthEvent]:
    """执行 Kimi Code OAuth 登录流程

    流程:
    1. 请求设备授权
    2. 打开浏览器进行用户授权
    3. 轮询等待授权完成
    4. 获取模型列表
    5. 应用配置
    """
```

#### 登出流程

```python
async def logout_kimi_code(
    config: Config
) -> AsyncIterator[OAuthEvent]:
    """登出 Kimi Code 账户

    清理:
    1. 删除存储的令牌
    2. 移除相关配置
    3. 重置默认模型
    """
```

#### 令牌管理

```python
def load_tokens(ref: OAuthRef) -> OAuthToken | None:
    """从 keyring 或文件加载令牌"""

def save_tokens(ref: OAuthRef, token: OAuthToken) -> OAuthRef:
    """保存令牌到 keyring 或文件"""

def delete_tokens(ref: OAuthRef) -> None:
    """删除存储的令牌"""
```

#### 平台管理

```python
def get_platform_by_id(platform_id: str) -> Platform | None:
    """根据 ID 获取平台配置"""

async def list_models(platform: Platform, api_key: str) -> list[ModelInfo]:
    """获取平台的可用模型列表"""

async def refresh_managed_models(config: Config) -> bool:
    """刷新托管平台的模型列表"""
```

## 关键依赖与配置

### 主要依赖

- **aiohttp**: 异步 HTTP 客户端
- **keyring**: 安全令牌存储
- **Pydantic**: 数据验证
- **webbrowser**: 浏览器自动打开

### 数据模型

```python
@dataclass(slots=True, frozen=True)
class OAuthEvent:
    """OAuth 事件消息"""
    type: OAuthEventKind  # "info", "error", "waiting", "verification_url", "success"
    message: str
    data: dict[str, Any] | None = None

    @property
    def json(self) -> str:
        """JSON 格式输出"""

@dataclass(slots=True)
class OAuthToken:
    """OAuth 令牌"""
    access_token: str
    refresh_token: str
    expires_at: float
    scope: str
    token_type: str

@dataclass(slots=True)
class DeviceAuthorization:
    """设备授权响应"""
    user_code: str
    device_code: str
    verification_uri: str
    verification_uri_complete: str
    expires_in: int | None
    interval: int
```

### 平台配置

```python
class Platform(NamedTuple):
    """平台配置"""
    id: str
    name: str
    base_url: str
    search_url: str | None = None
    fetch_url: str | None = None
    allowed_prefixes: list[str] | None = None

# 支持的平台
PLATFORMS = [
    Platform(
        id="kimi-code",
        name="Kimi Code",
        base_url="https://api.kimi.com/coding/v1",
        search_url="https://api.kimi.com/coding/v1/search",
        fetch_url="https://api.kimi.com/coding/v1/fetch",
    ),
    Platform(
        id="moonshot-cn",
        name="Moonshot AI Open Platform (moonshot.cn)",
        base_url="https://api.moonshot.cn/v1",
        allowed_prefixes=["kimi-k"],
    ),
    Platform(
        id="moonshot-ai",
        name="Moonshot AI Open Platform (moonshot.ai)",
        base_url="https://api.moonshot.ai/v1",
        allowed_prefixes=["kimi-k"],
    ),
]
```

### 模型信息

```python
class ModelInfo(BaseModel):
    """模型信息"""
    id: str
    context_length: int
    supports_reasoning: bool
    supports_image_in: bool
    supports_video_in: bool

    @property
    def capabilities(self) -> set[ModelCapability]:
        """派生模型能力"""
        caps: set[ModelCapability] = set()
        if self.supports_reasoning:
            caps.add("thinking")
        if "thinking" in self.id.lower():
            caps.update(("thinking", "always_thinking"))
        if self.supports_image_in:
            caps.add("image_in")
        if self.supports_video_in:
            caps.add("video_in")
        if "kimi-k2.5" in self.id.lower():
            caps.update(("thinking", "image_in", "video_in"))
        return caps
```

## 数据模型

### OAuthManager

```python
class OAuthManager:
    """OAuth 令牌管理器

    功能:
    - 令牌加载和缓存
    - 自动令牌刷新
    - API 密钥解析
    - 运行时集成
    """
    def __init__(self, config: Config) -> None:
        """初始化管理器"""

    def common_headers(self) -> dict[str, str]:
        """获取公共请求头（设备信息）"""

    def resolve_api_key(
        self, api_key: SecretStr, oauth: OAuthRef | None
    ) -> str:
        """解析 API 密钥（优先使用 OAuth 令牌）"""

    async def ensure_fresh(self, runtime: Runtime) -> None:
        """确保令牌是最新的"""

    @asynccontextmanager
    async def refreshing(self, runtime: Runtime) -> AsyncIterator[None]:
        """上下文管理器，在后台自动刷新令牌"""
```

### 配置集成

```python
# 在 Config 中添加 OAuth 配置
class LLMProvider(BaseModel):
    type: ProviderType
    base_url: str
    api_key: SecretStr
    oauth: OAuthRef | None = None  # OAuth 引用

class OAuthRef(BaseModel):
    storage: Literal["keyring", "file"]
    key: str  # 令牌键名

class MoonshotSearchConfig(BaseModel):
    base_url: str
    api_key: SecretStr
    oauth: OAuthRef | None = None

class MoonshotFetchConfig(BaseModel):
    base_url: str
    api_key: SecretStr
    oauth: OAuthRef | None = None
```

## 测试与质量

### 测试覆盖

- 单元测试：OAuth 流程测试
- 集成测试：端到端登录/登出测试
- Mock 测试：模拟 OAuth 服务器响应

### 质量保证

- **类型安全**: 完整的类型注解
- **错误处理**: 完整的异常处理
- **安全存储**: keyring 优先，文件后备
- **自动刷新**: 后台自动刷新过期令牌

## 常见问题 (FAQ)

### Q: 如何使用 OAuth 登录？
A: 运行 `kimi login`，会自动打开浏览器进行授权。授权完成后自动配置模型。

### Q: 令牌存储在哪里？
A: 优先使用系统 keyring 存储，如果不可用则回退到 `~/.kimi/credentials/` 目录。

### Q: 令牌会自动刷新吗？
A: 是的，OAuthManager 会在后台自动刷新过期令牌（距离过期 5 分钟内）。

### Q: 如何登出？
A: 运行 `kimi logout`，或在 Shell 模式下使用 `/logout` 斜杠命令。

### Q: 支持哪些平台？
A: 目前支持 Kimi Code、Moonshot AI (moonshot.cn)、Moonshot AI (moonshot.ai)。

### Q: 登录后如何切换模型？
A: 使用 `/model` 斜杠命令查看和切换可用模型。

### Q: OAuth 登录失败怎么办？
A: 检查网络连接，确认浏览器能正常打开授权页面。使用 `--json` 选项查看详细错误信息。

### Q: 如何在脚本中使用 OAuth？
A: 使用 `kimi login --json` 获取 JSON 格式的事件输出，便于脚本解析。

## 相关文件清单

### 核心文件

- `oauth.py` - OAuth 授权流程实现
- `platforms.py` - 平台配置和模型管理
- `__init__.py` - 模块初始化

### 集成文件

- `src/kimi_cli/cli/__init__.py` - CLI 子命令集成
- `src/kimi_cli/shell/slash.py` - 斜杠命令集成
- `src/kimi_cli/app.py` - 应用集成（OAuthManager）

### 测试文件

- `tests/test_auth.py` - OAuth 测试

## 架构集成

### 与 Shell 集成

```python
# 在 Shell 模式下
# 用户输入 /login
# 调用 login_kimi_code()
# 显示 OAuth 事件
# 完成后重新加载配置
```

### 与 CLI 子命令集成

```python
# kimi login 命令
@cli.command()
def login(json: bool = False) -> None:
    """Login to your Kimi account."""
    async for event in login_kimi_code(load_config()):
        if json:
            typer.echo(event.json)
        else:
            console.print(event.message)
```

### 与运行时集成

```python
# 在 KimiCLI 创建时
oauth_manager = OAuthManager(config)

# 在 Soul 运行时
async with oauth_manager.refreshing(runtime):
    await soul.run(input)
```

## 未来发展

### 功能扩展

- 支持更多 OAuth 提供商
- 设备代码授权优化
- 令牌安全增强
- 多账户支持

### 用户体验改进

- 授权状态持久化
- 更详细的错误提示
- 授权历史记录
- 令牌过期提醒
