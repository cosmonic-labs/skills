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

## WASI (wasip2)

The current WASI standard targets `wasm32-wasip2` (Rust 1.82+ tier-2 target):

- **Full Component Model support**
- **Native networking** via `wasi:http` and `wasi:sockets`
- **UTF-8 guaranteed** string encoding
- **Async** via `tokio` (1.50+) or `wstd`; WASIp3 (0.3) adds native async (experimental)
