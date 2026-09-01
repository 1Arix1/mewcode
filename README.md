# MewCode

MewCode 是一个使用 Python 开发的终端 AI 编程助手。它把大模型调用、文件与命令行工具、权限控制、会话记忆、MCP、子 Agent 和 Git worktree 协作整合在同一个终端界面中。

这个仓库适合作为 AI 应用开发作品集，也可以用于研究一个终端 Agent 如何组织模型、工具、上下文和执行权限。当前版本为 `0.2.0`，要求 Python `3.11` 或更高版本。

## 核心能力

| 能力 | 说明 |
| --- | --- |
| 多种模型协议 | 支持 Anthropic、OpenAI Responses API 和 OpenAI 兼容接口 |
| 终端交互界面 | 基于 Textual 提供持续对话、状态显示和命令面板 |
| 非交互模式 | 使用 `-p` 执行单次任务，可输出普通文本或 NDJSON 事件流 |
| 工具调用 | 提供文件读取、编辑、搜索、命令执行和用户确认等工具 |
| 权限控制 | 可通过配置文件或 `--mode` 调整工具执行权限 |
| 会话与记忆 | 保存对话状态，并支持压缩、恢复和记忆管理 |
| MCP 扩展 | 可连接 stdio 或 HTTP MCP Server |
| Agent 协作 | 包含子 Agent、团队任务、消息传递和进度管理能力 |
| Git worktree | 支持为隔离任务创建和管理工作树 |
| Hook | 可在指定事件上运行自定义命令或脚本 |

## 工作方式

```text
用户输入
   ↓
Textual 终端界面或 -p 单次任务
   ↓
模型客户端选择与上下文组装
   ↓
模型返回文本或工具调用
   ↓
权限检查 → 工具执行 → 结果写回对话
   ↓
继续推理，直到生成最终回答
```

## 快速开始

### 安装

下面以 Windows PowerShell 为例。

```powershell
git clone https://github.com/1Arix1/mewcode.git
Set-Location mewcode

python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -e .
```

### 配置模型

MewCode 会按顺序读取用户目录和当前项目中的配置文件。最简单的方式是在项目根目录创建 `.mewcode/config.yaml`。

```yaml
providers:
  - name: openai
    protocol: openai
    base_url: https://api.openai.com/v1
    model: your-model-name

permission_mode: default
```

API Key 建议放在环境变量中，不要写入配置文件或提交到 Git。

```powershell
$env:OPENAI_API_KEY = '在本机填写 API Key'
```

使用 Anthropic 协议时设置 `ANTHROPIC_API_KEY`。使用 `openai-compat` 协议时设置 `OPENAI_API_KEY`，并将 `base_url` 和 `model` 改为对应服务的值。

配置文件的读取顺序如下，后读取的项目配置可以覆盖用户级配置。

```text
~/.mewcode/config.yaml
当前目录/.mewcode/config.yaml
当前目录/.mewcode/config.local.yaml
```

### 启动

启动终端交互界面。

```powershell
mewcode
```

执行一次非交互任务。

```powershell
mewcode -p '解释当前项目的目录结构'
```

查看可用参数。

```powershell
mewcode --help
```

`--remote` 会在 `0.0.0.0:18888` 启动 WebSocket 服务。这个模式会监听所有网络接口，只应在可信网络和明确的访问控制下使用。

## 运行测试

开发依赖在 `pyproject.toml` 的 `dev` 依赖组中。使用 uv 时可以执行以下命令。

```powershell
uv sync --group dev
uv run pytest
```

没有安装 uv 时，也可以单独安装测试依赖。

```powershell
python -m pip install pytest pytest-asyncio
python -m pytest
```

## 项目结构

| 路径 | 作用 |
| --- | --- |
| `mewcode/__main__.py` | CLI 入口和运行模式选择 |
| `mewcode/app.py` | Textual 终端应用 |
| `mewcode/client.py` | Anthropic、OpenAI 和兼容接口客户端 |
| `mewcode/tools/` | 内置工具及其执行逻辑 |
| `mewcode/permissions/` | 权限模式和危险操作检查 |
| `mewcode/memory/` | 会话记忆和自动记忆 |
| `mewcode/mcp/` | MCP 客户端和工具封装 |
| `mewcode/agents/` | 子 Agent 定义、加载和追踪 |
| `mewcode/teams/` | 多 Agent 团队和任务协调 |
| `mewcode/worktree/` | Git worktree 生命周期管理 |
| `tests/` | 单元测试和行为测试 |

## 当前限制

部分沙箱、终端分屏和 worktree 协作能力依赖操作系统或外部程序，Windows、macOS 与 Linux 上的可用范围并不完全相同。模型是否支持工具调用、推理参数和上下文窗口，也取决于所选服务端接口。

项目能够执行命令和修改文件。首次使用时建议保留默认权限模式，在确认工具调用内容后再放宽权限。

## License

仓库目前未提供独立的开源许可证。在添加许可证之前，请勿默认将代码用于再分发或商业发布。
