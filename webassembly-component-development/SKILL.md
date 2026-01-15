---
name: webassembly-component-development
description: Comprehensive guide to WebAssembly component development covering WASI fundamentals, component composition patterns, language interoperability, runtime compatibility, and troubleshooting.
license: Apache-2.0
tags:
  - webassembly
  - component-model
  - wasi
  - composition
  - troubleshooting
---

# WebAssembly Component Development

This skill provides comprehensive expertise in developing WebAssembly components, including WASI fundamentals, composition patterns, runtime compatibility, and troubleshooting.

---

## Part 1: Core Concepts

### What is the Component Model?

The WebAssembly Component Model enables:
- Combining multiple components into a single application
- Using libraries written in one language from another language
- Building modular, reusable WebAssembly modules
- Creating component graphs with defined dependencies

### Benefits

1. **Language Interoperability:** Use the best language for each task
2. **Code Reuse:** Share components across projects
3. **Modularity:** Clear boundaries and interfaces
4. **Independent Development:** Teams can work on different components
5. **Type Safety:** WIT ensures type-safe composition

### WASI Preview1 vs Preview2

| Feature | Preview1 (wasip1) | Preview2 (wasip2) |
|---------|------------------|------------------|
| Target | `wasm32-wasi` or `wasm32-wasip1` | `wasm32-wasip2` |
| Networking | No native support | Native networking APIs |
| Component Model | Not supported | Full support |
| String Encoding | Platform-specific | UTF-8 guaranteed |
| Async | No | In progress |
| Maturity | Stable but deprecated | Current standard |

**Migration Note:** Some std library functions still use wasip1 APIs internally. Rust 1.82+ has `wasm32-wasip2` as tier-2 target.

---

## Part 2: Component Composition

### Composition Patterns

#### 1. Simple Dependency

One component depends on another component's interface.

```wit
// component-a/wit/world.wit
package example:component-a;

interface math {
    add: func(a: s32, b: s32) -> s32;
}

world component-a {
    export math;
}
```

```wit
// component-b/wit/world.wit
package example:component-b;

world component-b {
    import example:component-a/math;
    export calculate: func(x: s32, y: s32) -> s32;
}
```

#### 2. Adapter Pattern

Adapting one interface to another for compatibility.

```wit
world adapter {
    import old-api/interface;
    export new-api/interface;
}
```

#### 3. Middleware Pattern

Components that intercept and process data between other components.

```wit
world http-logger {
    import wasi:http/incoming-handler;
    export wasi:http/outgoing-handler;
    // Logs all HTTP requests/responses
}
```

#### 4. Facade Pattern

Single component providing simplified access to multiple components.

```wit
world application-facade {
    import database/client;
    import cache/client;
    import auth/service;
    export api/handler;
}
```

### Designing Component Boundaries

**Good Boundaries:**
- Clear, well-defined responsibilities
- Minimal coupling between components
- Coarse-grained interfaces (fewer, larger operations)
- Domain-aligned (matches business/technical domains)

```wit
// Good: Clear, focused interface
interface user-service {
    create-user: func(name: string, email: string) -> result<user-id, error>;
    get-user: func(id: user-id) -> result<user, error>;
    update-user: func(id: user-id, updates: user-updates) -> result<_, error>;
}
```

**Poor Boundaries (Anti-patterns):**
- Too fine-grained (many small operations)
- Tight coupling (components know too much about each other)
- Chatty interfaces (many back-and-forth calls)

```wit
// Bad: Too fine-grained, chatty interface
interface user-service {
    set-user-name: func(id: user-id, name: string);
    get-user-name: func(id: user-id) -> string;
    set-user-email: func(id: user-id, email: string);
    get-user-email: func(id: user-id) -> string;
}
```

### Composition Tools

#### wasm-tools compose

```bash
# Install wasm-tools
cargo install wasm-tools

# Compose components
wasm-tools compose component-a.wasm \
    --component component-b.wasm \
    --output composed.wasm
```

#### wasmCloud Composition

wasmCloud handles composition automatically via declarative manifests:

```yaml
# wasmcloud.toml
[[component]]
name = "my-app"
path = "./component-a.wasm"

[[component]]
name = "database"
path = "./component-b.wasm"
```

---

## Part 3: Language Interoperability

### Cross-Language Composition

**Example:** Python library used from Rust

```wit
// python-ml-lib/wit/world.wit
package ml:inference;

interface model {
    predict: func(features: list<float32>) -> list<float32>;
}

world ml-component {
    export model;
}
```

```rust
// Rust component using Python ML library
wit_bindgen::generate!({
    world: "app",
    path: "wit",
});

impl Guest for MyApp {
    fn process(data: Vec<f32>) -> Vec<f32> {
        ml::inference::model::predict(&data)
    }
}
```

### Supported Languages

- **Rust** - First-class support via `cargo component` or `wash build`
- **Python** - Via `componentize-py`
- **JavaScript/TypeScript** - Via `componentize-js`
- **Go** - Via TinyGo with component support
- **C/C++** - Via WASI SDK
- **C#** - Experimental support

### Language Compatibility Considerations

**Data Types:**
- WIT provides common types that map to each language
- Complex types (records, variants, lists) are automatically converted
- Strings are always UTF-8

**Performance:**
- Cross-language calls have overhead (serialization/deserialization)
- Keep interfaces coarse-grained to minimize calls
- Pass handles/resources instead of large data when possible

**Error Handling:**
- Use WIT `result` types for cross-language error propagation
- Each language maps `result<T, E>` to its native error handling

---

## Part 4: WASI Gotchas

### 1. No Threading Support

**Problem:** Thread pool operations are unsupported on wasm32-wasip2.

**Symptoms:**
- Panic with "operation not supported on this platform"
- Libraries like `rayon` fail at runtime
- Any code using `std::thread::spawn` fails

**Solution:**
- Avoid libraries that require threading (check dependency trees)
- Use single-threaded algorithms and data structures
- Consider async/await patterns instead of threads

### 2. String/Encoding Issues

**Problem:** Unsafe assumptions about string encoding across platforms.

**Key Facts:**
- `OsStr` from wasip2 is guaranteed to be valid UTF-8
- Avoid `as_encoded_bytes()` - uses unspecified encoding
- Conversion to `str` should almost never fail on wasip2

```rust
// Good: Safe UTF-8 handling
let path_str = path.to_str().expect("wasip2 guarantees UTF-8");

// Bad: Platform-specific encoding
let bytes = path.as_os_str().as_encoded_bytes(); // Don't do this!
```

### 3. Network Capabilities

**WASI Preview1:** No native networking support.

**WASI Preview2:**
- `wasi:http` - HTTP client and server APIs
- `wasi:sockets` - Low-level socket APIs
- Not all runtimes fully implement networking yet

### 4. Standard Library Gaps

Not all Rust standard library has migrated to WASIp2 APIs:

**Still Using WASIp1:**
- File descriptors
- Filesystem APIs (partially)
- Some environment variable handling

**Using WASIp2:**
- Most CLI interactions
- String/path handling
- Process arguments

**Implication:** Even when targeting `wasm32-wasip2`, you're using a mix of preview1 and preview2 APIs.

### 5. Dependency Management

**Problem:** Intentional duplicate dependencies cause confusion.

- `wasm32-wasip1` depends on `wasi` crate v0.11
- `wasm32-wasip2` depends on `wasi` crate v0.14

**Solution:** This is expected - let cargo handle the dual dependencies.

---

## Part 5: Runtime Compatibility

### WASI Adapters

WASI adapters bridge the gap between different WASI versions:

- **Purpose:** Implement WASI Preview 1 APIs in terms of WASI Preview 2 APIs
- **Used by:** Components built with `wasm32-wasip1` target
- **Cost:** Runtime indirection and instantiation overhead

### Unknown Import Errors

**The Most Common Error:**

```
Error: instantiation failed: unknown import
component name: wasi:cli/environment@0.2.0
```

**Root Causes:**
1. Runtime doesn't support WASI Preview 2
2. Runtime doesn't implement that specific interface
3. Version mismatch between component and runtime

**Diagnosis:**

```bash
# Check what your component imports
wasm-tools component wit your-component.wasm

# Try running and see what fails
wasmtime run component.wasm 2>&1 | grep "unknown import"
```

**Solutions:**

```bash
# Solution 1: Update runtime
cargo install wasmtime-cli --version 18.0.0

# Solution 2: Update component WIT definitions
wash wit update

# Solution 3: Remove dependency on unsupported interface
```

### Runtime-Specific Compatibility

#### wasmtime

- wasmtime 13+: WASI Preview 2 (0.2.0) support
- wasmtime 18+: Full WASI 0.2 implementation

```bash
wasmtime run --wasi preview2 component.wasm
wasmtime serve component.wasm  # For HTTP components
```

#### wasmCloud

- Full WASI 0.2 support in recent versions
- Custom wasmCloud interfaces (wasmcloud:*)
- `wasmcloud:wash` interface not published (use `wash build --skip-fetch` for plugins)

```bash
wash dev      # Development testing
wash up       # Run wasmCloud host
wash app deploy wadm.yaml
```

#### WasmEdge

- WASI Preview 1: Full support
- WASI Preview 2: Partial support (actively developing)

### Building Custom Adapters

When the built-in adapter doesn't match your runtime:

```toml
# Cargo.toml
[package.metadata.component]
adapter = "path/to/custom-adapter.wasm"
```

```bash
# Build adapter from wasmtime source
git clone https://github.com/bytecodealliance/wasmtime.git
cd wasmtime && git checkout v18.0.0
cd crates/wasi-preview1-component-adapter
cargo build --release --target wasm32-unknown-unknown
```

### Version Alignment Strategies

**Strategy 1: Match Runtime Version**

```toml
[dependencies]
wasi = "=0.14.0"  # Exact version matching runtime
```

**Strategy 2: Use Conservative Interfaces**

- ✅ `wasi:cli/environment`
- ✅ `wasi:cli/stdin/stdout/stderr`
- ✅ `wasi:filesystem` (basic operations)
- ⚠️ `wasi:http` (check runtime support)
- ⚠️ `wasi:sockets` (not universally available)

**Strategy 3: Feature Detection**

```rust
#[cfg(feature = "wasi-http")]
mod http_impl { /* HTTP-specific code */ }

#[cfg(not(feature = "wasi-http"))]
mod http_impl { /* Fallback implementation */ }
```

---

## Part 6: Testing Strategies

### 1. Lightweight Host Harness

```rust
use wasmtime::{Engine, Store, component::Component};

#[cfg(test)]
mod tests {
    #[test]
    fn test_component() -> Result<()> {
        let engine = Engine::default();
        let mut store = Store::new(&engine, ());
        let component = Component::from_file(&engine, "component.wasm")?;
        // Instantiate and test...
        Ok(())
    }
}
```

### 2. Mock Components

```rust
wit_bindgen::generate!({
    world: "mock-database",
    exports: {
        "database:client/query": MockDatabase,
    }
});

struct MockDatabase;

impl database::client::Query for MockDatabase {
    fn execute(query: String) -> Result<Vec<Row>, Error> {
        Ok(vec![/* test rows */])
    }
}
```

### 3. Integration Tests with wash

```bash
wash dev  # Start development environment
wash call my-component my-function '{"param": "value"}'
```

### 4. Feature Flags for Host-Specific Code

```rust
#[cfg(target_family = "wasm")]
mod wasm_impl {
    pub fn get_data() -> Vec<u8> {
        wasi::filesystem::read("/data/file.bin")
    }
}

#[cfg(not(target_family = "wasm"))]
mod native_impl {
    pub fn get_data() -> Vec<u8> {
        vec![1, 2, 3, 4] // Test data
    }
}
```

### 5. Environment Setup

```rust
use wasmtime_wasi::{WasiCtxBuilder, Dir};

#[test]
fn test_with_filesystem() -> Result<()> {
    let test_dir = tempfile::tempdir()?;
    std::fs::write(test_dir.path().join("test.txt"), "test data")?;

    let wasi = WasiCtxBuilder::new()
        .env("TEST_MODE", "true")?
        .preopened_dir(
            Dir::open_ambient_dir(test_dir.path(), ambient_authority())?,
            "/data",
        )?
        .build();

    // Test component...
    Ok(())
}
```

---

## Part 7: Debugging

### Debugging Workflow

1. **Check target compatibility:**
   ```bash
   rustc --print target-list | grep wasi
   ```

2. **Validate WIT interfaces:**
   ```bash
   wash wit update
   ```

3. **Inspect component imports:**
   ```bash
   wasm-tools component wit your-component.wasm
   ```

4. **Check for threading issues:**
   - Review dependencies for `rayon`, `tokio` (threaded), `std::thread`

### Common Error Patterns

| Error | Cause | Solution |
|-------|-------|----------|
| "operation not supported on this platform" | Threading issue | Remove threaded dependencies |
| "unknown import: wasi:cli/..." | Runtime doesn't support interface | Update runtime or remove dependency |
| WIT version mismatch | Version conflict | Run `wash wit update` |
| Component instantiation failed | Adapter/runtime incompatibility | Check adapter version |

### Tools

```bash
# Inspect component structure
wasm-tools print component.wasm

# Extract WIT interfaces
wasm-tools component wit component.wasm

# Validate component
wasm-tools validate component.wasm

# Check component info (wasmCloud)
wash inspect component.wasm
```

---

## Part 8: Performance Optimization

### Minimize Cross-Component Calls

```rust
// Bad: Multiple fine-grained calls
let name = user_service.get_name(id)?;
let email = user_service.get_email(id)?;

// Good: Single coarse-grained call
let user = user_service.get_user(id)?;
```

### Use Streaming for Large Data

```wit
interface data-processor {
    resource stream {
        read-chunk: func() -> option<list<u8>>;
    }
    process-stream: func(input: stream) -> result<_, error>;
}
```

### Batch Operations

```rust
// Bad: Many individual calls
for item in items {
    database.insert(item)?;
}

// Good: Single batch operation
database.insert_batch(items)?;
```

---

## Part 9: Best Practices

### Development

1. **Avoid Threading:** Design for single-threaded execution
2. **Use UTF-8:** Don't rely on platform-specific encodings
3. **Minimal Dependencies:** Fewer dependencies = fewer WASI compatibility issues
4. **Test Early:** Run components in target runtime during development

### Component Design

1. **Single Responsibility:** Each component should have one clear purpose
2. **Coarse-Grained Interfaces:** Design for fewer, larger operations
3. **Version Your Interfaces:** Use WIT package versioning
4. **Document Dependencies:** Note which WASI interfaces your component needs

### Runtime Compatibility

1. **Check Runtime Support:** Verify WASI interfaces before using them
2. **Keep Tools Updated:** Regularly update `wash`, `wasmtime`, and Rust
3. **Pin Versions in Production:** Use exact versions for reproducibility
4. **Maintain Compatibility Matrix:** Document supported runtimes

### CI/CD

```yaml
# .github/workflows/test.yml
- name: Test with wasmtime
  run: wasmtime run component.wasm

- name: Test with wasmCloud
  run: |
    wash build
    wash dev &
    # Run integration tests
```

---

## Common Architecture Patterns

### Microservices

```wit
world api-gateway {
    import user:service/handler;
    import order:service/handler;
    import payment:service/handler;
    export http:handler/incoming;
}
```

### Plugin Architecture

```wit
world app-core {
    export plugin:host/register;
    export plugin:host/execute;
}

world plugin {
    import plugin:host/core-api;
    export plugin:interface/handler;
}
```

### Data Pipeline

```wit
world data-pipeline {
    import source:reader/stream;
    import transform:processor/apply;
    import sink:writer/write;
    export pipeline:orchestrator/run;
}
```

---

## Summary

- **Component Model** enables language-agnostic, composable WebAssembly applications
- **WASI Preview2** is the current standard with full networking and UTF-8 guarantees
- **Design coarse-grained interfaces** for better performance
- **Test in target runtime** early and often
- **Use `wash build`** for wasmCloud projects - handles compatibility automatically
- **Document runtime requirements** and maintain compatibility matrices
