# WASI Gotchas and Runtime Compatibility

## Contents

- Gotchas: threading, string encoding, networking
- Unknown import errors
- Runtimes: wasmtime, jco, wasmCloud
- Version alignment

## 1. No Threading Support

**Problem:** `std::thread::spawn` is unsupported on `wasm32-wasip3`. Cooperative threading is in development in the component model and Rust, but not usable by default.

**Symptoms:**
- `failed to spawn thread: Os { code: 58, kind: Unsupported, message: "Not supported" }`
- "operation not supported on this platform"
- Libraries such as `rayon` fail at runtime

**Solution:**
- Avoid dependencies that need threads (check the dependency tree)
- Use async tasks on the single-threaded cooperative runtime (see [async.md](async.md))
- Get parallelism by running more component instances

## 2. String/Encoding Issues

- `OsStr` from WASI is guaranteed valid UTF-8
- Avoid `as_encoded_bytes()`; its encoding is unspecified
- Conversion to `str` should almost never fail

```rust
// Good
let path_str = path.to_str().expect("WASI guarantees UTF-8");

// Bad: platform-specific encoding
let bytes = path.as_os_str().as_encoded_bytes();
```

## 3. Networking

In WASI 0.3, networking and HTTP are async interfaces:
- `wasi:http` (`client`, `handler`): HTTP client and server
- `wasi:sockets` (`types`, `ip-name-lookup`): TCP/UDP sockets and DNS

A host or policy layer may grant only some of them. A missing grant shows up as an unknown-import error or an `access-denied` result, not a build error.

## Unknown Import Errors

The most common error:

```
Error: instantiation failed: unknown import
component name: wasi:cli/environment@0.3.1
```

**Root causes:**
1. The runtime doesn't support WASI 0.3 or the component model async features
2. The runtime doesn't implement that interface
3. Version mismatch: the component imports `@0.3.1` and the runtime provides a different 0.3.x

**Diagnosis:**

```bash
# Which interfaces and versions does the component import?
wasm-tools component wit your-component.wasm

# Run it and look for the failing import
wasmtime run component.wasm 2>&1 | grep "unknown import"
```

**Solutions:**
1. Update the runtime
2. Rebuild against the WASI version the runtime provides (`wkg wit fetch` or `wash wit update`)
3. Remove the dependency on the unsupported interface
4. As a last resort, build for `wasm32-wasip2`

## Runtimes

### wasmtime

- WASI 0.3 is enabled with `-S p3`; the component model async features are gated by `-W component-model-async`. Both flags appear in `wasmtime serve -S help` / `-W help` on wasmtime 49.0.2 (the `-S p3` and `-W component-model-async` entries; `wasmtime serve --help` also lists separate WASIp3 concurrency defaults). Newer releases may enable them by default, so check `wasmtime serve -S help`. The `serve` examples below were not run end to end.

```bash
wasmtime run component.wasm                              # wasi:cli/command components
wasmtime serve component.wasm                            # wasi:http/service components
wasmtime serve -S p3 component.wasm                      # enable WASIp3 APIs
wasmtime serve -S cli,inherit-network component.wasm     # give the guest host network access
wasmtime serve -S cli,allow-ip-name-lookup component.wasm
```

### Jco

Runs and transpiles components for Node.js and browsers. Its starter templates default to WASI 0.3.

### wasmCloud / Cosmonic

- Supports the async wasip3 worlds. Hosts may pin a specific 0.3.x (for example `wasi:http/handler@0.3.0`), so match the version your component imports.
- Provides `wasmcloud:*` interfaces such as `wasmcloud:secrets`.
- `wasmcloud:wash` is not published; build plugins with `wash build --skip-fetch`.

```bash
wash dev      # development loop with hot reload
wash build    # build a component
```

## Version Alignment

1. **Match the runtime:** pin the `wasip3` crate and the WASI 0.3.x versions your world imports to what the target runtime provides.
2. **Inspect imports:** `wasm-tools component wit component.wasm` lists every interface and version.
3. **Re-fetch on mismatch:** `wkg wit fetch`, or `wash wit update` in wasmCloud projects.
4. **Feature-gate fallbacks:**

```rust
#[cfg(all(target_os = "wasi", target_env = "p3"))]
mod platform { /* wasip3 implementation */ }

#[cfg(all(target_os = "wasi", target_env = "p2"))]
mod platform { /* wasip2 fallback */ }
```
