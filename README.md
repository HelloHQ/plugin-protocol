# HelloHQ Plugin Protocol

The canonical, machine-readable source of truth for the HelloHQ plugin contract.
Everything else — the host app, the SDKs, and the registry CI — consumes the
artifacts in this repository.

| Artifact | Tier | Consumed by |
|---|---|---|
| [`wit/hellohq-plugin.wit`](wit/hellohq-plugin.wit) | Tier 2 (Rust/Go/TS Wasm) | `wit-bindgen` / `jco`, the host FFI layer |
| [`sidecar/lifecycle.schema.json`](sidecar/lifecycle.schema.json) | Tier 1 (Python sidecar) | host sidecar driver, Python SDK |
| [`sidecar/envelope.schema.json`](sidecar/envelope.schema.json) | Tier 1 (Python sidecar) | host sidecar driver, Python SDK |

## Versioning

The protocol is versioned with semver — see [`VERSION`](VERSION) and
[`CHANGELOG.md`](CHANGELOG.md). Current: **1.0.0**.

- New host function or optional field → **minor**
- New required field or removed function → **major**
- New error variant or permission ID → **minor**

Plugins declare `min_host_version` in their manifest; the host refuses to load a
plugin whose required major exceeds its own.

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

Prose explanations of every type and message live in the host app docs under
`docs/plugin/02_protocol-reference.md`. This repo holds the normative artifacts;
where the two disagree, **this repo wins**.

## License

Apache 2.0 — see [LICENSE](LICENSE).
