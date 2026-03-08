# Copilot Instructions — wspulse/server

## Project Overview

wspulse/server is a **minimal, production-ready WebSocket server library** for Go. It manages concurrent connections, room-based broadcasting, session resumption, and heartbeat. Module path: `github.com/wspulse/server`. Package name: `server`. Depends on `github.com/wspulse/core` for shared types (`Frame`, `Codec`).

## Architecture

- **`server.go`** — `Server` interface (public API: `Send`, `Broadcast`, `Kick`, `GetConnections`, `Close`) and `NewServer` constructor. Implements `http.Handler`.
- **`hub.go`** — Central single-threaded event loop. Manages all sessions and routes messages via channels. No `net/http` imports.
- **`session.go`** — `Connection` interface and the unexported `session` struct. Per-connection `readPump` + `writePump` goroutine pair; ping/pong heartbeat; backpressure drop.
- **`options.go`** — `ServerOption` functional options, `ConnectFunc` type, and all `WithXxx` option builders.
- **`resume.go`** — Ring buffer for buffering frames during temporary disconnects (session resumption).
- **`errors.go`** — Server-only sentinel errors: `ErrConnectionNotFound`, `ErrDuplicateConnectionID`, `ErrServerClosed`.
- **`doc/protocol.md`** — Wire protocol specification.
- **`doc/internals.md`** — Internal architecture documentation.

## Development Workflow

```bash
# Run all tests with race detector
go test -race -count=3 ./...

# Vet
go vet ./...

# Lint (requires golangci-lint)
golangci-lint run ./...

# Format
goimports -w .
```

## Conventions

- **Go style**: `gofmt`/`goimports`, snake_case filenames, GoDoc on all public symbols, `if err != nil` error handling (never `panic`), secrets from env vars only.
- **Naming — readability is the highest priority**:
  - Use full words for all identifiers. Code is AI-generated; there is no excuse for cryptic names.
  - **Allowed abbreviations** (universally recognized only): ID, URL, HTTP, API, JSON, Msg, Err, Ctx, Buf, Cfg, Fn, Opt, Req, Resp, Src, Dst, Addr, Auth, Init, Exec, Cmd, Env, Pkg, Fmt, Doc, Spec, Sync, Async, Max, Min, Len, Cap, Idx, Tmp, Ref, Val, Str, Int, Bool, Impl, Repo.
  - **Banned** — half-word truncations that harm readability: `sess`, `conn`, `svc`, `mgr`, `recv`, `svr`, `tbl`, `hdlr`, `dlg`, `desc`, `proc`, `coll`.
  - When in doubt, spell out the full word.
- **Markdown**: no emojis in documentation files.
- **Git**:
  - Follow the commit message rules in [commit-message-instructions.md](commit-message-instructions.md).
  - All commit messages in English.
  - Each commit must represent exactly one logical change.
  - Run formatter and tests locally before committing.
- **Tests**: co-located with source (`_test.go`). Cover happy path and at least one error path. Required for new public functions.

## Critical Rules

1. **Read before write** — always read the target file, `doc/protocol.md`, and `doc/internals.md` fully before editing.
2. **Minimal changes** — one concern per edit; no drive-by refactors.
3. **No hardcoded secrets** — all configuration via environment variables.
4. **Hub serialization** — all session state mutations must go through the hub's event loop. Never mutate session state from outside the hub goroutine.
5. **Accuracy** — if you have questions or need clarification, ask the user. Do not make assumptions without confirming.
6. **Language consistency** — when the user writes in Traditional Chinese, respond in Traditional Chinese; otherwise respond in English.
