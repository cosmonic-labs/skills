---
name: wasi-troubleshooting
description: Expert in troubleshooting WASI (WebAssembly System Interface) and Component Model issues, including threading limitations, encoding problems, target selection, networking gaps, and common runtime errors.
license: Apache-2.0
---

# WASI & Component Model Troubleshooting

This skill provides expertise in diagnosing and fixing common issues with WASI (WebAssembly System Interface) and the WebAssembly Component Model.

## Top 5 Critical Gotchas

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
- Wait for WASI threads proposal to stabilize (in progress)

**Example Error:**
```
thread 'main' panicked at 'operation not supported on this platform'
```

### 2. String/Encoding Issues

**Problem:** Unsafe assumptions about string encoding across platforms.

**Key Facts:**
- `OsStr` from wasip2 is guaranteed to be valid UTF-8
- Avoid `as_encoded_bytes()` - uses unspecified encoding that only roundtrips on same platform/Rust version
- Conversion to `str` should almost never fail on wasip2

**Best Practice:**
```rust
// Good: Safe UTF-8 handling
let path_str = path.to_str().expect("wasip2 guarantees UTF-8");

// Bad: Platform-specific encoding
let bytes = path.as_os_str().as_encoded_bytes(); // Don't do this!
```

## WASI Preview1 vs Preview2

### Key Differences

| Feature | Preview1 (wasip1) | Preview2 (wasip2) |
|---------|------------------|------------------|
| Target | `wasm32-wasi` or `wasm32-wasip1` | `wasm32-wasip2` |
| Networking | No native support | Native networking APIs |
| Component Model | Not supported | Full support |
| String Encoding | Platform-specific | UTF-8 guaranteed |
| Async | No | In progress |
| Maturity | Stable but deprecated | Current standard |

### Migration Considerations

- Some std library functions still use wasip1 APIs internally
- Rust 1.82+ has `wasm32-wasip2` as tier-2 target

## Network Capabilities

### WASI Preview1 Limitations

**Problem:** No native networking support.

**Workarounds:**
- Use runtime-specific custom bindings (not portable)
- Rely on host-provided higher-level networking calls
- Limited to what host environment provides

### WASI Preview2 Improvements

**Available:**
- `wasi:http` - HTTP client and server APIs
- `wasi:sockets` - Low-level socket APIs

**Limitations:**
- Not all runtimes fully implement networking yet
- Use wasmtime for full support

## Standard Library Gaps

### Current State (2025)

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

## Dependency Management

### WASI Crate Versions

**Problem:** Intentional duplicate dependencies cause confusion.

- `wasm32-wasip1` depends on `wasi` crate v0.11
- `wasm32-wasip2` depends on `wasi` crate v0.14

**Solution:**
- This is expected and normal
- Don't try to force a single version
- Let cargo handle the dual dependencies

## Debugging Workflow

### When Components Fail to Run

1. **Check target compatibility:**
   ```bash
   # Verify you're using the right target
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

4. **Verify runtime support:**
   - Check runtime documentation for supported WASI interfaces
   - Ensure runtime version is recent enough
   - Look for "unknown import" error messages

5. **Check for threading:**
   - Review dependencies for thread usage
   - Look for `rayon`, `tokio` (with threaded runtime), or `std::thread`
   - Consider alternatives or feature flags to disable threading

### Common Error Patterns

**"operation not supported on this platform"**
→ Threading issue - remove threaded dependencies

**"unknown import: wasi:cli/..."**
→ Runtime doesn't support that WASI interface - update runtime or remove dependency

**WIT version mismatch**
→ Run `wash wit update`

**Component instantiation failed**
→ Check adapter compatibility and runtime support

## Testing Components

### Unit Testing with WASI

**Challenge:** Components expect certain environment setup (preopened directories, environment variables).

**Solutions:**

1. **Use lightweight host harness:**
   ```rust
   #[cfg(test)]
   mod tests {
       // Mock WASI environment
   }
   ```

2. **Integration tests with wash:**
   ```bash
   wash dev  # Provides full WASI environment
   ```

3. **Feature flags for host-specific code:**
   ```rust
   #[cfg(target_family = "wasm")]
   // WASI-specific code

   #[cfg(not(target_family = "wasm"))]
   // Native testing code
   ```

## Best Practices

1. **Avoid Threading:** Design for single-threaded execution
2. **Use UTF-8:** Don't rely on platform-specific encodings
3. **Check Runtime Support:** Verify WASI interfaces before using them
4. **Keep Tools Updated:** Regularly update `wash`, `wasmtime`, and Rust
5. **Test Early:** Run components in target runtime during development
6. **Minimal Dependencies:** Fewer dependencies = fewer WASI compatibility issues
7. **Document Requirements:** Note which WASI interfaces your component needs

## Resources

- WASI is evolving rapidly - preview2 is current
- When in doubt, use `wash dev` to test in a full WASI environment
