# 编码约定（Coding Conventions）

**分析日期（Analysis Date）：** 2026-05-17

## 语言与运行时（Language & Runtime）

**主要语言（Primary）：** TypeScript（ESNext 目标）搭配 ESM（`package.json` 中配置 `"type": "module"`）
**JSX：** 通过 `tsconfig.json` 使用 `react-jsx` 转换
**严格模式（Strict mode）：** `tsconfig.json` 中设置为 `"strict": false` — 代码库未启用完整的 TypeScript 严格模式，但各个文件广泛使用了显式类型注解（explicit type annotations）。

## 命名模式（Naming Patterns）

**文件（Files）：**
- 组件/模块文件使用大驼峰命名（PascalCase）：`App.tsx`、`ThemeProvider.tsx`、`Task.ts`、`ShellError.ts`
- 工具文件使用小驼峰命名（camelCase）：`costHook.ts`、`setup.ts`、`context.ts`
- 命令文件夹使用短横线命名（kebab-case）：`commands/install-slack-app/`、`commands/bridge-kick.ts`
- `.ts` 用于纯 TypeScript 文件，`.tsx` 用于包含 JSX/React 元素的文件

**函数（Functions）：**
- 所有函数和方法使用小驼峰命名（camelCase）：`formatTotalCost()`、`getStoredSessionCosts()`、`saveCurrentSessionCosts()`
- 访问器函数以 `get` 为前缀：`getSessionId()`、`getProjectRoot()`、`getTotalCostUSD()` — 参见 `src/bootstrap/state.ts`
- 谓词函数以 `is` 为前缀：`isTerminalTaskStatus()`、`isAbortError()`、`isENOENT()`、`isBareMode()` — 参见 `src/Task.ts`、`src/utils/errors.ts`
- React 钩子（hooks）函数以 `use` 为前缀：`useCostSummary()`、`useAfterFirstRender()` — 参见 `src/costHook.ts`、`src/hooks/useAfterFirstRender.ts`
- 异步函数（async functions）使用 `async function` 关键字声明：`export async function setup(...)` — 参见 `src/setup.ts`
- React 组件使用普通函数声明（plain function declarations）：`export function App(t0) {...}` — 参见 `src/components/App.tsx`

**变量（Variables）：**
- 所有变量和参数使用小驼峰命名（camelCase）：`sessionId`、`modelUsage`、`cwd`、`permissionMode`
- 优先使用 `const`，而非 `let`；`let` 仅用于可变计数器或累加器（mutable counters or accumulators）
- 常量使用大写蛇形命名（UPPER_SNAKE_CASE）：`MAX_STATUS_CHARS`、`TASK_ALPHABET`、`AGENT_ID_PATTERN`

**类型（Types）：**
- 所有类型/接口名称使用大驼峰命名（PascalCase）：`TaskType`、`TaskStatus`、`TaskContext`、`MacroConfig`、`MissingImport`
- 优先使用类型别名（type aliases）：`export type TaskType = 'local_bash' | 'local_agent' | ...`
- 接口（interfaces）用于包含方法的对象形状：`interface Props { children: React.ReactNode }`
- 使用品牌类型（branded types）增强类型安全：`SessionId = string & { readonly __brand: 'SessionId' }` — 参见 `src/types/ids.ts`
- 自定义错误类以 `Error` 为后缀：`ClaudeError`、`AbortError`、`ShellError`、`ConfigParseError` — 参见 `src/utils/errors.ts`
- 可辨识联合类型（discriminated union types）以 `Kind` 为后缀：`AxiosErrorKind = 'auth' | 'timeout' | 'network' | 'http' | 'other'`

## 代码风格（Code Style）

**格式化（Formatting）：**
- 项目根目录未检测到 Prettier 配置
- 字符串使用单引号：`import { existsSync } from 'fs'`
- 分号（Semicolons）：在 TypeScript 源文件中省略（`src/` 目录下的主要模式）；在编译/恢复后的输出文件中存在
- 缩进：2 个空格（根据编译输出的 `tsconfig.json` 和源代码的一致性推断）
- 尾随逗号（Trailing commas）：在多行对象/数组字面量中使用

**代码检查（Linting）：**
- 项目根目录未检测到 ESLint 或 Biome 配置
- 内联 eslint 禁用注释频繁出现，用于自定义规则：
  - `/* eslint-disable custom-rules/no-process-exit */` — 参见 `src/setup.ts`
  - `/* eslint-disable @typescript-eslint/no-require-imports */` — 参见 `src/tools.ts`
  - `/* eslint-disable custom-rules/no-process-env-top-level */` — 参见 `src/tools.ts`
  - `/* biome-ignore lint/suspicious/noConsole */` — 参见 `src/setup.ts`
  - `// biome-ignore-all assist/source/organizeImports` — 参见 `src/tools.ts`、`src/commands.ts`
  - 这些注释暗示存在一个内部 lint 系统（Ant 专用），但在当前的恢复源代码树中不存在

## 导入组织（Import Organization）

**顺序（观察到的模式）：**
1. 来自 `node_modules` 的外部包（按来源分组）：`import { feature } from 'bun:bundle'`、`import chalk from 'chalk'`、`import { z } from 'zod/v4'`
2. 使用 `src/` 前缀的内部路径别名：`import { getCwd } from 'src/utils/cwd.js'`
3. 使用 `./` 或 `../` 的相对导入：`import { formatTotalCost } from './cost-tracker.js'`
4. 仅类型导入（type-only imports）通过 `import type {...}` 分组 — 与值导入分开：`import type { AgentId } from './types/ids.js'`

**路径别名（Path Aliases）：**
- `src/*` 通过 `tsconfig.json` 中的 paths 配置映射到 `./src/*`
- 广泛使用：`import { getCwd } from 'src/utils/cwd.js'`
- 同一目录或直接父目录内优先使用相对路径

**扩展名（Extensions）：**
- 所有本地导入使用显式的 `.js` 扩展名，即使源文件是 `.ts`/`.tsx`：`import { App } from './components/App.js'`
- 这是使用 TypeScript 的 ESM 的标准做法（输出时将 `.ts` 映射到 `.js`）

## 错误处理（Error Handling）

**模式（Patterns）：**

1. **自定义错误类（Custom Error Classes）**（`src/utils/errors.ts`）：
   ```typescript
   export class ClaudeError extends Error {
     constructor(message: string) {
       super(message)
       this.name = this.constructor.name
     }
   }

   export class ShellError extends Error {
     constructor(
       public readonly stdout: string,
       public readonly stderr: string,
       public readonly code: number,
       public readonly interrupted: boolean,
     ) {
       super('Shell command failed')
       this.name = 'ShellError'
     }
   }
   ```

2. **错误辅助函数（Error Helper Functions）**（`src/utils/errors.ts`）：
   - `errorMessage(e: unknown): string` — 从任何抛出的值中提取消息
   - `toError(e: unknown): Error` — 将抛出值规范化为 Error 实例
   - `isAbortError(e: unknown): boolean` — 跨 SDK/AbortController/自定义错误类检查中止形状的错误
   - `isENOENT(e: unknown): boolean` — 检查文件系统未找到错误
   - `classifyAxiosError(e: unknown): { kind, status, message }` — 将 axios 错误归类为 'auth'/'timeout'/'network'/'http'/'other'
   - `shortErrorStack(e: unknown, maxFrames = 5): string` — 截断的堆栈信息，供模型使用
   - `getErrnoCode(e: unknown): string | undefined` — 提取 errno 代码，无需类型转换

3. **遥测安全错误（Telemetry-safe errors）**：
   ```typescript
   export class TelemetrySafeError_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS extends Error {
     readonly telemetryMessage: string
     constructor(message: string, telemetryMessage?: string) {
       super(message)
       this.name = 'TelemetrySafeError'
       this.telemetryMessage = telemetryMessage ?? message
     }
   }
   ```
   长类名模式（`_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS`）用于确认开发者已验证 PII 安全性 — 该模式也出现在 `AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS` 中。

4. **捕获点模式（Catch-site pattern）**：捕获的错误通过 `logError(error)`（来自 `src/utils/log.js`）记录，而非重新抛出 — 参见 `src/context.ts:108`、`src/setup.ts:154`。

5. **守卫模式（Guard pattern）**：在函数开头检查 `process.env.NODE_ENV === 'test'`，在测试期间短路执行 — 参见 `src/context.ts:37`。

## 日志记录（Logging）

**框架（Framework）：** `src/utils/log.js` 和 `src/utils/diagLogs.js` 中的自定义工具

**模式（Patterns）：**
- `logForDiagnosticsNoPII(level: string, event: string, attrs?: object): void` — 结构化诊断日志，保证不含 PII
- `logError(error: unknown): void` — 统一的错误日志记录入口
- `logEvent(eventName: string, attrs: object): void` — 分析事件发送
  - 这是主要的遥测/分析日志记录路径，在整个代码库中被广泛使用
  - 分析元数据值使用特殊的品牌类型：`AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS`
- `logForDebugging(...args): void` — 调试级别日志记录（在生产环境中可能被去除）
- 通过 `logForDiagnosticsNoPII('info', 'event_name', { duration_ms: ... })` 模式进行计时，配合手动 `Date.now()` 开始/结束配对 — 参见 `src/context.ts`、`src/setup.ts`

## 注释（Comments）

**何时添加注释（When to Comment）：**
- 公共 API 函数和复杂逻辑使用 JSDoc/TSDoc
- 非显而易见的行为、性能说明和恢复工作区使用内联注释
- 解释 WHY（而非 what）的注释 — 例如 `src/bootstrap/state.ts` 中的 "DO NOT ADD MORE STATE HERE - BE JUDICIOUS WITH GLOBAL STATE"
- 关于导入顺序的警告：`// biome-ignore-all assist/source/organizeImports: ANT-ONLY import markers must not be reordered`

**JSDoc/TSDoc：**
- 用于带有参数、返回值和 `@example` 块的导出函数
- 用于解释用途的类级别文档
- 来自 `src/utils/errors.ts` 的示例：
  ```typescript
  /**
   * Extract error message + top N stack frames from an unknown error.
   * Use when the error flows to the model as a tool_result...
   */
  ```

**模式化注释（Pattern comments）：**
- `// Dead code elimination:` — 解释条件性 require
- `// IMPORTANT:` — 关键顺序约束
- `// Note:` — 非显而易见的行为细节

## 函数设计（Function Design）

**规模（Size）：** 函数通常规模较小到适中（10-50 行）。较大函数存在但属于例外情况（例如 `src/setup.ts` 中的 `setup()` 约 477 行）。

**参数（Parameters）：** 对于复杂的参数列表（React props、配置对象），通过解构对象（destructured objects）使用命名参数。简单函数使用位置参数（positional arguments，最多 2-3 个）。示例来自 `src/setup.ts`：
```typescript
export async function setup(
  cwd: string,
  permissionMode: PermissionMode,
  allowDangerouslySkipPermissions: boolean,
  worktreeEnabled: boolean,
  ...
```

**返回值（Return Values）：** 所有导出函数具有类型化的返回值。函数要么返回数据，要么返回 `void`。错误情况使用 `null` 返回值（而非 `undefined`）表示"无结果"：`getStoredSessionCosts(...): StoredCostState | undefined`。

## 模块设计（Module Design）

**导出（Exports）：**
- 优先使用命名导出（named exports），而非默认导出（default exports）
- 在模块末尾通过 `export { ... }` 块进行桶导出（barrel exports）— 参见 `src/cost-tracker.ts:49-69`
- React 组件以命名函数声明形式导出：`export function App(...)`
- 类型在声明时内联导出，或通过 `export type` 单独导出

**桶文件（Barrel Files）：**
- `src/ink.ts` 作为公共 API 表面，重新导出 `src/ink/` 内部模块
- 各个工具/命令模块在中央注册表中注册：`src/tools.ts`、`src/commands.ts`、`src/tasks.ts`

**状态管理（State Management）：**
- 全局状态位于 `src/bootstrap/state.ts`，使用模块作用域的 `STATE` 对象，配合显式的 getter/setter 函数
- 状态变更通过命名函数进行（而非直接属性访问）
- 使用对象展开（object spread）的不可变更新模式：`...current`
- React 状态通过上下文提供者（context providers）管理：`AppStateProvider`、`StatsProvider`、`FpsMetricsProvider`

## 条件性导入（Conditional Requires）

受特性开关（feature flags）控制的工具和命令使用特定的 `require()` 模式，配合 eslint-disable 注释实现死代码消除（dead code elimination）：
```typescript
/* eslint-disable @typescript-eslint/no-require-imports */
const LocalWorkflowTask = feature('WORKFLOW_SCRIPTS')
  ? require('./tasks/LocalWorkflowTask/LocalWorkflowTask.js').LocalWorkflowTask
  : null
/* eslint-enable @typescript-eslint/no-require-imports */
```
该模式出现在 `src/tools.ts`、`src/commands.ts` 和 `src/tasks.ts` 中。

---

*约定分析日期：2026-05-17*
