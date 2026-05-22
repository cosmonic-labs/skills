---
name: webassembly-component-development
description: Comprehensive guide to WebAssembly component development covering WASI fundamentals, component composition patterns, language interoperability, runtime compatibility, and troubleshooting. Use this skill when building WebAssembly components, writing or editing WIT (WebAssembly Interface Types) definitions, composing components with wac or wasm-tools, targeting wasip1/wasip2/wasip3, working with `.wit` or `.wasm` files, or whenever the user mentions WASI, the component model, wkg, wstd, or component composition.
license: Apache-2.0
---

# WebAssembly Component Development

This skill provides comprehensive expertise in developing WebAssembly components, including WASI fundamentals, composition patterns, runtime compatibility, and troubleshooting.

## Key Defaults

- **Build:** Use `wash build` when the project has `.wash/config.yaml`. Use `wash build --skip-fetch` for plugins (since `wasmcloud:wash` interface isn't published). Otherwise `cargo build --target wasm32-wasip2 --release`.
- **Target:** `wasm32-wasip2` (Rust 1.82+ tier-2 target). WASI 0.2 (wasip2) is the current standard.
- **Async:** Pick one — `tokio` 1.50+ with `RUSTFLAGS="--cfg tokio_unstable"`, or `wstd` (lighter, Wasm-native). Do not mix.
- **Strings:** Always UTF-8 on wasip2. Avoid `as_encoded_bytes()`.
- **Threading:** Unsupported. Design single-threaded. Avoid `rayon`, threaded `tokio`, `std::thread::spawn`.
- **wasmCloud docs:** Always use https://wasmcloud.com/docs/ — never `/docs/v1/`.

## Reference Files

Load the relevant file when working on the matching topic:

| Topic | File |
|-------|------|
| Component Model fundamentals, benefits, WASI (wasip2) basics | [core-concepts.md](core-concepts.md) |
| Composition patterns (simple dependency, adapter, middleware, facade), boundary design, building with `wash build` / `cargo`, `wac`, `.wash/config.yaml` | [composition-and-building.md](composition-and-building.md) |
| Cross-language composition, supported languages (Rust, Python, JS/TS, Go, C/C++, C#), data type / performance / error handling considerations | [language-interop.md](language-interop.md) |
| WASI gotchas (no threading, UTF-8 string assumptions, network capabilities), unknown-import errors, runtime-specific compatibility (wasmtime flags, wasmCloud), version alignment strategies | [gotchas-and-runtime.md](gotchas-and-runtime.md) |
| Async with `tokio` on wasip2, `wstd` (Wasm-native async stdlib), when to choose which | [async.md](async.md) |
| Testing (host harness, mocks, wash integration, feature flags, env setup), debugging workflow, common error patterns, `wasm-tools`/`wash inspect`, performance (minimize cross-component calls, streaming, batching) | [testing-debugging-performance.md](testing-debugging-performance.md) |
| Documentation links, development/component-design/runtime-compat best practices, CI/CD examples, common architecture patterns (microservices, plugin, data pipeline) | [patterns-and-best-practices.md](patterns-and-best-practices.md) |

## Summary

- **Component Model** enables language-agnostic, composable WebAssembly applications
- **WASI 0.2 (wasip2)** is the current standard with full networking and UTF-8 guarantees
- **Design coarse-grained interfaces** for better performance
- **Test in target runtime** early and often
- **Use `wash build`** for wasmCloud projects — handles compatibility automatically
- **Document runtime requirements** and maintain compatibility matrices
