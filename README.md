<div align="center">

# csc-pkg

**CoStrict 二进制分发仓库**

[项目简介](#项目简介) · [安装](#安装) · [使用](#快速开始) · [发布](#发布说明)

</div>

---

## 项目简介

本仓库用于构建并分发 [CoStrict](https://github.com/zgsm-sangfor/csc) 的命令行二进制文件。

CoStrict 是一款免费开源的 AI 辅助编程工具，专为企业级开发场景设计，支持私有化部署，是组织级安全、标准化 AI 开发工作流的选择。

| 能力 | 说明 |
|------|------|
| **多协议 API 兼容** | 支持 CoStrict / OpenAI 兼容 / Gemini / xAI / Bedrock / Vertex / Foundry 等多种协议 |
| **50+ 内置工具** | 文件读写、Shell 执行、Web 搜索、Agent 调度、MCP 扩展——开箱即用 |
| **结构化 Agent 编排** | 需求分析 → 架构设计 → 任务规划 → 编码 → 验证全链路覆盖 |
| **私有化部署** | Remote Control Server (Docker) + Bridge Mode，数据不出内网 |
| **Feature Flag 体系** | 多种默认特性 + 环境变量热开关，按需启用零代码改动 |
| **安全合规** | 权限模式分级、Policy 策略网关、Worktree 隔离、审计日志 |
| **跨平台** | macOS / Windows / Linux |

> **注意**：本仓库仅负责二进制打包与发布。CoStrict 源码及详细文档请访问 [zgsm-sangfor/csc](https://github.com/zgsm-sangfor/csc)。

---

## 发布说明

推送以 `v` 开头的 tag 到本仓库后，GitHub Actions 会自动：

1. 拉取 [csc](https://github.com/zgsm-sangfor/csc) 仓库的 `main` 分支最新代码
2. 执行 `bun install` 与 `bun run build`
3. 使用 `bun build --compile` 编译以下目标平台二进制：
   - `linux-x64`、`linux-x64-baseline`、`linux-x64-musl`、`linux-x64-musl-baseline`
   - `windows-x64`、`windows-x64-baseline`
   - `darwin-arm64`、`darwin-x64`、`darwin-x64-baseline`
4. 打包为 `.tar.gz`（Linux / macOS）或 `.zip`（Windows）
5. 创建 GitHub Release 并上传资源

> **前置条件**：csc 源码仓库为私有仓库，需要在 `Settings → Secrets and variables → Actions` 中配置名为 `CSC_PAT` 的 Personal Access Token（至少拥有 `repo` 读取权限），供工作流拉取代码。

示例：

```bash
git tag v4.2.16
git push origin v4.2.16
```

Release 资源命名：`csc-{version}-{target}.{tar.gz|zip}`

---

## 安装

从 [Releases](https://github.com/zgsm-sangfor/csc-pkg/releases) 页面下载对应平台压缩包，解压后将可执行文件加入 `PATH`。

```bash
# macOS ARM64 示例
curl -L -o csc-4.2.16-darwin-arm64.tar.gz \
  https://github.com/zgsm-sangfor/csc-pkg/releases/download/v4.2.16/csc-4.2.16-darwin-arm64.tar.gz
tar -xzf csc-4.2.16-darwin-arm64.tar.gz
mv csc-darwin-arm64 /usr/local/bin/csc
chmod +x /usr/local/bin/csc
```

---

## 快速开始

### 环境要求

- [Bun](https://bun.sh/) >= 1.2.0（源码运行）

### 源码版

```bash
git clone https://github.com/zgsm-sangfor/csc.git
cd csc
bun install
bun run dev       # 版本号显示正常即成功
bun run build     # 生产构建 → dist/cli.js + chunk 文件
```

### 首次登录

启动后输入 `/login`，选择认证方式：

**1. CoStrict 企业登录（推荐）**

浏览器自动打开 CoStrict OAuth 页面，完成 SSO 登录后自动写入凭证。

**2. 第三方 API 直连**

选择 OpenAI 兼容 / Gemini 等协议，填写 Base URL 和 API Key：

| 字段 | 说明 | 示例 |
|------|------|------|
| Base URL | API 服务地址 | `https://api.example.com/v1` |
| API Key | 认证密钥 | `sk-xxx` |
| 快速模型 | 轻量任务模型 | `model-fast` |
| 均衡模型 | 日常任务模型 | `model-balanced` |
| 高性能模型 | 复杂任务模型 | `model-large` |

也可直接编辑配置文件配置环境变量：

```json
{
  "env": {
    "OPENAI_BASE_URL": "https://api.example.com/v1",
    "OPENAI_API_KEY": "sk-xxx",
    "DEFAULT_FAST_MODEL": "model-fast",
    "DEFAULT_BALANCED_MODEL": "model-balanced",
    "DEFAULT_LARGE_MODEL": "model-large"
  }
}
```

---

## 架构总览

```
┌───────────────────────────────────────────────────────────┐
│  CLI 入口 (cli.tsx) — 快速路径 + 完整 CLI                 │
├───────────────────────────────────────────────────────────┤
│  REPL 对话循环                                             │
│  REPL.tsx → QueryEngine → query() → API Client            │
│  流式响应 · Tool 调度 · 上下文压缩 · 记忆管理             │
├───────────────┬───────────────────────────────────────────┤
│  CoStrict 专有 │            通用工具层                     │
│  ┌──────────┐ │  ┌─────────────────────────────────────┐  │
│  │ Provider  │ │  │ 50+ 内置工具                        │  │
│  │  OAuth    │ │  │ 文件 / Shell / Agent / Web / MCP   │  │
│  │  Token    │ │  │ Cron / Worktree / LSP / Skill      │  │
│  │  Fetch    │ │  └─────────────────────────────────────┘  │
│  │ Models    │ ├───────────────────────────────────────────┤
│  │ Mapping   │ │            Agent 编排层                   │
│  └──────────┘ │  ┌─────────────────────────────────────┐  │
│  ┌──────────┐ │  │ CoStrict Agents                     │  │
│  │ Agents   │ │  │ StrictSpec · TDD · Wiki · Security  │  │
│  │          │ │  └─────────────────────────────────────┘  │
│  └──────────┘ │  ┌─────────────────────────────────────┐  │
│  ┌──────────┐ │  │ 通用 Agents                         │  │
│  │ Skills   │ │  │ AgentTool · Team · Coordinator      │  │
│  │          │ │  │ Fork Subagent · Worktree 隔离       │  │
│  └──────────┘ │  └─────────────────────────────────────┘  │
├───────────────┴───────────────────────────────────────────┤
│  API 兼容层 (多协议) → 统一流适配器 → 下游代码零改动     │
├───────────────────────────────────────────────────────────┤
│  基础设施                                                 │
│  Feature Flag · Daemon · Bridge/RCS · Voice Mode          │
│  BG Sessions · Policy 网关 · Sentry · GrowthBook          │
└───────────────────────────────────────────────────────────┘
```

---

## API 多协议兼容

CoStrict 通过流适配器模式将多种 API 协议转为内部统一格式，下游代码零改动。

| 协议 | 启用方式 | 适用场景 |
|------|---------|---------|
| **CoStrict** | `/login` 选 CoStrict | 企业 SSO + AI Gateway 动态模型 |
| **OpenAI 兼容** | 按登录或环境变量启用 | Ollama / DeepSeek / vLLM |
| **Gemini** | 环境变量启用 | Google Gemini API |
| **xAI Grok** | 环境变量启用 | xAI Grok API |
| **AWS Bedrock** | 环境变量启用 | AWS Bedrock |
| **Google Vertex** | 环境变量启用 | Vertex AI |
| **Foundry** | 环境变量启用 | Foundry |

---

## 工具体系

### 内置工具分类

| 分类 | 工具 | 说明 |
|------|------|------|
| **文件操作** | `FileReadTool` `FileEditTool` `FileWriteTool` `GlobTool` `GrepTool` | 代码读写与搜索 |
| **Shell 执行** | `BashTool` `PowerShellTool` `REPLTool` | 命令执行与交互式 REPL |
| **Agent 协作** | `AgentTool` `TaskCreateTool` `TaskUpdateTool` `TaskListTool` `TaskGetTool` `TaskOutputTool` `TaskStopTool` | 多 Agent 任务调度 |
| **团队编排** | `TeamCreateTool` `TeamDeleteTool` `SendMessageTool` | Agent Team 编排 |
| **规划验证** | `EnterPlanModeTool` `ExitPlanModeTool` `VerifyPlanExecutionTool` | 结构化规划与验证 |
| **Web 网络** | `WebFetchTool` `WebSearchTool` `WebBrowserTool` | 网页获取/搜索/浏览器自动化 |
| **MCP 扩展** | `MCPTool` `McpAuthTool` `ListMcpResourcesTool` `ReadMcpResourceTool` | Model Context Protocol |
| **调度定时** | `CronCreateTool` `CronDeleteTool` `CronListTool` | Cron 定时任务 |
| **工作区** | `EnterWorktreeTool` `ExitWorktreeTool` | Git Worktree 隔离 |
| **交互辅助** | `AskUserQuestionTool` `TodoWriteTool` `SkillTool` `ToolSearchTool` `DiscoverSkillsTool` `ConfigTool` | 用户交互与配置 |
| **开发辅助** | `LSPTool` `NotebookEditTool` `BriefTool` | LSP / Notebook / 会话摘要 |
| **远程触发** | `RemoteTriggerTool` `MonitorTool` | 远程 Agent 触发与监控 |

### MCP 扩展

```bash
csc mcp serve     # 启动 MCP 服务器
csc mcp add       # 添加 MCP 服务
csc mcp list      # 列出已配置服务
```

---

## Feature Flag 体系

所有特性通过 `FEATURE_<FLAG_NAME>=1` 环境变量按需启用。

### 默认启用（Dev + Build）

| 优先级 | Feature | 说明 |
|--------|---------|------|
| 核心 | `BUDDY` | 宠物伙伴 |
| 核心 | `TRANSCRIPT_CLASSIFIER` | 会话自动分类 |
| 核心 | `BRIDGE_MODE` | Bridge / Remote Control |
| 核心 | `AGENT_TRIGGERS_REMOTE` | 远程 Agent 触发 |
| 核心 | `CHICAGO_MCP` | Computer Use MCP |
| 核心 | `VOICE_MODE` | 语音输入 |
| 监控 | `SHOT_STATS` | 统计面板 |
| 监控 | `PROMPT_CACHE_BREAK_DETECTION` | 缓存断裂检测 |
| 监控 | `TOKEN_BUDGET` | Token 预算管理 |
| P0 | `AGENT_TRIGGERS` | 本地 Agent 触发 |
| P0 | `ULTRATHINK` | 深度思考 |
| P0 | `BUILTIN_EXPLORE_PLAN_AGENTS` | 内置探索/规划 Agent |
| P0 | `LODESTONE` | Lodestone |
| P1 | `EXTRACT_MEMORIES` | 记忆提取 |
| P1 | `VERIFICATION_AGENT` | 验证 Agent |
| P1 | `KAIROS_BRIEF` | Kairos 摘要 |
| P1 | `AWAY_SUMMARY` | 离线摘要 |
| P1 | `ULTRAPLAN` | 超级规划 |
| P2 | `DAEMON` | 守护进程 |

### 按需启用

| Feature | 说明 |
|---------|------|
| `BG_SESSIONS` | 后台会话 (ps/logs/attach/kill) |
| `FORK_SUBAGENT` | Fork 子 Agent 并行执行 |
| `PROACTIVE` | 主动式 Agent |
| `KAIROS` | Kairos 全功能 |
| `COORDINATOR_MODE` | Coordinator 协调模式 |
| `TEMPLATES` | Template 任务系统 |
| `SSH_REMOTE` | SSH 远程连接 |
| `WEB_BROWSER_TOOL` | 浏览器自动化 |
| `WORKFLOW_SCRIPTS` | 工作流脚本 |
| `BYOC_ENVIRONMENT_RUNNER` | BYOC 环境运行器 |
| `SELF_HOSTED_RUNNER` | 自托管运行器 |

```bash
FEATURE_FORK_SUBAGENT=1 FEATURE_BG_SESSIONS=1 bun run dev
```

---

## Workspace Packages

| Package | 说明 | 状态 |
|---------|------|------|
| `@ant/ink` | Forked Ink 终端 UI 框架 | 完整 |
| `@ant/computer-use-mcp` | Computer Use MCP Server | 完整 |
| `@ant/computer-use-input` | 键鼠模拟 (多平台 backend) | 完整 |
| `@ant/computer-use-swift` | 截图 + 应用管理 (多平台) | 完整 |
| `remote-control-server` | 自托管 RCS (Docker + Web UI) | 完整 |
| `audio-capture-napi` | 原生音频捕获 | 已恢复 |
| `image-processor-napi` | 图像处理 | 已恢复 |
| `color-diff-napi` | 颜色差异计算 | 完整 |
| `modifiers-napi` | 键盘修饰键检测 | Stub |
| `url-handler-napi` | URL scheme 处理 | Stub |

---

## 测试与质量

| 指标 | 数据 |
|------|------|
| 测试数量 | 2430 tests |
| 测试文件 | 137 files |
| 断言调用 | 3982 expect() |
| 框架 | `bun:test` |
| Lint | Biome |
| Format | Biome |

```bash
bun test                                       # 全量测试
bun test src/utils/__tests__/hash.test.ts       # 单文件
bun test --coverage                            # 覆盖率
bun run lint                                    # 静态检查
bun run health                                  # 健康检查
```

---

## 开发调试

### 常用脚本

| 命令 | 说明 |
|------|------|
| `bun run dev` | 开发模式（MACRO + 全 Feature） |
| `bun run dev:inspect` | 带 Inspector 的开发模式 |
| `bun run build` | 生产构建 (code splitting) |
| `bun run lint:fix` | 自动修复 Lint |
| `bun run format` | 格式化 |
| `bun run health` | 健康检查 |
| `bun run generate:skills` | 生成 Skills 索引 |
| `bun run rcs` | 启动 RCS |
| `bun run docs:dev` | 文档站 (Mintlify) |

### VS Code 调试

```bash
bun run dev:inspect   # → ws://localhost:8888/xxx
# VS Code: F5 → "Attach to Bun (TUI debug)"
```

---

## Teach Me

内置苏格拉底式问答学习：

```bash
/teach-me CoStrict 架构
/teach-me Tool 系统 --level beginner --resume
```

---

## 许可证与社区

- 源码仓库：[zgsm-sangfor/csc](https://github.com/zgsm-sangfor/csc)
- 问题反馈：[zgsm-sangfor/csc/issues](https://github.com/zgsm-sangfor/csc/issues)
