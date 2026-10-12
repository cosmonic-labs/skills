---
name: webassembly-component-development
description: Build, compose, run, test and debug WebAssembly components targeting WASI 0.3 (wasip3) with native async. Use this skill whenever the user is building or troubleshooting a component or `.wasm` file, targeting `wasm32-wasip3` (or falling back to wasip2/wasip1), choosing an async approach, composing components with wac or wasm-tools, resolving WIT dependencies (wkg, wit-bindgen, `wash wit update`), hitting "unknown import" or version-mismatch errors, picking a runtime (wasmtime, jco, wasmCloud), or working across languages (Rust, JS/TS, Go, Python, C/C++). Also use it when the user mentions WASI, the component model, wasip3, or component composition. For designing or reviewing the WIT interfaces themselves (naming, capability scoping, errors, versioning), use wit-interface-design.
license: Apache-2.0
---

# WebAssembly Component Development

Build components against **WASI 0.3 (wasip3)**. It adds native async, `stream<T>` and `future<T>` to the component model, so most of the old workarounds (poll loops, `tokio_unstable`, third-party async runtimes) are no longer needed for new code. Fall back to wasip2 only when a runtime or dependency requires it.

## Key Defaults

- **Target:** `wasm32-wasip3` (Tier 2 in Rust, stable since 1.100.0). Use the `wasip3` crate for WASI bindings (`wasip2` crate for the fallback). Gate code with `#[cfg(all(target_os = "wasi", target_env = "p3"))]`.
- **Build:** use `wash build` when the project has `.wash/config.yaml`. For plugins use `wash build --skip-fetch` (the `wasmcloud:wash` interface isn't published). Otherwise `cargo build --target wasm32-wasip3 --release`.
- **Async:** use the component model's native async: `async fn` in exported handlers, `wit_stream` / `wit_future` for streams and futures, `wit_bindgen::spawn_local` for background tasks. Don't pull in `tokio` or `wstd` for new p3 code (see [async.md](async.md)).
- **Threading:** not available yet. `std::thread::spawn` returns an error on wasip3, and cooperative threading is still in development. Design single-threaded and avoid `rayon`.
- **Strings:** UTF-8. Avoid `as_encoded_bytes()`.
- **Version matching:** the runtime must implement the exact WASI 0.3.x versions your component imports. Check with `wasm-tools component wit component.wasm`.
- **wasmCloud docs:** use https://wasmcloud.com/docs/, never `/docs/v1/`.

## Reference Files

Load the relevant file when working on the matching topic:

| Topic | File |
|-------|------|
| Component Model, wasip3 vs wasip2, targets and worlds | [core-concepts.md](core-concepts.md) |
| Composition patterns, building with `wash` / `cargo`, `wac`, `.wash/config.yaml` | [composition-and-building.md](composition-and-building.md) |
| Languages and toolchains (Rust, JS/TS, Go, Python, C/C++) | [language-interop.md](language-interop.md) |
| Native async on wasip3: handlers, streams, futures, spawning; the wasip2 fallback (`tokio`, `wstd`) | [async.md](async.md) |
| Gotchas (threading, strings, networking), unknown-import errors, runtimes (wasmtime, jco, wasmCloud), version alignment | [gotchas-and-runtime.md](gotchas-and-runtime.md) |
| Testing with mock components, debugging workflow, error patterns, performance | [testing-debugging-performance.md](testing-debugging-performance.md) |
| Development practices, CI/CD, architecture patterns | [patterns-and-best-practices.md](patterns-and-best-practices.md) |
| Designing or reviewing WIT interfaces (naming, capability grants, errors, versioning) | the `wit-interface-design` skill |

## Related skills

- `wit-interface-design`: how to design the WIT contract.
- `wash`: the `wash build` / `wash dev` toolchain.
- `cosmonic-go`: Go components with `componentize-go` on async wasip3.
