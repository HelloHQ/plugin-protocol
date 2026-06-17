# Changelog

All notable changes to the HelloHQ plugin protocol are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/) and the
protocol adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

(nothing yet)

## [0.1.0] — 2026-06-16 — Component Model rewrite (pre-stable reset)

The Tier-2 ABI is **reset to pre-stable** and rewritten as the real WebAssembly
**Component Model + WASI 0.3** contract the async runtime (`hellohq-wasm-runtime`)
implements and verifies end-to-end on both the Cranelift and the no-JIT iOS
Pulley backends. The previous `1.0.0` entry (below) was a **pre-launch draft with
no consumers** — single `host` interface, a JSON-over-`hq_read` byte protocol, an
`ai-complete` stub — and is **superseded in place** (no users to migrate, so no
dual-version/dual-load machinery). The protocol returns to `1.0.0` when it
stabilizes.

### The contract (vs the superseded 1.0.0 draft)
- One `host` grab-bag → **focused, independently-gated interfaces**:
  `workspace`, `storage`, `events`, `log`, `inference`. Exports interface
  `plugin` → `guest` (same `init`/`run`/`metadata`).
- `inference.complete` **streams** token deltas (`-> result<stream<string>,
  api-error>`); the synchronous `ai-complete` stub is gone (streaming works on
  Wasmtime 45 / component-model-async).
- `storage` is a **real key-value capability** (was "planned").
- `events`: `emit-event(name, payload)` → `events.emit(plugin-event)`.
- Outbound HTTP is the **standard `wasi:http`** (host-gated: origin allowlist +
  H4/H5 SSRF), replacing the bespoke `network:fetch`.
- `world hellohq-plugin@0.1.0`.

## [1.0.0] — 2026-06-06 — SUPERSEDED (pre-launch draft, never consumed)

### Added
- Initial stable protocol.
- Tier 2 WIT package `hellohq:plugin@1.0.0`: `types`, `host`, `plugin`
  interfaces and the `hellohq-plugin` world.
- Tier 2 `host.ai-complete` with `chat-message` / `inference-opts` /
  `inference-response` records. NOTE: the Tier-2 path is a non-functional stub
  in 1.0.0 (returns an "unavailable" sentinel); the working `ai:inference` path
  is the Tier-1 sidecar host-call below.
- Tier 1 sidecar NDJSON schemas: `lifecycle` (ready/shutdown/ping/pong/event)
  and `envelope` (request/success-response/error-response).
- Tier 1 sidecar **host-call** schemas (`sidecar/host-calls.schema.json`):
  `ai_complete`/`ai_response`, `storage_get`/`storage_set`/`storage_delete`/
  `storage_response`, and `http_request`/`http_response`. These back the
  `ai:inference`, `plugin:storage`, and `network:fetch` permissions for Tier-1
  plugins via the stop-the-world stdin/stdout RPC.
