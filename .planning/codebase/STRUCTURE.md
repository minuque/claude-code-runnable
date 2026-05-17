# Codebase Structure

**Analysis Date:** 2026-05-17

## Directory Layout

```
claude-code/
├── src/                        # Main source code
│   ├── dev-entry.ts            # Dev workspace entry point (scans missing imports)
│   ├── main.tsx                # React/Ink application root
│   ├── commands.ts             # Slash command registry (80+ commands)
│   ├── context.ts              # System/user context generation for prompts
│   ├── query.ts                # Core conversation query loop
│   ├── QueryEngine.ts          # High-level query orchestration
│   ├── setup.ts                # Session bootstrapping
│   ├── Tool.ts                 # Base tool abstraction
│   ├── tools.ts                # Tool instantiation with feature flags
│   ├── Task.ts                 # Base task abstraction
│   ├── tasks.ts                # Task registration
│   ├── ink.ts                  # Ink framework re-exports with ThemeProvider
│   ├── entrypoints/            # Application entry points
│   │   ├── cli.tsx             # Main CLI entry with fast-path routing
│   │   ├── init.ts             # Singleton initialization (config, auth, telemetry)
│   │   ├── mcp.ts              # MCP server entry point
│   │   └── sdk/                # SDK entry types and runtime
│   ├── bootstrap/
│   │   └── state.ts            # Process-wide global singleton state
│   ├── state/                  # React/UI state management
│   │   ├── AppState.tsx        # React context provider
│   │   ├── AppStateStore.ts    # State shape definition
│   │   ├── store.ts            # Observable store implementation
│   │   ├── selectors.ts        # State selectors
│   │   └── onChangeAppState.ts  # State change callback
│   ├── commands/               # Slash command implementations (~80 directories)
│   │   ├── init.js             # /init
│   │   ├── commit.js           # /commit
│   │   ├── config/             # /config
│   │   ├── help/               # /help
│   │   ├── memory/             # /memory
│   │   ├── review.js           # /review
│   │   ├── plan/               # /plan
│   │   ├── doctor/             # /doctor
│   │   ├── bridge/             # /bridge
│   │   ├── agent/              # /agent
│   │   ├── mcp/                # /mcp
│   │   └── ...                 # 70+ more commands
│   ├── tools/                  # Tool implementations (~40+ directories)
│   │   ├── BashTool/           # Bash command execution
│   │   ├── FileEditTool/       # File editing
│   │   ├── FileReadTool/       # File reading
│   │   ├── FileWriteTool/      # File writing
│   │   ├── GlobTool/           # Glob pattern matching
│   │   ├── GrepTool/           # Text search
│   │   ├── AgentTool/          # Agent sub-tasks
│   │   ├── MCPTool/            # MCP tool delegation
│   │   ├── SkillTool/          # Skill execution
│   │   ├── WebSearchTool/      # Web search
│   │   ├── WebFetchTool/       # Web fetch
│   │   ├── TaskCreateTool/     # Task creation
│   │   ├── TaskGetTool/        # Task status
│   │   ├── ...                 # 25+ more tools
│   │   ├── shared/             # Shared tool utilities
│   │   └── testing/            # Testing permission tools
│   ├── tasks/                  # Background task implementations
│   │   ├── LocalShellTask/     # Shell command background task
│   │   ├── LocalAgentTask/     # In-process agent background task
│   │   ├── RemoteAgentTask/    # Remote agent task
│   │   ├── DreamTask/          # Memory consolidation task
│   │   ├── LocalWorkflowTask/  # Workflow scripts task
│   │   └── MonitorMcpTask/     # MCP monitoring task
│   ├── services/               # Business logic and external integrations
│   │   ├── api/                # Anthropic Messages API client
│   │   │   ├── claude.ts       # Core API streaming client
│   │   │   ├── errors.ts       # API error categorization
│   │   │   ├── withRetry.ts    # Retry logic
│   │   │   └── logging.ts      # API usage logging
│   │   ├── analytics/          # Telemetry and GrowthBook feature flags
│   │   ├── mcp/                # MCP server connection management
│   │   ├── lsp/                # LSP integration
│   │   ├── oauth/              # OAuth authentication
│   │   ├── compact/            # Context compaction logic
│   │   ├── contextCollapse/    # Advanced context collapse
│   │   ├── plugins/            # Plugin system integration
│   │   ├── AgentSummary/       # Agent summary generation
│   │   ├── SessionMemory/      # Session memory service
│   │   ├── PromptSuggestion/   # Prompt suggestion engine
│   │   ├── policyLimits/       # Policy limits enforcement
│   │   ├── remoteManagedSettings/  # MDM settings
│   │   ├── settingsSync/       # Settings synchronization
│   │   ├── teamMemorySync/     # Team memory sync
│   │   ├── extractMemories/    # Memory extraction
│   │   ├── autoDream/          # Auto-dream (Kairos) service
│   │   ├── toolUseSummary/     # Tool use summary generation
│   │   ├── skillSearch/        # Skill search (experimental)
│   │   └── voice/              # Voice mode services
│   ├── components/             # React/Ink UI components
│   │   ├── design-system/      # ThemedBox, ThemedText, ThemeProvider, color
│   │   ├── messages/           # Message rendering components
│   │   ├── permissions/        # Permission dialog components (Bash, File, MCP, etc.)
│   │   ├── mcp/                # MCP UI components
│   │   ├── agents/             # Agent UI components
│   │   ├── tasks/              # Task UI components
│   │   ├── skills/             # Skills UI components
│   │   ├── memory/             # Memory UI components
│   │   ├── shell/              # Shell UI components
│   │   ├── sandbox/            # Sandbox UI components
│   │   ├── diff/               # Diff rendering
│   │   ├── HelpV2/             # Help system UI
│   │   ├── Settings/           # Settings dialogs
│   │   ├── PromptInput/        # Prompt input components
│   │   ├── Spinner/            # Loading spinner
│   │   ├── FeedbackSurvey/     # Feedback survey
│   │   ├── LogoV2/             # Logo display
│   │   ├── Passes/             # Passes UI
│   │   ├── ClaudeCodeHint/     # Hint UI
│   │   ├── DesktopUpsell/      # Desktop upsell
│   │   ├── TrustDialog/        # Trust dialog
│   │   ├── wizard/             # Wizard components
│   │   ├── ui/                 # Generic UI components
│   │   ├── hooks/              # Component-specific hooks
│   │   └── grove/              # Grove components
│   ├── hooks/                  # React hooks
│   │   ├── useCanUseTool.tsx   # Tool permission checking
│   │   ├── useSettings.ts      # Settings access
│   │   ├── useTasksV2.ts       # Task management
│   │   ├── useReplBridge.tsx   # REPL bridge integration
│   │   ├── useVoice.ts         # Voice integration
│   │   ├── toolPermission/     # Tool permission hooks
│   │   └── notifs/             # Notification hooks
│   ├── cli/                    # CLI subcommand handlers
│   │   ├── handlers/           # Auth, agent, MCP, plugin, auto-mode handlers
│   │   ├── transports/         # SSE, Hybrid, Serial transport
│   │   ├── print.ts            # Print utilities
│   │   ├── structuredIO.ts     # Structured I/O
│   │   └── remoteIO.ts         # Remote I/O
│   ├── ink/                    # Custom Ink framework (fork)
│   │   ├── components/         # Box, Text, Button, Link, AppContext, etc.
│   │   ├── hooks/              # use-input, use-app, use-stdin, etc.
│   │   ├── events/             # Click, input, focus events
│   │   ├── layout/             # Layout engine
│   │   ├── termio/             # Terminal I/O
│   │   ├── dom.ts              # DOM abstraction
│   │   ├── focus.ts            # Focus management
│   │   └── frame.ts            # Animation frame management
│   ├── bridge/                 # Remote bridge mode
│   │   ├── bridgeMain.ts       # Bridge main loop
│   │   ├── bridgeMessaging.ts  # Bridge messaging protocol
│   │   ├── remoteBridgeCore.ts # Core remote bridge logic
│   │   ├── initReplBridge.ts   # Bridge initialization
│   │   ├── createSession.ts    # Session creation
│   │   ├── peerSessions.ts     # Peer session management
│   │   ├── inboundMessages.ts  # Inbound message handling
│   │   ├── inboundAttachments.ts # Attachment handling
│   │   ├── jwtUtils.ts         # JWT utilities
│   │   ├── sessionRunner.ts    # Session runner
│   │   └── types.ts            # Bridge types
│   ├── screens/                # Top-level screen components
│   │   ├── REPL.tsx            # Main REPL screen
│   │   ├── ResumeConversation.tsx # Session resume
│   │   └── Doctor.tsx          # Doctor diagnostic screen
│   ├── constants/              # Shared constants
│   │   ├── prompts.ts          # System prompts
│   │   ├── tools.ts            # Tool constants
│   │   ├── system.ts           # System constants
│   │   ├── messages.ts         # Message constants
│   │   ├── files.ts            # File-related constants
│   │   ├── keys.ts             # Key constants
│   │   └── ...
│   ├── context/                # React contexts
│   │   ├── notifications.tsx   # Notification context
│   │   ├── mailbox.tsx         # Mailbox context
│   │   ├── voice.tsx           # Voice context
│   │   └── ...
│   ├── types/                  # TypeScript type definitions
│   │   ├── message.ts          # Message types
│   │   ├── ids.ts              # ID types
│   │   ├── permissions.ts      # Permission types
│   │   ├── plugin.ts           # Plugin types
│   │   ├── hooks.ts            # Hook types
│   │   ├── utils.ts            # Utility types
│   │   └── generated/          # Generated protobuf types
│   ├── schemas/                # Zod validation schemas
│   ├── coordinator/            # Coordinator mode (multi-agent orchestration)
│   │   ├── coordinatorMode.ts  # Coordinator mode configuration
│   │   └── workerAgent.ts      # Worker agent orchestration
│   ├── proactive/              # Proactive mode (Kairos)
│   ├── assistant/              # Assistant/Kairos mode (persistent assistant)
│   ├── buddy/                  # AI companion/pet (ASCII sprite)
│   ├── plugins/                # Plugin system
│   │   └── bundled/            # Bundled plugins
│   ├── skills/                 # Skill system
│   │   └── bundled/            # Bundled skills (claude-api, verify)
│   ├── server/                 # HTTP server components
│   ├── remote/                 # Remote session management
│   ├── ssh/                    # SSH integration
│   ├── voice/                  # Voice mode implementations
│   ├── vim/                    # Vim mode integration
│   ├── query/                  # Query utilities
│   ├── jobs/                   # Job classifier
│   ├── keybindings/            # Keybinding system
│   ├── memdir/                 # Memory directory management
│   ├── costHook.ts             # Cost tracking hook
│   ├── cost-tracker.ts         # Cost tracking
│   ├── dialogLaunchers.tsx     # Dialog launchers
│   ├── interactiveHelpers.tsx  # Interactive helpers
│   ├── replLauncher.tsx        # REPL launcher
│   ├── history.ts              # Session history
│   ├── native-ts/              # Native TypeScript shims
│   │   ├── color-diff/         # Color diff NAPI shim
│   │   ├── file-index/         # File index shim
│   │   └── yoga-layout/        # Yoga layout shim
│   └── upstreamproxy/          # Upstream proxy
├── vendor/                     # Vendored source code
│   ├── audio-capture-src/      # Audio capture source
│   ├── image-processor-src/    # Image processor source
│   ├── modifiers-napi-src/     # Modifiers NAPI source
│   └── url-handler-src/        # URL handler source
├── shims/                      # Local package shims for NAPI modules
│   ├── ant-claude-for-chrome-mcp/
│   ├── ant-computer-use-input/
│   ├── ant-computer-use-mcp/
│   ├── ant-computer-use-swift/
│   ├── color-diff-napi/
│   ├── modifiers-napi/
│   └── url-handler-napi/
├── xiaohongshu/                # Documentation screenshots (README assets)
├── .claude/                    # Claude project settings
├── .planning/                  # Planning documents
├── package.json                # Package manifest (Bun)
├── tsconfig.json               # TypeScript configuration
├── bun.lock                    # Bun lockfile
├── AGENTS.md                   # Agent/development guidelines
└── README.md                   # Project README (Chinese)
```

## Directory Purposes

**`src/` — Main Source:**
- Purpose: All application source code, grouped by domain
- Contains: TypeScript/TSX files, organized into ~40 subdirectories
- Key files: `dev-entry.ts`, `main.tsx`, `commands.ts`, `QueryEngine.ts`, `Tool.ts`

**`src/entrypoints/` — Application Entry Points:**
- Purpose: All entry points into the application from different invocation modes
- Contains: CLI entry (`cli.tsx`), init (`init.ts`), MCP entry (`mcp.ts`), SDK types
- Key files: `cli.tsx` (main CLI routing), `init.ts` (singleton bootstrapping)

**`src/commands/` — Slash Commands:**
- Purpose: All slash-command implementations, one subdirectory per command
- Contains: ~80 command directories, each with its own `index.ts` or `index.tsx`
- Key files: `commands.ts` (registry of all commands)

**`src/tools/` — Tool Implementations:**
- Purpose: Tool definitions callable by the AI model, one subdirectory per tool
- Contains: ~40 tool directories, each with its own implementation file
- Key files: `Tool.ts` (base abstraction), `tools.ts` (registry with feature-flag gating)

**`src/services/` — Business Logic Layer:**
- Purpose: External integrations and business services
- Contains: API client, analytics, MCP, LSP, OAuth, compaction, memory, plugins, voice, etc.
- Key files: `api/claude.ts` (Anthropic API client), `analytics/` (GrowthBook + telemetry)

**`src/components/` — UI Components:**
- Purpose: React/Ink UI components for the terminal interface
- Contains: Design system, dialog components, permission prompts, message rendering
- Key files: `design-system/` (ThemedBox, ThemedText, ThemeProvider), `permissions/` (all permission dialogs)

**`src/ink/` — Custom Ink Framework:**
- Purpose: Forked and extended Ink terminal React renderer
- Contains: DOM abstraction, event system, layout engine, hooks, components
- Key files: `dom.ts`, `focus.ts`, `frame.ts`, `components/Box.ts`, `components/Text.ts`

**`src/hooks/` — React Hooks:**
- Purpose: Shared React hooks for the Ink-based application
- Contains: ~60 hook files covering tool permissions, settings, tasks, voice, input, etc.
- Key files: `useCanUseTool.tsx`, `useSettings.ts`, `useTasksV2.ts`

**`src/state/` — Application State Management:**
- Purpose: React Context-based app state with observable store
- Contains: Store implementation, state shape, selectors, provider
- Key files: `AppState.tsx` (provider), `AppStateStore.ts` (state shape), `store.ts` (observable store)

**`src/bootstrap/` — Process Bootstrap State:**
- Purpose: Module-level global state singleton for process-wide shared state
- Contains: State definitions, getter/setter functions, state management
- Key files: `state.ts` (the comprehensive global state singleton)

**`src/cli/` — CLI Subcommand Handlers:**
- Purpose: Subcommand handlers for CLI flags and transports for structured output
- Contains: Handler implementations, transport abstractions
- Key files: `handlers/auth.ts`, `transports/SSETransport.ts`, `transports/Transport.ts`

**`src/bridge/` — Remote Bridge Mode:**
- Purpose: Remote control protocol implementation for serving Claude Code over network
- Contains: WebSocket transport, session management, JWT auth, polling
- Key files: `bridgeMain.ts`, `remoteBridgeCore.ts`, `bridgeMessaging.ts`

**`src/tasks/` — Background Task Implementations:**
- Purpose: Background process task types
- Contains: Task implementations for shell, agent, dream, workflow, monitor
- Key files: `LocalShellTask/`, `LocalAgentTask/`, `DreamTask/`

**`src/utils/` — Utility Modules:**
- Purpose: Cross-cutting utility functions and helpers
- Contains: ~100+ files covering git, shell, auth, config, file operations, permissions, settings, telemetry, etc.
- Key files: `auth.ts`, `config.ts`, `Shell.ts`, `git.ts`, `attachments.ts`, `messages.ts`

**`src/constants/` — Shared Constants:**
- Purpose: Centralized constants used across the application
- Contains: Prompts, tools, API limits, system messages, icons, emoji
- Key files: `prompts.ts` (system prompt construction), `tools.ts` (tool name constants)

**`src/types/` — TypeScript Type Definitions:**
- Purpose: Shared TypeScript types, interfaces, and generated protobuf types
- Contains: Message types, permission types, ID types, plugin types, generated protobuf event types
- Key files: `message.ts`, `permissions.ts`, `ids.ts`, `generated/events_mono/`

**`vendor/` — Vendored Source Code:**
- Purpose: Native addon source code restored from the npm package
- Contains: C++/Rust source for audio capture, image processing, modifiers, URL handling
- Generated: No
- Committed: Yes

**`shims/` — Local Package Shims:**
- Purpose: TypeScript shim packages that replace native NAPI modules with pure TS implementations
- Contains: Package.json + index.ts for each shim, referenced via `file:./shims/` in dependencies
- Generated: No
- Committed: Yes

**`src/native-ts/` — Native TypeScript Shims:**
- Purpose: TypeScript reimplementations of native functionality (color-diff, file-index, yoga-layout)
- Contains: Pure TS implementations used when native modules are unavailable
- Generated: No
- Committed: Yes

**`src/skills/bundled/` — Bundled Skills:**
- Purpose: Built-in skills for the skill system (Claude API usage examples in multiple languages)
- Contains: Claude API skill templates (Python, TypeScript, Java, Go, etc.) and verification skills
- Generated: No
- Committed: Yes

## Key File Locations

**Entry Points:**
- `src/dev-entry.ts`: Development workspace entry — version check, missing-import scan, routes to CLI entry
- `src/entrypoints/cli.tsx`: Main CLI entry — fast-path routing for all CLI flags and subcommands
- `src/entrypoints/init.ts`: Singleton one-time initialization — config, auth, telemetry, policy
- `src/main.tsx`: React/Ink application shell — mounts AppStateProvider, renders screens

**Configuration:**
- `package.json`: ESM, Bun package manager, all dependency declarations
- `tsconfig.json`: ESNext target, bundler module resolution, `src/*` path alias, JSX react-jsx
- `src/utils/config.ts`: Runtime config loading and management
- `src/utils/settings/`: Settings system (.claude/settings.json parsing, MDM integration)
- `src/utils/env.ts` / `src/utils/envDynamic.ts`: Environment configuration

**Core Logic:**
- `src/QueryEngine.ts`: Central conversation orchestration — message loop, API calls, tool dispatch
- `src/query.ts`: Lower-level query loop implementation
- `src/context.ts`: System/user context generation (git status, CLAUDE.md, memory files, etc.)
- `src/commands.ts`: Command registry with 80+ slash commands
- `src/Tool.ts`: Tool abstraction with ~40 implementations
- `src/tools.ts`: Tool instantiation with feature-flag DCE gates
- `src/coordinator/coordinatorMode.ts`: Multi-agent orchestration mode

**State Management:**
- `src/bootstrap/state.ts`: Process-wide global mutable singleton (session ID, project root, telemetry)
- `src/state/AppState.tsx`: React Context provider for UI state
- `src/state/store.ts`: Observable store implementation (`getState`, `setState`, `subscribe`)

**API Client:**
- `src/services/api/claude.ts`: Anthropic Messages API streaming client
- `src/services/api/errors.ts`: API error categorization and handling
- `src/services/api/withRetry.ts`: Retry logic with exponential backoff

**Testing:**
- No dedicated test directory exists. Tests are meant to be placed near the code they test.

## Naming Conventions

**Files:**
- PascalCase for React component files and tool/class files: `BashTool.ts`, `AppStateStore.ts`, `CompanionSprite.tsx`
- camelCase for utility files and non-class modules: `auth.ts`, `attachments.ts`, `claude.ts`, `startupProfiler.ts`
- kebab-case for command directories: `src/commands/install-slack-app/`, `src/commands/add-dir/`, `src/commands/break-cache/`
- `.tsx` extension for React components with JSX: `AppState.tsx`, `CompanionSprite.tsx`, `REPL.tsx`
- `.ts` extension for modules without JSX
- Snake-case for generated protobuf files: `events_mono/`

**Directories:**
- PascalCase for tool, task, component, and service directories: `BashTool/`, `DreamTask/`, `DesignSystem/`
- camelCase for utility and bridge directories: `utils/bash/`, `bridge/mcp/`
- kebab-case for command directories: `install-slack-app/`

## Where to Add New Code

**New Feature (non-command):**
- Primary code: `src/services/<feature-name>/` if it's a business service
- UI components: `src/components/<ComponentName>/` with PascalCase directory
- Hooks: `src/hooks/use<FeatureName>.ts` (or .tsx if JSX needed)
- Types: `src/types/` if new type definitions are needed

**New Command:**
- Implementation: `src/commands/<command-name>/index.ts` (kebab-case directory)
- Registration: Add import + entry in `src/commands.ts`

**New Tool:**
- Implementation: `src/tools/<ToolName>/<ToolName>.ts` (PascalCase directory)
- Base class: Extend `Tool` from `src/Tool.ts`
- Registration: Add to `src/tools.ts`
- Tests: Place in the same directory as the tool

**New Background Task:**
- Implementation: `src/tasks/<TaskTypeName>/` (PascalCase directory)
- Base type: Implement `Task` from `src/Task.ts`
- Registration: Add to `src/tasks.ts`

**Utility Function:**
- Shared helpers: `src/utils/<category>/` — find the relevant subdirectory or create one
- Keep modules focused — prefer many small files over broad utility modules

**UI Component:**
- Implementation: `src/components/<ComponentName>/` with PascalCase directory
- Use existing design system components from `src/components/design-system/` (ThemedBox, ThemedText)
- Add hooks in `src/hooks/` not inside the component directory

**Tests:**
- No consolidated test directory. Place tests near the feature they cover, named after the module.

## Special Directories

**`vendor/`:**
- Purpose: Native addon source code (audio-capture, image-processor, modifiers-napi, url-handler)
- Generated: No (restored from npm package)
- Committed: Yes

**`shims/`:**
- Purpose: TypeScript-only shims for native NAPI modules that cannot run without native compilation
- Generated: No (written for restoration compatibility)
- Committed: Yes

**`src/native-ts/`:**
- Purpose: Pure TypeScript reimplementations of native module functionality
- Generated: No
- Committed: Yes

**`src/types/generated/`:**
- Purpose: Auto-generated protobuf type definitions (events_mono, google protobuf)
- Generated: Yes
- Committed: Yes

**`src/skills/bundled/`:**
- Purpose: Built-in skill templates for the skill system (claude-api usage in multiple languages, verification)
- Generated: No
- Committed: Yes

**`.planning/`:**
- Purpose: Architecture and planning documents
- Generated: Yes (written by mapping tools)
- Committed: Yes

---

*Structure analysis: 2026-05-17*
