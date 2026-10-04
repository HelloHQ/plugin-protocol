# HelloHQ Plugin Protocol

The canonical, machine-readable source of truth for the HelloHQ plugin contract.
Everything else — the host app, the SDKs, and the registry CI — consumes the
artifacts in this repository.

| Artifact | Tier | Consumed by |
|---|---|---|
| [`wit/hellohq-plugin.wit`](wit/hellohq-plugin.wit) | Tier 2 (Rust/Go/TS Wasm) — **Component Model + WASI 0.3** | `wit-bindgen` / `jco` / `cargo-component`, the host FFI layer |
| [`sidecar/lifecycle.schema.json`](sidecar/lifecycle.schema.json) | Tier 1 (Python sidecar) | host sidecar driver, Python SDK |
| [`sidecar/envelope.schema.json`](sidecar/envelope.schema.json) | Tier 1 (Python sidecar) | host sidecar driver, Python SDK |
| [`sidecar/host-calls.schema.json`](sidecar/host-calls.schema.json) | Tier 1 (Python sidecar) | host sidecar driver, Python SDK |

## Versioning

The protocol is versioned with semver — see [`VERSION`](VERSION) and
[`CHANGELOG.md`](CHANGELOG.md). Current: **0.1.0** — pre-stable: the Tier-2 ABI
was reset to the Component Model + WASI 0.3 contract (the prior 1.0.0 was a
pre-launch draft with no consumers, superseded in place). Returns to 1.0.0 at
stabilization.

- New host function or optional field → **minor**
- New required field or removed function → **major**
- New error variant or permission ID → **minor**

Plugins declare `min_host_version` in their manifest; the host refuses to load a
plugin whose required major exceeds its own.

## Outbound HTTP (`network:fetch`)

The same host policy applies on every transport a plugin can fetch through:
the Tier-1 sidecar `http_request` host call
([`sidecar/host-calls.schema.json`](sidecar/host-calls.schema.json)), the
Tier-2 host-call `http_request`, and Tier-2 `wasi:http`.

- **Origins.** HTTPS only, behind an SSRF guard, limited to the origins the
  manifest declares on the permission itself:
  `permissions[{"id": "network:fetch"}].scope.origins`. There is no top-level
  `network` block. Redirects are not followed.
- **Request headers.** Only `Accept`, `Accept-Language`, `Content-Type`,
  `If-None-Match` and `If-Modified-Since` are forwarded. Every other header is
  dropped, not rejected: `Authorization`, `Cookie`, `User-Agent`, API-key and
  signature headers included. Credentials are host-only.
- **Response headers.** `Set-Cookie` and `Set-Cookie2` are stripped.
- **Bodies are bytes.** Request and response bodies are capped at 8 MiB. On the
  JSON transports a response body that is valid UTF-8 is sent as text with no
  `body_encoding` (the wire is unchanged for existing plugins); anything else is
  sent as base64 with `"body_encoding": "base64"`, so binary responses arrive
  byte-exact. A reader treats an absent `body_encoding` as `"utf8"`. The JSON
  transports take the request body as text and send its UTF-8 bytes.
  `wasi:http` carries bytes natively in both directions, with no encoding field.
- **Errors.** Error text returned to a plugin never contains the request URL,
  path, query, headers or a resolved IP address; at most the origin host.
  Branch on the error code, not the message.

## Generating bindings

```bash
# Rust
wit-bindgen rust wit/hellohq-plugin.wit --world hellohq-plugin

# C / C++
wit-bindgen c wit/hellohq-plugin.wit --world hellohq-plugin

# TypeScript (via jco)
jco types wit/hellohq-plugin.wit --world hellohq-plugin
```

The official SDKs in [`HelloHQ/plugin-sdk`](https://github.com/HelloHQ/plugin-sdk)
ship pre-generated bindings and idiomatic wrappers on top of these.

## Relationship to the docs

Prose explanations of every type and message live in the workspace docs under
`docs/plugin/05-protocol-reference.md`. This repo holds the normative artifacts;
where the two disagree, **this repo wins**.

## License

Apache 2.0 — see [LICENSE](LICENSE).
