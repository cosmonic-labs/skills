# WASI Gotchas and Runtime Compatibility

## Contents

- WASI Gotchas: threading, string encoding, network capabilities
- Unknown Import Errors
- Runtime-Specific Compatibility (wasmtime, wasmCloud)
- Version Alignment Strategies

## 1. No Threading Support

**Problem:** Thread pool operations are unsupported on wasm32-wasip2.

**Symptoms:**
- Panic with "operation not supported on this platform"
- Libraries like `rayon` fail at runtime
- Any code using `std::thread::spawn` fails

**Solution:**
- Avoid libraries that require threading (check dependency trees)
- Use single-threaded algorithms and data structures
- Consider async/await patterns instead of threads

## 2. String/Encoding Issues

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

## 3. Network Capabilities

Available WASI networking interfaces:
- `wasi:http` - HTTP client and server APIs
- `wasi:sockets` - Low-level socket APIs

## Unknown Import Errors

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

# Solution 2: Re-fetch WIT dependencies
wkg wit fetch

# Solution 3: Remove dependency on unsupported interface
```

## Runtime-Specific Compatibility

### wasmtime

- Full WASI 0.2 support (stable)
- WASI 0.3 (WASIp3) support is experimental — adds native async

```bash
wasmtime run component.wasm
wasmtime serve component.wasm                          # HTTP components
wasmtime serve -S cli component.wasm                   # Enable wasi:cli (env vars, filesystem, sockets, clocks)
wasmtime serve -S cli,inherit-network component.wasm   # Full host network access within the guest
wasmtime serve -S cli,allow-ip-name-lookup component.wasm # Enable wasi:sockets/ip-name-lookup
wasmtime serve -S p3 component.wasm                    # Enable WASIp3 (experimental)
```

### wasmCloud

- Full WASI 0.2 support
- Custom wasmCloud interfaces (wasmcloud:*)
- `wasmcloud:wash` interface not published (use `wash build --skip-fetch` for plugins)

```bash
wash dev      # Development testing with hot-reload
```

## Version Alignment Strategies

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
