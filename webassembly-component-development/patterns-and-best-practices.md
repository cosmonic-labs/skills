# Best Practices and Common Architecture Patterns

## Contents

- Documentation references
- Development practices
- Component design principles
- Runtime compatibility practices
- CI/CD
- Common architecture patterns: microservices, plugin, data pipeline

## Documentation References

- **wasmCloud docs:** Always use https://wasmcloud.com/docs/ — do NOT use v1 docs (`/docs/v1/`), which are outdated
- **wstd docs:** https://docs.rs/wstd/latest/wstd/
- **Component Model:** https://component-model.bytecodealliance.org/

## Development

1. **Use `wash build`** when the project has `.wash/config.yaml`; otherwise use `wasm-tools component new` to produce a component
2. **Avoid Threading:** Design for single-threaded execution
3. **Use UTF-8:** Don't rely on platform-specific encodings
4. **Minimal Dependencies:** Fewer dependencies = fewer WASI compatibility issues
5. **Test Early:** Run components in target runtime during development
6. **Async options:** Use `tokio` (1.50+, with `--cfg tokio_unstable`) or `wstd` for async in Wasm components

## Component Design

1. **Single Responsibility:** Each component should have one clear purpose
2. **Coarse-Grained Interfaces:** Design for fewer, larger operations
3. **Version Your Interfaces:** Use WIT package versioning
4. **Document Dependencies:** Note which WASI interfaces your component needs

## Runtime Compatibility

1. **Check Runtime Support:** Verify WASI interfaces before using them
2. **Keep Tools Updated:** Regularly update `wash`, `wasmtime`, and Rust
3. **Pin Versions in Production:** Use exact versions for reproducibility
4. **Maintain Compatibility Matrix:** Document supported runtimes

## CI/CD

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
