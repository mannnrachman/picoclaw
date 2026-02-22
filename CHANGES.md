# PicoClaw — Changes & Implementations Log

> **Purpose**: This file documents all merged PRs, features, and refactors applied in this fork (`mannnrachman/picoclaw`).
> It is intentionally kept separate from `README.md` to preserve upstream documentation integrity.
> Last updated: 2026-02-22

---

## Table of Contents

- [New Modules (Added from Scratch)](#new-modules-added-from-scratch)
- [Merged Pull Requests](#merged-pull-requests)
- [Refactors & Improvements](#refactors--improvements)
- [Build & Infrastructure](#build--infrastructure)
- [Documentation Updates](#documentation-updates)
- [Branch Recovery Notes](#branch-recovery-notes)

---

## New Modules (Added from Scratch)

### `pkg/gateway/` — REST Gateway API
- `api.go` — Full REST API handler with route definitions
- `api_test.go` — Integration tests for gateway endpoints
- `middleware.go` — Auth and logging middleware
- Entry point added via `cmd/picoclaw/cmd_gateway.go`

### `pkg/security/` — Security Policy Engine
- `policy.go` — Configurable security policy engine (path traversal, SSRF, command injection, container escape prevention)
- `policy_test.go` — Full test coverage
- `approval.go` — IM-based approval flow for sensitive operations
- `approval_test.go` — Approval flow tests

### `pkg/tracing/` — OpenTelemetry Tracing
- `tracing.go` — Jaeger/OTLP-compatible tracing integration
- Configurable trace exporter endpoint

### `pkg/command/` — Slash Command System
- `types.go` — Command interface and type definitions
- `registry.go` — Command registry with dynamic registration
- `session.go` — Session-scoped command handling
- `basic.go` — Built-in commands: `/show`, `/list`, `/switch`, `/start`, `/help`

### `api/` — OpenAPI Specification
- `api/openapi.yaml` — OpenAPI 3.1 full specification
- `cmd/picoclaw/api/openapi.yaml` — Embedded spec for gateway

### `pkg/providers/key_rotator.go`
- Key rotation support for LLM providers (load balancing / failover)

### `pkg/utils/urlsafe.go`
- URL-safe string encoding/decoding utility with tests

---

## Merged Pull Requests

### PR #249 — REST API + OpenTelemetry Tracing
**Source**: `pr-249` → resolved in `merge-pr-249` (commit `9f8593f`)
- Added REST API with OpenAPI 3.1 specification
- Added OpenTelemetry tracing with Jaeger backend
- Fixed panic in atomic save when `sanitizeHistory` shrinks history
- Fixed Gemini turn-ordering 400 error from corrupted session history
- Extended `pkg/agent/context.go` with tracing support
- Extended `pkg/session/manager.go` with session listing
- Added `pkg/providers/key_rotator.go`
- Updated `go.mod` / `go.sum` with tracing dependencies

### PR #316 — Configurable Summary & Compression Thresholds
**Source**: `pr-316` → resolved in `merge-pr-316` (commit `ab3199e`)
- Added `compressionTriggerRatio`, `summaryTriggerMessages`, `summaryTriggerRatio`, `temperature` fields to `AgentInstance`
- Added `MaxTokens` and `Temperature` to `ToolLoopConfig`
- `toolloop.go`: Passes `max_tokens` and `temperature` to LLM options if not already set
- Updated `pkg/config/config.go` and `defaults.go` with new threshold config keys
- Updated `config/config.example.json` with new fields
- Fixed `subagent_tool_test.go` test alignment

### PR #335 — Slash Command System & Session Switching
**Source**: `pr-335` → resolved in `merge-pr-335`
- Implemented `/session` command for context switching
- Implemented `/show [model|channel|agents]` command
- Implemented `/help` command
- Refactored module command system handling
- Updated `pkg/agent/loop.go` to route slash commands through command registry
- Updated `pkg/session/manager.go` with `ListSessions()`, `SwitchSession()`, and user-boundary `TruncateHistory`
- Updated `pkg/state/state.go` with session state tracking

### PR #421 — Security Hardening
**Source**: `pr-421` → resolved in `merge-pr-421`
- SSRF prevention in web tools
- Path traversal protection in filesystem tools
- Command injection hardening in shell tools
- Container escape vulnerability fixes
- Added `pkg/security/approval.go` — IM-based approval for sensitive ops
- Added `pkg/security/policy.go` — configurable policy engine

### PR #282 — (upstream merge)
### PR #303 — (upstream merge)
### PR #317 — (upstream merge)
### PR #393 — (upstream merge)
### PR #428 — (upstream merge)
### PR #486 — (upstream merge)

### PR #492 — Provider Protocol Refactor
- Refactored provider creation with protocol-based configuration
- Used provider-specific protocol instead of generic OpenAI protocol
- Added `thought_signature` support for Gemini

---

## Refactors & Improvements

### `pkg/agent/`
- `context.go`: Extended with tracing hooks, improved context builder, added `context_test.go`
- `instance.go`: Minor fix for default value handling
- `loop.go`: 
  - Added slash command routing via command registry
  - Added configurable compression/summary thresholds
  - Fixed split logic to avoid breaking tool call groups mid-sequence
  - Improved `forceCompression` mid-point calculation
  - Fixed `summarizeSession` to respect tool call boundaries

### `pkg/bus/`
- `bus.go`: Extended with message filtering and routing improvements
- `types.go`: Added new message type definitions
- `bus_test.go`: Comprehensive test suite added

### `pkg/channels/`
- `discord.go`: Added typing indicator persistence during agent processing
- `telegram.go`: Removed legacy polling code, cleaned up handler
- `whatsmeow.go`: Added native WhatsApp channel support

### `pkg/config/`
- `config.go`: Added gateway config, security policy config, tracing config, threshold config
- `defaults.go`: Added defaults for new config fields
- `config.example.json`: Updated with all new fields

### `pkg/providers/`
- `http_provider.go`: Extended with retry logic and better error handling
- `types.go`: Added new protocol type definitions
- `protocoltypes/types.go`: Extended protocol type system
- `key_rotator.go`: New — provider key rotation

### `pkg/session/`
- `manager.go`: Added `ListSessions()`, `SwitchSession()`, smarter `TruncateHistory` (user-boundary aware)
- `manager_test.go`: Added comprehensive tests

### `pkg/state/`
- `state.go`: Added session state tracking, per-user state isolation

### `pkg/skills/`
- `installer.go`: Extended with better error handling and install verification

### `pkg/tools/`
- `edit.go`: Improved line-range editing, added atomic write
- `filesystem.go`: Added SSRF/path-traversal protection, improved symlink handling
- `shell.go`: Command injection hardening, timeout improvements
- `web.go`: SSRF protection, configurable fetch limits
- `toolloop.go`: Added `MaxTokens` and `Temperature` passthrough, cleaned up tool call normalization
- `cron.go`: Extended cron expression support
- All above include extended `_test.go` coverage

### `pkg/utils/`
- `media.go`: Extended MIME type detection
- `urlsafe.go`: New — URL-safe encoding utility
- `urlsafe_test.go`: Full test coverage

---

## Build & Infrastructure

- `Dockerfile`: Updated Go version and build flags
- `docker-compose.yml`: Added gateway service, health check endpoints, volume mounts
- `go.mod` / `go.sum`: Added dependencies for:
  - OpenTelemetry (tracing)
  - Jaeger exporter
  - whatsmeow (WhatsApp)
  - readline (CLI)

---

## Documentation Updates

- `README.md`: Added gateway setup, security policy config, tracing setup, model_list template
- `README.ja.md`: Translated new sections to Japanese
- `README.zh.md`: Translated new sections to Chinese




