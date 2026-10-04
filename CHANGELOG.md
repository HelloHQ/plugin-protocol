# Changelog

All notable changes to the HelloHQ plugin protocol are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/) and the
protocol adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

Additive only: no message gains a required field, and an existing plugin sees
the same wire for text responses. Ships in the next minor version.

### Added
- `http_response.body_encoding` (`"utf8"` | `"base64"`, absent means
  `"utf8"`) in `sidecar/host-calls.schema.json`. The host sends a response
  body that is not valid UTF-8 as base64 with `"body_encoding": "base64"`, so
  binary responses (PDFs, images) arrive byte-exact. Text bodies are unchanged
  and carry no `body_encoding`. The Tier-2 host-call `http_request` reply uses
  the same field in its `data` object.

### Documented
- The host's request-header allowlist for `network:fetch`: only `Accept`,
  `Accept-Language`, `Content-Type`, `If-None-Match` and `If-Modified-Since`
  are forwarded; every other header (credentials, cookies, `User-Agent`,
  vendor key/signature headers) is dropped, not rejected. `Set-Cookie` is
  stripped from responses.
- Error text returned to a plugin never contains the request URL, path,
  query, headers or a resolved IP address.
- The 8 MiB request and response caps, and that a body on GET/HEAD is refused.
- Outbound HTTP policy summary in the README, covering the sidecar host call,
  the Tier-2 host call and `wasi:http`.

### Fixed
- `http_request` pointed at a manifest `network.allowed_origins` field that
  does not exist. The allowlist is the `network:fetch` permission's own
  `scope.origins`.
- The `ready` message's `protocol_version` example said `1.0.0`; the protocol
  is `0.1.0`.
- The README pointed at a protocol reference path that no longer exists.

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
