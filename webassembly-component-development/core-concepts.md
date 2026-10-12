# Core Concepts: Component Model and WASI

## What is the Component Model?

The WebAssembly Component Model enables:
- Combining multiple components into a single application
- Using libraries written in one language from another language
- Building modular, reusable WebAssembly modules
- Creating component graphs with defined dependencies

## Benefits

1. **Language Interoperability:** Use the best language for each task
2. **Code Reuse:** Share components across projects
3. **Modularity:** Clear boundaries and interfaces
4. **Independent Development:** Teams can work on different components
5. **Type Safety:** WIT ensures type-safe composition
6. **Virtualization:** Any import can be satisfied by a host, a plugin, or another component

## WASI 0.3 (wasip3)

WASI 0.3 is the current target. It adds native async to the component model:

- **`async func`** in WIT, mapped to `async fn` in languages that have it
- **`stream<T>` and `future<T>`** as first-class types, replacing the 0.2 `wasi:io` pollable and stream resources
- **One interface family** for HTTP, sockets, filesystem, clocks, random and CLI, redesigned around async
- **Worlds:** `wasi:cli/command` (exports `run`) and `wasi:http/service` (exports `handler`) are the standard entry points
- **UTF-8** string encoding throughout
- **No threads yet.** Cooperative threading is still in development

## Targets

| Target | Status | Use when |
|--------|--------|----------|
| `wasm32-wasip3` | Rust Tier 2, stable since 1.100.0 | Default for new components |
| `wasm32-wasip2` | Stable | A runtime or dependency doesn't support 0.3 yet; `tokio` or `wstd` is required |
| `wasm32-wasip1` | Maintenance only | Core modules, legacy toolchains |

Runtimes that execute wasip3 components must support WASI 0.3 and the component model async features. Wasmtime and Jco do. See [gotchas-and-runtime.md](gotchas-and-runtime.md).

## Designing the interfaces

Use the `wit-interface-design` skill for how to write WIT: capability-scoped imports, async-by-default, virtualizable APIs, typed errors, versioning.
