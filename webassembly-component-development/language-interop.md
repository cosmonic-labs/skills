# Language Interoperability

## Cross-Language Composition

A component built in one language can import or export interfaces implemented in another. The WIT interface is the only contract: a Python or Go component can satisfy an import of a Rust component with no shared code. Design those interfaces with the `wit-interface-design` skill.

## Supported Languages

| Language | Toolchain | wasip3 notes |
|----------|-----------|--------------|
| **Rust** | `wash build` or `cargo build --target wasm32-wasip3`; `wasip3` crate | Tier 2 target, stable since Rust 1.100.0. First-class. |
| **JavaScript / TypeScript** | `jco` / `componentize-js` | Jco's starter templates default to WASI 0.3. Runs in Node.js and browsers. |
| **Go** | `componentize-go` on async WASI p3 (see the `cosmonic-go` skill) | The known-working versions and build rules live in that skill. TinyGo targets wasip2. |
| **Python** | `componentize-py` | Check its docs for the WASI 0.3 status before relying on async. |
| **C / C++** | `wasi-sdk` | `wasm32-wasip3` needs wasi-sdk 34 or later. |
| **C#** | Experimental | |

## Language Compatibility Considerations

**Data types**
- WIT types (records, variants, lists, options, results, resources, `stream<T>`, `future<T>`) map onto each language's idioms
- Strings are always UTF-8
- Use `list<u8>` when the data may not be valid UTF-8

**Performance**
- Cross-component calls copy data; keep each call meaningful and batch where you can
- Pass handles (resources) or streams instead of large buffers

**Errors**
- Use WIT `result<T, E>` with a typed `E`. Each language maps it to its native error handling.
- Traps are for bugs, not for expected failures

**Async**
- `async func` maps to the language's async mechanism where one exists. Check each toolchain's support before designing an async API around it.
