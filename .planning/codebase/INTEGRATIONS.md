# External Integrations

**Analysis Date:** 2026-05-17

## APIs & External Services

**LLM Inference (Primary):**
- **Anthropic API** (`api.anthropic.com`) - Core LLM inference for Claude Code
  - SDK: `@anthropic-ai/sdk` at `src/services/api/claude.ts`
  - Client factory: `src/services/api/client.ts`
  - Auth: `ANTHROPIC_API_KEY` env var or OAuth token via `claude.ai` login
  - Beta headers: Multiple feature-flag beta headers defined in `src/constants/betas.ts` (interleaved thinking, context-1m, structured outputs, web search, tool search, effort, prompt caching, etc.)
  - Custom base URL supported via `ANTHROPIC_BASE_URL` env var

**LLM Inference (Third-Party Providers):**
- **AWS Bedrock** - Alternative provider via `@aws-sdk/client-bedrock-runtime`
  - Detect: `CLAUDE_CODE_USE_BEDROCK=1` env var
  - Client: `src/utils/model/bedrock.ts`
  - Region: `AWS_REGION` or `AWS_DEFAULT_REGION` (default: `us-east-1`)
  - Auth: AWS SDK credential chain
  - Inference profiles: Supports `ListInferenceProfiles` via `@aws-sdk/client-bedrock`

- **Google Vertex AI** - Alternative provider via `google-auth-library`
  - Detect: `CLAUDE_CODE_USE_VERTEX=1` env var
  - Auth: Standard GCP credential chain via `google-auth-library`
  - Project: `ANTHROPIC_VERTEX_PROJECT_ID` required
  - Region: Model-specific env vars (`VERTEX_REGION_CLAUDE_3_5_HAIKU`, `VERTEX_REGION_CLAUDE_3_7_SONNET`, etc.) or `CLOUD_ML_REGION`

- **Microsoft Foundry (Azure)** - Alternative provider
  - Detect: `CLAUDE_CODE_USE_FOUNDRY=1` env var
  - Resource: `ANTHROPIC_FOUNDRY_RESOURCE` env var
  - Auth: `ANTHROPIC_FOUNDRY_API_KEY` or Azure AD `DefaultAzureCredential`

**OAuth / Authentication:**
- **Claude.ai OAuth** - Primary authentication for Claude Code subscribers
  - Config: `src/constants/oauth.ts`
  - Client: `src/services/oauth/client.ts`
  - Scopes: `user:inference`, `user:profile`, `user:sessions:claude_code`, `user:mcp_servers`, `user:file_upload`, `org:create_api_key`
  - OAuth flows: Authorization code flow with PKCE, device flow with QR code, manual token entry
  - Endpoints: `CONSOLE_AUTHORIZE_URL`, `CLAUDE_AI_AUTHORIZE_URL`, `TOKEN_URL`, `API_KEY_URL`
  - Supports prod/staging/local OAuth configs via `USE_STAGING_OAUTH` / `USE_LOCAL_OAUTH` env vars
  - Custom OAuth URL via `CLAUDE_CODE_CUSTOM_OAUTH_URL` env var

- **API Key Authentication** - Fallback auth method
  - Stored in: macOS Keychain, filesystem, or env var `ANTHROPIC_API_KEY`
  - Managed by: `src/utils/auth.ts`, `src/utils/authPortable.ts`, `src/utils/secureStorage/`

**Analytics & Telemetry:**
- **Datadog** - Event logging sink
  - Endpoint: `https://http-intake.logs.us5.datadoghq.com/api/v2/logs`
  - Client token: embedded in `src/services/analytics/datadog.ts` (pubbbf48e6d78dae54bceaa4acf463299bf)
  - Gated by GrowthBook feature `tengu_log_datadog_events`
  - Flush interval: 15s, max batch: 100 events

- **First-Party Event Logging** (Anthropic internal) - Analytics via Anthropic API
  - Endpoint: `https://api.anthropic.com/api/event_logging/batch`
  - Exporter: `src/services/analytics/firstPartyEventLoggingExporter.ts`
  - Uses protobuf-generated types: `ClaudeCodeInternalEvent`, `GrowthbookExperimentEvent`
  - Failed events persisted to `$CLAUDE_CONFIG_DIR/telemetry/` for retry

- **GrowthBook** - Feature flags and A/B testing
  - SDK: `@growthbook/growthbook` at `src/services/analytics/growthbook.ts`
  - Client keys (per environment): `sdk-zAZezfDKGoZuXXKe` (external), `sdk-xRVcrliHIlrg4og4` (ant staging), `sdk-yZQvlplybuXjYh6L` (ant dev)
  - Used for: feature gates, event sampling config, dynamic config values

- **OpenTelemetry** - Full observability stack
  - Signals: Traces (`sdk-trace-base`), Logs (`sdk-logs`), Metrics (`sdk-metrics`)
  - Instrumentation: `src/utils/telemetry/instrumentation.ts`
  - BigQuery Metrics: Custom `BigQueryMetricsExporter` at `src/utils/telemetry/bigqueryExporter.ts` sending to `api.anthropic.com/api/claude_code/metrics`
  - Session tracing: `src/utils/telemetry/sessionTracing.ts` and `src/utils/telemetry/betaSessionTracing.ts`

**Feature Flags:**
- **GrowthBook** - Main feature flag system (see Analytics section)

## Data Storage

**Local File System:**
- Configuration: JSON files in `$CLAUDE_CONFIG_DIR` (platform-specific via `env-paths` package)
- Session state: JSON files on local filesystem
- File locks: `proper-lockfile` for concurrency control
- Plugin storage: `.claude/plugins/` directory in user's config home
- Telemetry buffer: `$CLAUDE_CONFIG_DIR/telemetry/` for failed 1P events

**No external database** - All state is ephemeral (in-memory) or persisted to local filesystem

**File Storage:**
- **Anthropic Files API** - File upload/download for attachments
  - Docs: https://docs.anthropic.com/en/api/files-content
  - Client: `src/services/api/filesApi.ts`
  - Beta header: `files-api-2025-04-14,oauth-2025-04-20`
  - Base URL: `ANTHROPIC_BASE_URL` or `https://api.anthropic.com`

**Caching:**
- `lru-cache` - In-memory LRU cache used throughout the codebase
- File system cache for GrowthBook feature flag values (survives restarts)
- No external caching service (Redis, etc.)

## Authentication & Identity

**Auth Provider:**
- **Anthropic OAuth** - Primary identity provider via `claude.ai`
  - Implementation: PKCE OAuth 2.0 authorization code flow with local HTTP server for callback
  - Token refresh: Automatic with retry via `checkAndRefreshOAuthTokenIfNeeded`
  - Session ingress auth tokens for remote sessions (separate short-lived tokens)
  - Trusted device tokens for elevated security (CCR v2)

- **API Key** - Fallback for direct API access
  - macOS Keychain integration via `src/utils/secureStorage/`
  - Cross-platform: `src/utils/authPortable.ts`

## Monitoring & Observability

**Error Tracking:**
- No external error tracking service (Sentry, etc.)
- Error logging via `logError` utility and OpenTelemetry logs
- Console-based diagnostic output

**Logs:**
- Structured: OpenTelemetry LoggerProvider with `BatchLogRecordProcessor` + custom BigQuery exporter
- Debug: `logForDebugging` utility (console/file with level support)
- Diagnostics: `logForDiagnosticsNoPII` for privacy-safe diagnostic logs
- MCP-specific logging via `logMCPDebug`
- Error logs: `logError` with stack trace capture (`stack-utils`)

## CI/CD & Deployment

**Hosting:**
- Self-hosted / user-installed CLI tool (no SaaS hosting by the tool itself)
- Distribution via npm registry (or custom channels)
- Update server: `https://storage.googleapis.com/claude-code-dist-86c565f3-f756-42ad-8dfa-d59b1c096819/claude-code-releases` (GCS bucket)

**CI Pipeline:**
- No CI pipeline files found in repository root
- GitHub Actions workflow used for integration (generated by the tool, not CI for the tool itself)
- Auto-update mechanism in `src/utils/autoUpdater.ts` fetches latest version from GCS bucket

## Environment Configuration

**Required env vars:**
- `ANTHROPIC_API_KEY` - API key for direct Anthropic API access (if not using OAuth)
- `ANTHROPIC_BASE_URL` - Custom API base URL (optional)

**Provider-specific env vars:**
- `CLAUDE_CODE_USE_BEDROCK=1` + `AWS_REGION` - AWS Bedrock
- `CLAUDE_CODE_USE_VERTEX=1` + `ANTHROPIC_VERTEX_PROJECT_ID` - Google Vertex AI
- `CLAUDE_CODE_USE_FOUNDRY=1` + `ANTHROPIC_FOUNDRY_RESOURCE` - Microsoft Foundry

**Proxy/mTLS env vars:**
- `HTTPS_PROXY` / `HTTP_PROXY` / `NO_PROXY` - Proxy configuration for all HTTP traffic
- `CLAUDE_CODE_PROXY` - Additional proxy config
- `CLAUDE_CODE_CLIENT_CERT` / `CLAUDE_CODE_CLIENT_KEY` - mTLS client certificate paths
- `NODE_EXTRA_CA_CERTS` - Additional CA certificates

**Other notable env vars:**
- `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC` - Disables non-essential network calls
- `CLAUDE_CODE_REMOTE=true` - Enables remote/container mode
- `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` - Disables background task processing
- `CLAUDE_CODE_ABLATION_BASELINE` - Ablation experiment baseline mode
- `ENABLE_CLAUDEAI_MCP_SERVERS` - Enables MCP servers from claude.ai org configs
- `CLAUDE_CODE_SKIP_BEDROCK_AUTH` - Skip AWS Bedrock auth for development

**Secrets location:**
- macOS Keychain (via `src/utils/secureStorage/macOsKeychainHelpers.ts`)
- Encrypted filesystem storage (via `src/utils/secureStorage/`)
- In-memory after retrieval; cleared on session end
- `.env` file: Not present in repository (environment configuration noted but not committed)

## Webhooks & Callbacks

**Incoming:**
- **Hook System** - Configurable webhook/HTTP hooks for events
  - Implementation: `src/utils/hooks/execHttpHook.ts`
  - SSRF-protected via `src/utils/hooks/ssrfGuard.ts`
  - Supports: HTTP POST hooks, prompt hooks, agent hooks
  - Configurable via settings with allowlist restrictions

**Outgoing:**
- **GitHub Webhook Messages** - UI component at `src/components/messages/UserGitHubWebhookMessage.tsx` for displaying webhook-triggered messages
- **Webhook Sanitizer** - `src/bridge/webhookSanitizer.ts` (stub implementation)
- **Bridge Callbacks** - Webhook-like callbacks from remote sessions via the environments API (`src/bridge/`)

## Protocol Integrations

**MCP (Model Context Protocol):**
- SDK: `@modelcontextprotocol/sdk`
- Client: `src/services/mcp/client.ts` with transport layer
- OAuth: Full MCP OAuth flow at `src/services/mcp/auth.ts`
- Cross-App Access (XAA): `src/services/mcp/xaa.ts` for enterprise managed authorization
- Registry: `src/services/mcp/officialRegistry.ts` fetches from `https://api.anthropic.com/mcp-registry/v0/servers`
- Claude.ai org MCP configs: `src/services/mcp/claudeai.ts`
- In-process transport: `src/services/mcp/InProcessTransport.ts`
- SDK control transport: `src/services/mcp/SdkControlTransport.ts`

**LSP (Language Server Protocol):**
- Implementation: `src/services/lsp/LSPClient.ts`
- Protocol libraries: `vscode-jsonrpc`, `vscode-languageserver-protocol`
- Includes: server manager, diagnostic registry, passive feedback, config

**SSH:**
- Implementation: `src/ssh/SSHSessionManager.ts`, `src/ssh/createSSHSession.ts`
- Used for remote development session connections

**WebSocket:**
- Client: `ws` npm package at `src/cli/transports/WebSocketTransport.ts`
- Uses for: remote session communication, real-time message streaming
- Supports: reconnection with exponential backoff, keepalive/ping, liveness detection

**SSE (Server-Sent Events):**
- Implementation: `src/cli/transports/SSETransport.ts`
- Uses for: streaming responses from session ingress
- AXSOS-based HTTP streaming with retry logic

**mTLS:**
- Configuration: `src/utils/mtls.ts`
- Env vars: `CLAUDE_CODE_CLIENT_CERT`, `CLAUDE_CODE_CLIENT_KEY`
- HTTP agent + WebSocket + fetch TLS options all configurable

## Real-Time Communication

**Bridge (Remote Control):**
- Full protocol implementation in `src/bridge/`
- Endpoints-based polling for work items
- WebSocket-based REPL bridge for interactive remote sessions
- JWT-based authentication with trusted device tokens
- Session ingress for claude.ai integration

**Teleport:**
- Implementation: `src/utils/teleport/`
- HTTP API for session teleportation between environments
- OAuth headers for authentication

---

*Integration audit: 2026-05-17*
