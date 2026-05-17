# Technology Stack

**Analysis Date:** 2026-05-17

## Languages

**Primary:**
- TypeScript (with JSX/TSX) - All application source in `src/`, `vendor/`, and `shims/`

**Secondary:**
- JavaScript (.js, .mjs, .cjs) - Some vendor and shim packages
- Audio capture native modules (Rust via `audio-capture-napi`) - Voice recording support

## Runtime

**Environment:**
- Bun >=1.3.5 (primary runtime, configured via `packageManager: "bun@1.3.5"`)
- Node >=24.0.0 (secondary runtime requirement)

**Package Manager:**
- Bun (v1.3.5)
- Lockfile: `bun.lock` (present in repository root)

## Frameworks

**Core:**
- Ink (React for CLIs) - Terminal UI rendering framework; the app's UI is composed as React components rendered via Ink's custom reconciler
- React + `react-reconciler` - Custom Ink reconciler at `src/ink/reconciler.ts` that renders React components to terminal output
- @anthropic-ai/sdk - Official Anthropic API client for LLM inference
- @anthropic-ai/claude-agent-sdk - SDK for agent-mode functionality
- @modelcontextprotocol/sdk - MCP (Model Context Protocol) SDK for tool server communication

**Testing:**
- Not detected from root-level config; test framework configuration may be partial or missing in this restored tree

**Build/Dev:**
- Bun built-in bundler (via `bun:bundle` module and `bun run`) - Bundle/build tool, no separate bundler config files
- TypeScript compiler via `tsconfig.json` (ESNext target, ESNext module, bundler resolution)

## Key Dependencies

**Critical:**
- `@anthropic-ai/sdk` (*) - Core Anthropic Claude API client for all LLM inference
- `@modelcontextprotocol/sdk` (*) - MCP client/server SDK for tool integration protocol
- `axios` (*) - HTTP client used across all API integrations (Datadog, OAuth, GrowthBook, MCP registry, etc.)
- `react` (*) + `ink` (*) + `react-reconciler` (*) - Terminal UI rendering stack
- `@commander-js/extra-typings` (*) - CLI argument parsing and command routing
- `zod` (*) - Schema validation for API responses and config data
- `@anthropic-ai/claude-agent-sdk` (*) - Agent SDK for multi-agent and assistant mode

**Infrastructure:**
- `@aws-sdk/client-bedrock-runtime` (*) - AWS Bedrock integration for Claude models
- `google-auth-library` (*) - GCP authentication for Vertex AI
- `@growthbook/growthbook` (*) - Feature flag and A/B testing framework
- `@opentelemetry/api` (*), `@opentelemetry/sdk-trace-base`, `@opentelemetry/sdk-logs`, `@opentelemetry/sdk-metrics`, `@opentelemetry/core`, `@opentelemetry/resources`, `@opentelemetry/semantic-conventions`, `@opentelemetry/api-logs` - Full OpenTelemetry observability stack
- `ws` (*) - WebSocket client for real-time session communication
- `undici` (*) - HTTP client library (undici fetch)
- `https-proxy-agent` (*) - HTTPS proxy support
- `chokidar` (*) - File system watching
- `vscode-jsonrpc` (*) + `vscode-languageserver-protocol` (*) + `vscode-languageserver-types` (*) - LSP protocol implementation
- `execa` (*) - Subprocess spawning and management
- `zod` (*) - Runtime schema validation
- `lru-cache` (*) - In-memory caching
- `xss` (*) - XSS sanitization for MCP auth UI
- `yaml` (*) - YAML parsing for config files
- `marked` (*) + `highlight.js` (*) - Markdown rendering and syntax highlighting

**Shims/Stubs:**
- `@ant/claude-for-chrome-mcp` (`file:./shims/...`) - Chrome bridge MCP server (stub)
- `@ant/computer-use-mcp` (`file:./shims/...`) - Computer use MCP server (stub)
- `@ant/computer-use-input` (`file:./shims/...`) - Computer use input handling (stub)
- `@ant/computer-use-swift` (`file:./shims/...`) - Computer use Swift bindings (stub)
- `color-diff-napi`, `modifiers-napi`, `url-handler-napi` (`file:./shims/...`) - Native addon stubs

**Additional notable:**
- `chalk`, `figures`, `cli-boxes`, `wrap-ansi`, `strip-ansi`, `ansi-styles` - Terminal text formatting
- `fuse.js` - Fuzzy string matching for command/search
- `diff` - Text comparison for diff views
- `picomatch` - Glob pattern matching
- `semver` - Semantic version comparison
- `execa` - Child process management
- `tree-kill` - Kill process trees
- `proper-lockfile` - File-based locking
- `qrcode` - QR code generation for OAuth device flow
- `asciichart` - ASCII chart rendering for stats
- `bidi-js` - Bidirectional text support
- `signal-exit` - Process exit signal handling
- `zod` - Schema validation (used pervasively)

## Configuration

**Environment:**
- Configuration loaded from settings JSON files in `$CLAUDE_CONFIG_DIR` (managed by `src/utils/config.ts`, `src/utils/settings/settings.ts`)
- Environment variables applied by `src/utils/managedEnv.ts` and `src/utils/envUtils.ts`
- Key env vars: `ANTHROPIC_API_KEY`, `ANTHROPIC_BASE_URL`, `CLAUDE_CODE_USE_BEDROCK`, `CLAUDE_CODE_USE_VERTEX`, `CLAUDE_CODE_USE_FOUNDRY`, `AWS_REGION`, `CLAUDE_CODE_PROXY`, `HTTPS_PROXY`, `NODE_EXTRA_CA_CERTS`, `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`

**Build:**
- `tsconfig.json` - TypeScript configuration (ESNext target, strict mode off, bundler module resolution, React JSX transform)
- No separate bundler/Webpack/Rollup config; Bun built-in bundler is used

**Runtime Config:**
- `src/constants/oauth.ts` - OAuth endpoints and scopes (prod/staging/local)
- `src/constants/keys.ts` - GrowthBook client keys (per environment)
- `src/constants/product.ts` - Product URLs and remote session URL builders
- `src/constants/betas.ts` - Anthropic API beta feature header constants
- `src/constants/prompts.ts` - System prompts
- `src/constants/github-app.ts` - GitHub Actions workflow templates

## Platform Requirements

**Development:**
- Bun >=1.3.5
- Node >=24.0.0
- TypeScript (project configured with strict mode off)

**Production:**
- Bun runtime deployment
- Native modules: `color-diff-napi`, `modifiers-napi`, `url-handler-napi` (Windows-specific native addons)
- Optional: audio-capture-napi (for voice input), Sharp (image-processing, via `@img/sharp-win32-x64`)
- Cross-platform: Windows (win32), macOS (darwin), Linux

---

*Stack analysis: 2026-05-17*
