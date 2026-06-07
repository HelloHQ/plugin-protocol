# Changelog

All notable changes to the HelloHQ plugin protocol are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/) and the
protocol adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned (v2 — minor bump, no breaking changes)
- **Tier-2 (Wasm) `ai:inference`**: a working `host.ai-complete`. The WIT
  signature ships in 1.0.0, but the Tier-2 path is currently a stub —
  synchronous Wasmtime host callbacks cannot await inference — so only the
  Tier-1 sidecar path works today.
- `host.storage-execute` (SQL form, requires `plugin:storage`): the Tier-2 WIT
  surface for storage. Tier-1 already has key-value storage (see 1.0.0 below).
- `host.network-fetch` (byte form, requires `network:fetch`): the Tier-2 WIT
  surface for network. Tier-1 already has HTTP fetch (see 1.0.0 below).
- `host.read-external-file` (requires `read:external_input`).

The Tier-2 WIT forms above are sketched, commented-out, at the foot of
`wit/hellohq-plugin.wit` and must not be relied upon until released.

## [1.0.0] — 2026-06-06

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
