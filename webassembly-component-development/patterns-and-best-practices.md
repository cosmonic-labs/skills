# Best Practices and Common Architecture Patterns

## Contents

- Documentation references
- Development practices
- Component design principles
- Runtime compatibility practices
- CI/CD
- Common architecture patterns

## Documentation References

- **wasmCloud docs:** Always use https://wasmcloud.com/docs/ — do NOT use v1 docs (`/docs/v1/`), which are outdated
- **wstd docs:** https://docs.rs/wstd/latest/wstd/
- **Component Model:** https://component-model.bytecodealliance.org/

## Development

1. **Target `wasm32-wasip3`.** Use `wash build` when the project has `.wash/config.yaml`; otherwise `cargo build --target wasm32-wasip3 --release`
2. **Avoid Threading:** Design for single-threaded, cooperative execution
3. **Use UTF-8:** Don't rely on platform-specific encodings
4. **Minimal Dependencies:** Fewer dependencies = fewer WASI compatibility issues
5. **Test Early:** Run components in target runtime during development
6. **Async:** On wasip3 use native component-model async (see [async.md](async.md)). Use `tokio` or `wstd` only on the wasip2 fallback

## Component Design

1. **Single Responsibility:** Each component should have one clear purpose
2. **Coarse-Grained Operations:** Design for fewer, larger calls across component boundaries. Keep *interfaces* fine-grained, one capability each. For WIT design rules (naming, capability grants, errors, versioning), see the `wit-interface-design` skill
3. **Version Your Interfaces:** Use WIT package versioning
4. **Document Dependencies:** Note which WASI interfaces your component needs

## Runtime Compatibility

1. **Check Runtime Support:** Verify the runtime supports WASI 0.3 and the exact interface versions you import
2. **Keep Tools Updated:** Regularly update `wash`, `wasmtime`, and Rust
3. **Pin Versions in Production:** Use exact versions for reproducibility
4. **Maintain Compatibility Matrix:** Document supported runtimes

## CI/CD

```yaml
# .github/workflows/test.yml
- name: Test with wasmtime
  run: wasmtime run component.wasm   # or: wasmtime serve component.wasm

- name: Test with wasmCloud
  run: |
    wash build
    wash dev &
    # Run integration tests
```

## Common Architecture Patterns

Design each interface with the `wit-interface-design` skill. These describe the shape of the world, not the WIT.

- **Gateway / microservices:** an HTTP `service` component imports narrow capabilities for each downstream service and exports `wasi:http/handler`.
- **Plugin architecture:** the core component defines a small plugin interface and imports it; each plugin exports it. Because the interface is virtualizable, plugins can be swapped, wrapped or mocked.
- **Data pipeline:** stages import a source and export a sink with matching `stream<T>` types, so stages compose in sequence without adapters.
- **Middleware chain:** components that import and export the same interface (such as `wasi:http/handler`), stacked for logging, auth and caching.
