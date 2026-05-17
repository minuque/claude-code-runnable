# Testing Patterns

**Analysis Date:** 2026-05-17

## Test Framework

**Runner:**
- No test runner explicitly configured in `package.json` — no `jest.config.*`, `vitest.config.*`, or test script found at project root
- The project uses **Bun** (`bun@1.3.5`) as its runtime and package manager
- Bun ships with a built-in test runner (`bun test`), which is the likely test framework
- `bun.lock` present (lockfile for Bun)

**Config:**
- No test configuration file detected at project root
- `"types": ["bun"]` in `tsconfig.json` indicates Bun type definitions are available in all source files

**Run Commands (inferred from project setup):**
```bash
bun test                    # Run all tests (if any exist — none found in current state)
bun run dev                 # Smoke-test the CLI interactively
bun run version             # Verify CLI boot and version output
```

**Current state:** The restored codebase does NOT contain any test files (`*.test.ts`, `*.spec.ts`, `__tests__/`) in `src/` or at project root. The `package.json` exposes no `test`, `lint`, or `coverage` scripts.

## Test File Organization

**Location:** Not established — `AGENTS.md` advises placing tests "close to the feature they cover" rather than in a centralized `test/` directory.

**Naming convention (guidance from AGENTS.md):** Name test files after the module or behavior under test.

## Infrastructure for Testing (`NODE_ENV === 'test'` Guards)

The codebase uses `process.env.NODE_ENV === 'test'` runtime checks extensively (>30 locations across `src/`) to modify behavior when running under test. This is the primary testing infrastructure mechanism.

**Key test-mode behaviors:**
```typescript
// src/context.ts:37 — Short-circuit expensive operations
if (process.env.NODE_ENV === 'test') {
  return null  // Avoid cycles in tests
}

// src/setup.ts:444 — Early return to skip production setup
if (process.env.NODE_ENV === 'test') {
  return
}

// src/tools.ts:244 — Conditionally register test-only tools
...(process.env.NODE_ENV === 'test' ? [TestingPermissionTool] : []),

// src/utils/git/gitFilesystem.ts:331 — Fast intervals for tests
const WATCH_INTERVAL_MS = process.env.NODE_ENV === 'test' ? 10 : 1000

// src/services/analytics/config.ts:21 — Disable telemetry in tests
process.env.NODE_ENV === 'test' ||

// src/ink/reconciler.ts:292 — Ink reconciler behavior for test environments
if (process.env.NODE_ENV === 'test') {
```

## Test-Only Exports

**`src/bootstrap/state.ts`:**
- `resetStateForTests()` — Resets all global state to initial values (guarded: throws if `NODE_ENV !== 'test'`)
- `resetModelStringsForTestingOnly()` — Resets model strings cache for test re-initialization
- `resetTotalDurationStateAndCost_FOR_TESTS_ONLY()` — Resets cost/duration accumulators
- Named with suffix `_FOR_TESTS_ONLY` to signal these are not for production use

```typescript
// Only used in tests
export function resetStateForTests(): void {
  if (process.env.NODE_ENV !== 'test') {
    throw new Error('resetStateForTests can only be called in tests')
  }
  Object.entries(getInitialState()).forEach(([key, value]) => {
    STATE[key as keyof State] = value as never
  })
  // ...
}
```

**`src/cost-tracker.ts`:**
- `resetStateForTests` — Exported from cost-tracker (re-exports from state)
- `setCostStateForRestore` — Accepts snapshot data, used by both session restore and test setup

## VCR (Video Cassette Recorder) Fixture System

**Location:** `src/services/vcr.ts`

The codebase includes a VCR-like fixture system that records and replays API responses for deterministic testing. When `NODE_ENV === 'test'` or `FORCE_VCR` is set, operations use cached fixture files instead of making real API calls.

```typescript
function shouldUseVCR(): boolean {
  if (process.env.NODE_ENV === 'test') {
    return true
  }
  if (process.env.USER_TYPE === 'ant' && isEnvTruthy(process.env.FORCE_VCR)) {
    return true
  }
  return false
}
```

**Fixture storage:**
- Input is hashed (SHA1, first 12 hex chars) to produce a deterministic filename
- Fixtures stored at `fixtures/` under `CLAUDE_CODE_TEST_FIXTURES_ROOT` or `getCwd()`
- Files stored as JSON: `fixtures/{fixtureName}-{hash}.json`

## TestingPermissionTool

**Location:** `src/tools/testing/TestingPermissionTool.tsx`

A test-only tool that always requires permission, used for end-to-end testing of the permission dialog flow. Only registered when `process.env.NODE_ENV === 'test'` (gated in `src/tools.ts:244`).

```typescript
isEnabled() {
  return "production" === 'test'
}
```

## Mocking Strategy

No mocking framework found. Comments in the codebase reference common test spy patterns:
- `spyOn()` is referenced in comments (`buddy/companion.ts`, `bridge/bridgeEnabled.ts`, `cli/exit.ts`) suggesting **Bun's built-in `mock`/`spyOn`** is the intended mechanism
- Comments discuss spying on `console.error` and `process.stdout.write`
- Module-level indirection for testability:
  ```typescript
  // Capture the module (not .isSkillSearchEnabled directly) so spyOn() in tests
  // patches what we actually call — a captured function ref would point past the spy.
  ```
  (`src/constants/prompts.ts:93-94`) — Functions are imported as module references rather than captured values so `spyOn()` can intercept them.

## Coverage

**Requirements:** None enforced — no coverage configuration detected.

**View Coverage:** No coverage tooling configured.

## Test Types

**Unit Tests:**
- Not present in current codebase
- Guidance in `AGENTS.md`: "place them close to the feature they cover"

**Integration Tests:**
- VCR fixture system (`src/services/vcr.ts`) suggests fixture-based integration testing of API calls

**E2E Tests:**
- `TestingPermissionTool` references end-to-end testing: "Used for end-to-end testing."
- No E2E test framework detected (Playwright, Cypress, etc. not in dependencies)

## CI Detection

```typescript
// src/utils/auth.ts:265
if (isEnvTruthy(process.env.CI) || process.env.NODE_ENV === 'test') {
```

The codebase recognizes `CI` environment variable as a signal to modify behavior, implying CI pipeline awareness but no CI configuration detected in the repository.

---

*Testing analysis: 2026-05-17*
