# Testing, Debugging, and Performance

## Contents

- Testing: mock components, host harness, integration, native fallbacks
- Debugging workflow, common error patterns, tools
- Performance

## Testing Strategies

### 1. Test doubles are components

Every interface is virtualizable, so a mock is just another component that exports the same interface. To test a component, compose it with mock implementations of its imports (for example with `wac plug`) and drive its exports. No special test hooks are needed. If an import can't be mocked without bypassing it, the interface design is wrong (see `wit-interface-design`).

### 2. Run it in the target runtime

```bash
wasmtime run component.wasm            # wasi:cli/command components
wasmtime serve component.wasm          # wasi:http/service components
wash dev                               # wasmCloud: build, run and hot-reload
```

Test in the runtime you deploy to as early as possible. Behavior differs by runtime and by WASI version.

### 3. Host harness

Embed Wasmtime in a Rust test to instantiate a component, call its exports and supply controlled imports (clocks, files, environment). Check the current `wasmtime` and `wasmtime-wasi` docs for the API; it changes between releases, and wasip3 async calls use the concurrent APIs (`call_concurrent`, `run_concurrent`).

### 4. Native fallbacks for unit tests

Keep pure logic free of WASI so it runs under plain `cargo test`. Gate the platform code:

```rust
#[cfg(all(target_os = "wasi", target_env = "p3"))]
mod platform { /* wasip3 implementation */ }

#[cfg(not(target_os = "wasi"))]
mod platform { /* native test implementation */ }
```

## Debugging

### Workflow

1. **Check the target is installed:**
   ```bash
   rustup target list --installed | grep wasi
   rustc --print target-list | grep wasi
   ```
2. **Fetch WIT dependencies:**
   ```bash
   wash build        # fetches automatically in wash projects
   wash wit update   # fix mismatched WIT definitions
   wkg wit fetch     # manual management
   ```
3. **Inspect imports and exports:**
   ```bash
   wasm-tools component wit your-component.wasm
   ```
4. **Check for threading:** review dependencies for `rayon`, `std::thread`, threaded runtimes.
5. **Check WASI versions** against the runtime (see [gotchas-and-runtime.md](gotchas-and-runtime.md)).

### Common Error Patterns

| Error | Cause | Solution |
|-------|-------|----------|
| "operation not supported on this platform" / "failed to spawn thread" | Threading | Remove threaded dependencies |
| "unknown import: wasi:..." | Runtime lacks the interface or version | Update the runtime, rebuild against its WASI version, or drop the dependency |
| WIT version mismatch | Conflicting dependency versions | `wkg wit fetch` or `wash wit update`, then rebuild |
| Component fails to instantiate | Runtime lacks component-model async support | Use a runtime that supports WASI 0.3, or build for wasip2 |
| Async call never completes | Blocking work in a single-threaded instance | Yield, split the work, or move it to another component |

### Tools

```bash
wasm-tools print component.wasm          # text form
wasm-tools component wit component.wasm  # extract WIT
wasm-tools validate component.wasm       # validate
wash inspect component.wasm              # wasmCloud component info
```

## Performance

- **Minimize cross-component calls.** Each call copies data across the boundary. Prefer one operation that returns a record over several getters.
- **Stream large data.** Use `stream<T>` in the interface, not a sequence of chunk calls.
- **Batch operations.** One call that takes a `list<item>` beats a loop of individual calls.
- **Pass handles, not payloads,** when the data stays on the implementer's side.
- **Keep CPU-heavy work short.** An instance is single-threaded; long loops block its other tasks.
