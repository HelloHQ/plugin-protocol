# Changelog

All notable changes to the HelloHQ plugin protocol are documented here.
The format follows [Keep a Changelog](https://keepachangelog.com/) and the
protocol adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Planned (v2 — minor bump, no breaking changes)
- `host.ai-complete` + `chat-message` / `inference-opts` / `inference-response`
  (requires `ai:inference`, Verified tier).
- `host.storage-execute` (requires `plugin:storage`).
- `host.network-fetch` (requires `network:fetch`, Verified tier).
- `host.read-external-file` (requires `read:external_input`).

These are sketched, commented-out, at the foot of `wit/hellohq-plugin.wit` and
must not be relied upon until released.

## [1.0.0] — 2026-06-06

### Added
- Initial stable protocol.
- Tier 2 WIT package `hellohq:plugin@1.0.0`: `types`, `host`, `plugin`
  interfaces and the `hellohq-plugin` world.
- Tier 1 sidecar NDJSON schemas: `lifecycle` (ready/shutdown/ping/pong/event)
  and `envelope` (request/success-response/error-response).
