<!-- refreshed: 2026-05-17 -->
# 架构

**分析日期：** 2026-05-17

## 系统概述

```text
┌───────────────────────────────────────────────────────────────────────┐
│                        CLI 入口点                                       │
│  `src/dev-entry.ts` → `src/entrypoints/cli.tsx`                      │
│  快速路径路由：--version, --help, bridge, daemon, chrome 等           │
└──────────────────────────┬────────────────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────────────────┐
│                        初始化                                           │
│  `src/entrypoints/init.ts`（记忆化单例）                                │
│  配置加载、认证、遥测设置、工作目录设置、MCP 启动                      │
│  `src/setup.ts` — 会话引导                                              │
└──────────────────────────┬────────────────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────────────────┐
│                     应用程序外壳                                        │
│  `src/main.tsx` — 带有 AppStateProvider 的 React 根组件                │
│  `src/screens/` — REPL.tsx, ResumeConversation.tsx, Doctor.tsx        │
│  通过 `src/ink.ts` 渲染 Ink 终端 UI                                    │
└──────┬──────────────────────┬──────────────────────┬──────────────────┘
       │                      │                      │
       ▼                      ▼                      ▼
┌──────────────┐   ┌──────────────────┐   ┌──────────────────────┐
│ 命令          │   │ 查询引擎           │   │ 工具与任务             │
│ `commands.ts`│   │ `QueryEngine.ts` │   │ `Tool.ts` / `Task.ts`│
│ 80+ 个子     │   │ 主消息循环：        │   │                      │
│ 命令在        │   │ query() 循环 →    │   │ 40+ 工具             │
│ `commands/`  │   │ API 调用 →        │   │ 实现在               │
│              │   │ 工具执行 →         │   │ `src/tools/`         │
│              │   │ 压缩 → 重复        │   │                      │
│              │   │                   │   │ 6 种任务类型在        │
│              │   │                   │   │ `src/tasks/`         │
└──────────────┘   └──────────────────┘   └──────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────────────────┐
│                     服务层                                              │
│  `src/services/api/` — Anthropic API 客户端 + 重试 + 认证              │
│  `src/services/mcp/` — MCP 服务器管理                                  │
│  `src/services/lsp/` — LSP 集成                                        │
│  `src/services/analytics/` — 遥测 + GrowthBook 特性标记                │
│  `src/services/oauth/` — OAuth 认证                                    │
│  `src/services/compact/` — 上下文压缩                                  │
└──────────────────────────┬────────────────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────────────────┐
│                   工具与平台层                                           │
│  `src/utils/` — 约 100+ 个工具模块（git、认证、shell、文件等）         │
│  `src/bootstrap/state.ts` — 进程级全局单例                             │
│  `src/state/` — 基于 React Context 的 React AppState 存储              │
│  `src/bridge/` — 远程桥接模式（--remote-control）                      │
└───────────────────────────────────────────────────────────────────────┘
```

## 组件职责

| 组件 | 职责 | 文件 |
|-----------|----------------|------|
| 开发入口（Dev Entry） | 版本/帮助检查、缺失导入扫描、路由到实际入口 | `src/dev-entry.ts` |
| CLI 入口（CLI Entry） | 快速路径路由（version、bridge、daemon、chrome 等） | `src/entrypoints/cli.tsx` |
| 初始化（Init） | 单例初始化（配置、认证、遥测、MCP） | `src/entrypoints/init.ts` |
| 引导（Setup） | 会话引导、工作目录、工作树、权限模式 | `src/setup.ts` |
| 应用外壳（App Shell） | 带有 AppStateProvider 的 React 根组件 | `src/main.tsx` |
| QueryEngine | 主对话循环：提示词 -> API -> 工具 -> 压缩 -> 重复 | `src/QueryEngine.ts` |
| query.ts | 核心查询循环实现 | `src/query.ts` |
| 命令（Commands） | 斜杠命令注册表（80+ 命令） | `src/commands.ts` |
| 工具（Tools） | 基础工具抽象与注册表 | `src/Tool.ts` |
| 工具注册表（Tools Registry） | 带特性标记门控的工具实例化 | `src/tools.ts` |
| 任务（Tasks） | 后台任务抽象注册表 | `src/tasks.ts`, `src/Task.ts` |
| 状态引导（State Bootstrap） | 进程级全局可变单例 | `src/bootstrap/state.ts` |
| 状态存储（State Store） | 基于 React Context 的不可变应用状态存储 | `src/state/AppState.tsx`, `src/state/store.ts` |
| Ink | 自定义终端 UI 框架（Ink 的分支） | `src/ink/` |
| 设计系统（Design System） | 主题化 UI 组件（Box、Text、ThemeProvider） | `src/components/design-system/` |
| CLI 处理器（CLI Handlers） | 认证、MCP、代理、插件、自动模式子命令 | `src/cli/handlers/` |
| CLI 传输层（CLI Transports） | 用于结构化输出的 SSE、混合、串行批处理传输 | `src/cli/transports/` |
| 桥接（Bridge） | 用于远程控制的远程桥接模式 | `src/bridge/` |
| API 客户端（API Client） | 带重试、日志记录、认证的 Anthropic Messages API | `src/services/api/` |
| 分析（Analytics） | 遥测事件、GrowthBook 特性门控 | `src/services/analytics/` |

## 模式概览

**总体：** 采用领域驱动目录分组（Domain-Driven Directory Grouping）和特性标记死代码消除（Feature-Flag Dead Code Elimination）的模块化单体（Modular Monolith）。

**关键特性：**
- **通过 `bun:bundle` 的 `feature()` 进行特性标记门控（Feature-Flag Gating）** — 构建时的死代码消除会从外部构建中移除内部专用特性。特性名称如 `'KAIROS'`、`'BRIDGE_MODE'`、`'VOICE_MODE'`、`'DAEMON'`、`'COORDINATOR_MODE'`。
- **条件特性的动态 `require()`** — 内部专用特性使用 `const X = feature('FLAG') ? require('./path') : null`，而非静态导入，这样外部构建可以通过摇树优化（Tree-Shaking）移除它们。
- **双层状态管理（Two-Tier State Management）** — 模块级全局单例（Module-Level Global Singleton）（`src/bootstrap/state.ts`）用于进程全局状态，React Context 存储（`src/state/`）用于 UI 响应式状态。
- **自定义 Ink 框架** — 代码库携带了 Ink（终端 React 渲染器）的一个分支副本，位于 `src/ink/` 中，包含用于事件、焦点管理、动画帧和标签状态的自定义扩展。
- **按领域分目录分组（Domain-Per-Directory Grouping）** — 工具位于 `src/tools/`，命令位于 `src/commands/`，服务位于 `src/services/`，组件位于 `src/components/`。
- **使用 `type: "module"` 的 ESM 模块** — 所有导入使用 `.js` 扩展名（TypeScript moduleResolution: bundler）。

## 层次结构

**CLI 入口层：**
- 目的：解析 CLI 参数并快速路径路由到专用处理器
- 位置：`src/entrypoints/cli.tsx`
- 包含：标记解析、桥接路由、守护进程路由、chrome/chrome-mcp 路由
- 依赖：`src/entrypoints/init.ts`、`src/utils/`、`src/bridge/`、`src/daemon/`
- 使用者：`src/dev-entry.ts`（开发工作区）或打包后的二进制文件

**初始化层：**
- 目的：一次性异步设置 — 配置、认证、遥测、OpenTelemetry、策略、MCP
- 位置：`src/entrypoints/init.ts`
- 包含：配置加载、OAuth 令牌获取、OpenTelemetry 计量器/日志记录器/追踪器设置、GrowthBook 初始化、策略限制、远程托管设置、工作树创建、Unix 域套接字消息传递
- 依赖：`src/bootstrap/state.ts`、`src/utils/`、`src/services/`
- 使用者：会话开始前的 CLI 入口

**应用层（React/Ink TUI）：**
- 目的：通过 Ink 使用 React 进行终端 UI 渲染
- 位置：`src/main.tsx`、`src/screens/`、`src/components/`
- 包含：REPL 屏幕、命令对话框、权限对话框、消息显示、提示输入、设计系统组件
- 依赖：`src/ink/`（自定义 Ink 分支）、React、`src/state/`、`src/hooks/`
- 使用者：最终用户交互

**查询循环层：**
- 目的：核心对话处理 — 用户输入、API 调用、工具执行、上下文压缩
- 位置：`src/QueryEngine.ts`、`src/query.ts`
- 包含：消息处理、API 流式传输、工具分发、普通压缩/响应式压缩（Compact/Reactive Compact）、上下文折叠（Context Collapse）
- 依赖：`src/services/api/`、`src/Tool.ts`、`src/tools/`、`src/services/compact/`、`src/context.ts`
- 使用者：主 TUI 渲染循环

**命令层：**
- 目的：斜杠命令分发（`/init`、`/commit`、`/config` 等）
- 位置：`src/commands.ts`、`src/commands/`
- 包含：80+ 个命令实现，每个在其各自的子目录中
- 依赖：`src/services/`、`src/utils/`、`src/hooks/`
- 使用者：提示输入处理

**工具层：**
- 目的：供模型调用的工具定义（BashTool、FileEditTool、GlobTool 等）
- 位置：`src/Tool.ts`、`src/tools.ts`、`src/tools/`
- 包含：40+ 个工具实现、工具权限系统、工具测试框架
- 依赖：`src/utils/`、`src/services/`、`src/types/`
- 使用者：QueryEngine 在工具使用轮次中调用

**服务层：**
- 目的：业务逻辑和外部集成
- 位置：`src/services/`
- 包含：API 客户端（`src/services/api/`）、MCP 服务器管理（`src/services/mcp/`）、LSP（`src/services/lsp/`）、分析/遥测（`src/services/analytics/`）、OAuth（`src/services/oauth/`）、上下文压缩（`src/services/compact/`）、代理摘要、记忆、策略限制、插件、设置同步、语音等
- 依赖：`src/utils/`、`src/constants/`、`src/types/`
- 使用者：QueryEngine、命令、工具

**桥接层：**
- 目的：远程控制服务器模式 — 通过网络提供 Claude Code 服务
- 位置：`src/bridge/`
- 包含：WebSocket 传输、JWT 认证、会话管理、轮询配置、权限回调
- 依赖：`src/utils/`、`src/services/`
- 使用者：CLI 快速路径（`--remote-control`）

**平台层：**
- 目的：横切工具、原生绑定、兼容性
- 位置：`src/utils/`、`src/vendor/`、`shims/`、`src/native-ts/`
- 包含：Git 工具、shell/终端、认证、文件操作、设置、权限、OpenTelemetry 辅助函数、NAPI 模块的原生垫片（Shim）
- 依赖：标准库、外部 npm 包
- 使用者：所有层

## 数据流

### 主要请求路径（对话轮次）

1. **用户输入** — 用户在 REPL 中输入内容（`src/screens/REPL.tsx`）。由 `processUserInput()`（`src/utils/processUserInput/`）处理。
2. **斜杠命令检查** — 如果输入以 `/` 开头，则分发给 `src/commands/` 中匹配的命令。
3. **消息准备** — `query()`（位于 `src/query.ts`）构建消息数组，从 `src/context.ts` 预置用户上下文，追加系统上下文。
4. **API 调用** — 通过 `src/services/api/claude.ts` 将消息发送到 Anthropic Messages API。流式响应解析为 `StreamEvent[]`。
5. **工具使用处理** — 当 API 返回 `tool_use` 块时，`QueryEngine.ts` 分发给 `src/tools/` 中匹配的工具。工具执行通过 `src/hooks/useCanUseTool.ts` 进行权限检查。
6. **工具结果** — 工具执行产生 `ToolResultBlockParam`；结果追加到消息中并发送回 API。
7. **上下文压缩** — 当令牌数超过阈值时，`src/services/compact/` 对消息进行压缩。可使用响应式压缩（Reactive Compaction）（`src/services/compact/reactiveCompact.ts`）或上下文折叠（Context Collapse）（`src/services/contextCollapse/`）。
8. **输出渲染** — 助手响应通过 Ink 组件（`src/components/messages/`）流式渲染。

### 桥接模式（远程控制）

1. 客户端发起与桥接服务器的 WebSocket 连接。
2. `src/bridge/bridgeMain.ts` 管理连接生命周期。
3. `src/bridge/inboundMessages.ts` / `src/bridge/inboundAttachments.ts` 处理传入数据。
4. `src/bridge/remoteBridgeCore.ts` 实现核心桥接协议。
5. 通过 `src/bridge/createSession.ts`、`src/bridge/peerSessions.ts` 管理会话。

**状态管理：**
- **进程级全局状态** — `src/bootstrap/state.ts`：一个模块级的 `State` 单例，保存项目根目录、会话 ID、遥测计量器、计数器、API 请求状态、插件状态、定时任务等。通过 getter/setter 函数访问。在所有层中使用。
- **UI 响应式状态** — `src/state/AppState.tsx` + `src/state/store.ts`：基于 React Context 的存储，暴露 `getState()`、`setState()`、`subscribe()`。保存设置、模型配置、工具权限上下文、任务、团队、通知状态、推测状态等。
- **文件状态缓存** — `src/utils/fileStateCache.ts`：跟踪文件内容以进行差异检测。

## 关键抽象

**Tool**（`src/Tool.ts`）：
- 目的：定义模型可以调用的工具。每个工具都有名称、输入 JSON 模式（Schema）、描述和异步 `run()` 方法。
- 模式：基于类，实现 `Tool` 接口。通过 `ToolPermissionContext` 实现权限钩子（Hook）。
- 关键实现：`BashTool`、`FileEditTool`、`FileReadTool`、`FileWriteTool`、`GlobTool`、`GrepTool`、`AgentTool`、`SkillTool`、`MCPTool`、`WebSearchTool`、`TaskCreateTool` 等。

**Task**（`src/Task.ts`）：
- 目的：后台进程抽象。任务具有类型（`local_bash`、`local_agent`、`remote_agent`、`dream`、`local_workflow`、`monitor_mcp`）和状态生命周期（`pending` -> `running` -> `completed`/`failed`/`killed`）。
- 模式：基于接口，包含 `kill()` 方法。任务在 `src/tasks.ts` 中自行注册。
- 关键实现：`LocalShellTask`、`LocalAgentTask`、`RemoteAgentTask`、`DreamTask`。

**Command**（`src/commands.ts`）：
- 目的：斜杠命令定义。每个命令都有名称、描述、isEnabled 检查和参数规范。
- 模式：基于对象的 `Command` 类型。
- 范围：在 `src/commands.ts` 中注册了 80+ 个命令。

**AppState Store**（`src/state/store.ts`）：
- 目的：带有订阅（Subscribe）模式的简单可观察状态容器。
- 模式：原生 JS 存储 — `getState()`、`setState(updater)`、`subscribe(listener)`。通过 `AppStateProvider` 包装在 React Context 中。

**CLI Transport**（`src/cli/transports/Transport.ts`）：
- 目的：用于结构化/非交互模式的输出传输抽象。
- 实现：`SSETransport`、`HybridTransport`、`SerialBatchEventUploader`。

**Ink Framework**（`src/ink/`）：
- 目的：自定义终端 React 渲染器。Ink 库的分支，带有自定义扩展。
- 包含：DOM 抽象、事件系统（输入、点击、焦点）、布局（Box、Text）、钩子（Hook）（动画、间隔、选择、输入）。

## 入口点

**开发入口（Dev Entry）（`src/dev-entry.ts`）：**
- 位置：`src/dev-entry.ts`
- 触发方式：`bun run dev` 或 `bun run start`
- 职责：扫描缺失的相对导入，处理 `--version` 和 `--help`，然后路由到 `src/entrypoints/cli.tsx`

**CLI 入口（CLI Entry）（`src/entrypoints/cli.tsx`）：**
- 位置：`src/entrypoints/cli.tsx`
- 触发方式：普通 CLI 调用
- 职责：`--version`、`--dump-system-prompt`、`--claude-in-chrome-mcp`、`--computer-use-mcp`、`--daemon-worker`、`--remote-control`/`bridge`、`daemon`、`ps`/`logs`/`attach`/`kill` 的快速路径路由，最后回退到 `init.ts` -> `main.tsx`。

**SDK 入口（SDK Entry）（`src/entrypoints/sdk/`）：**
- 位置：`src/entrypoints/sdk/`
- 触发方式：编程式 SDK 使用
- 职责：SDK 类型和 Agent SDK 运行时集成。

**MCP 入口（MCP Entry）（`src/entrypoints/mcp.ts`）：**
- 位置：`src/entrypoints/mcp.ts`
- 触发方式：作为 MCP 服务器运行时
- 职责：用于与 MCP 兼容工具集成的 MCP 服务器入口点。

**初始化入口（Init Entry）（`src/entrypoints/init.ts`）：**
- 位置：`src/entrypoints/init.ts`
- 触发方式：启动时调用一次
- 职责：单例异步初始化 — 配置、认证、OpenTelemetry、遥测、策略限制、MCP 预连接、工作树设置。已记忆化，因此只运行一次。

## 架构约束

- **线程模型：** 单线程事件循环（Bun/Node.js）。后台任务作为子进程或通过 Task 系统的进程内工作线程生成。原生 NAPI 模块（`color-diff-napi`、`modifiers-napi`、`url-handler-napi`）在 `src/native-ts/` 中以 TypeScript 垫片（Shim）实现形式存在。
- **全局状态：** 在 `src/bootstrap/state.ts` 中存在大量的模块级全局状态。单例模式用于会话 ID、项目根目录、成本跟踪、遥测计量器、API 请求状态。必须谨慎管理，以避免在测试/并行上下文中出现问题。
- **循环导入：** 在许多文件中都有显式文档说明。例如，`src/coordinator/coordinatorMode.ts` 记录了循环依赖路径："filesystem -> permissions -> ... -> coordinatorMode"。代码库使用多种技术来打破循环：仅类型导入（Type-Only Import）、动态 `require()` 和依赖注入（例如，从 QueryEngine 注入的 `scratchpadDir` 参数）。
- **特性标记死代码消除（Feature-Flag DCE）：** 来自 `bun:bundle` 的 `feature('FLAG_NAME')` 函数支持构建时死代码消除。整个条件块会从外部构建中移除。这被广泛使用——每个内部专用特性都通过这种方式门控。
- **模块求值顺序敏感性：** 某些模块（例如 `src/entrypoints/cli.tsx` 第 21-26 行）记录说明环境变量必须在特定导入之前设置，因为模块在导入时会捕获值到模块级 `const` 中。`ABLATION_BASELINE` 特性标记块必须在 BashTool/AgentTool 导入之前运行。
- **引导隔离（Bootstrap Isolation）：** 一条 lint 规则（`custom-rules/bootstrap-isolation`）限制了 `src/bootstrap/state.ts` 可以导入的内容——它必须保持在导入 DAG 中的叶子节点地位。路径别名导入通过 `eslint-disable` 注释显式记录了这一点。

## 反模式

### 特性标记竞态条件（Feature-Flag Data Races）

**现象：** 特性标记通过 `src/services/analytics/growthbook.js` 中的 `checkStatsigFeatureGate_CACHED_MAY_BE_STALE` 进行检查，在 GrowthBook 完全初始化之前的启动早期阶段可能返回过期的值。
**问题原因：** 在 GrowthBook 初始化完成之前读取标记的代码路径可能会观察到不正确的默认值，导致运行间行为不一致。
**正确做法：** 在适用情况下，使用 `await waitForPolicyLimitsToLoad()` 或类似的初始化 Promise 等待后再读取标记，如桥接模式在 `src/entrypoints/cli.tsx:155` 中所做的那样。

### 导入时的模块级常量捕获（Module-Level Const Capture at Import Time）

**现象：** 多个模块（例如 BashTool、AgentTool、PowerShellTool）在导入时将 `process.env.DISABLE_BACKGROUND_TASKS` 捕获到模块级 `const` 中，这意味着环境变量必须在导入发生之前设置。
**问题原因：** 如果环境变量设置较晚（在导入已经求值之后），模块级的 `const` 会保留旧值。这会导致启动顺序改变行为的微妙错误。
**正确做法：** 使用 getter 函数或惰性访问（Lazy Access）在调用时读取环境变量，而不是在导入时读取，或者像 `src/entrypoints/cli.tsx:19` 所做的那样显式记录导入顺序要求。

## 错误处理

**策略：** 混合方法 — 某些错误使用显式的 try/catch，其他使用哨兵值（Sentinel Value）（返回 null、空数组）。API 错误通过 `src/services/api/errors.ts` 中的 `categorizeRetryableAPIError()` 处理。提示过长的错误通过 `isPromptTooLongMessage()` 检测。

**模式：**
- `src/utils/errors.ts` 中的 `errorMessage(err)` 工具函数，用于一致的错误字符串提取
- `src/services/api/withRetry.ts` 中的 `FallbackTriggeredError`，用于重试耗尽
- `src/utils/log.ts` 中的 `logError()`，用于带有内存存储的错误日志记录
- `ConfigParseError`，用于配置解析失败
- `ImageSizeError` / `ImageResizeError`，用于图像验证
- 通过 CLI 处理器中的 `process.exit(1)` 退出进程（`src/cli/handlers/auth.ts` 等）

## 横切关注点

**日志记录：** 使用 `src/utils/log.ts`（`logError`）、`src/utils/debug.ts`（`logForDebugging`、`logAntError`）、`src/utils/diagLogs.ts`（`logForDiagnosticsNoPII`）。通过 `src/utils/telemetry/` 进行 OpenTelemetry 日志记录。

**验证：** 使用 Zod 模式（Schema）进行运行时数据验证（依赖）。使用 JSON Schema 进行工具输入验证。使用 `ajv` 进行 JSON Schema 验证。

**认证：** 基于 OAuth，通过 Anthropic API。在 `src/services/oauth/` 中实现，令牌管理在 `src/utils/auth.ts` 中。还支持通过 `src/utils/authFileDescriptor.ts` / `src/utils/authPortable.ts` 进行 API 密钥认证。

**配置：** 从 `.claude/settings.json`（项目级）和全局配置加载设置。通过 `src/utils/config.ts` 和 `src/utils/settings/` 管理。支持通过 `src/utils/settings/mdm/` 进行 MDM 托管设置。

**权限：** 用于工具执行的细粒度权限系统。在 `src/utils/permissions/` 中实现，带有用于自动批准的 YOLO 分类器。权限模式可以是：`'default'`、`'bypass'`、`'yolo'`、`'classify'`、`'acl'`。

---

*架构分析：2026-05-17*
