# Component Composition and Building

## Contents

- Composition patterns: simple dependency, adapter, middleware, facade
- Designing Component Boundaries
- Building Components (`wash build`, raw `cargo`)
- Composition Tools: `wac`, wasmCloud
- Build Configuration with `.wash/config.yaml`

## Composition Patterns

Design the WIT for each of these with the `wit-interface-design` skill. This file covers only what each pattern is for.

1. **Simple dependency:** one component imports an interface another exports. `wac plug` wires them together.
2. **Adapter:** a component imports one interface and exports a different one, translating between them, so unmodified components stay compatible.
3. **Middleware:** a component imports and exports the *same* interface (for HTTP, import `wasi:http/handler` and export `wasi:http/handler`, as in the `middleware` world). Because both sides share a shape, middleware chain freely: logging, auth, caching, rate limiting.
4. **Facade:** one component imports several narrow capabilities and exports a simpler API over them.

## Designing Component Boundaries

Good boundaries:
- One clear responsibility per component
- Minimal coupling; components depend on interfaces, never on each other's internals
- Narrow, single-capability interfaces, with coarse operations inside each one
- Domain-aligned

Anti-patterns:
- Chatty interfaces: many small back-and-forth calls across the boundary. Batch, or return records and streams.
- Wide catch-all interfaces that grant more than a caller needs
- Components that assume who implements their imports (host, plugin or peer)

## Building Components

If the project has a `.wash/config.yaml`, use `wash` to build and develop:

```bash
wash build              # Build a valid WebAssembly component
wash dev                # Development with hot-reload
wash build --skip-fetch
```

`wash build` automatically:
- Compiles to the correct `wasm32-wasip3` target
- Handles WIT dependency fetching and binding generation
- Respects `.wash/config.yaml` build configuration

For non-wash projects, target wasip3 (or wasip2 as a fallback):

```bash
cargo build --target wasm32-wasip3 --release
cargo build --target wasm32-wasip2 --release   # fallback
```

## Composition Tools

### wac (WebAssembly Composition)

A modern composition tool using a declarative `.wac` language:

```bash
# Install wac
cargo install wac-cli

# Quick plug: wire one component's exports into another's imports
wac plug component-a.wasm --plug component-b.wasm -o composed.wasm

# Or use a .wac composition file for complex graphs
wac compose composition.wac -o composed.wasm
```

### wasmCloud

wasmCloud deploys components as workloads and wires their imports to hosts, plugins or other components. Deploy with `wash` and the wasmCloud docs at https://wasmcloud.com/docs/.

## Build Configuration with .wash/config.yaml

For projects using `wash`, the `.wash/config.yaml` file in the project root configures custom build commands:

```yaml
# .wash/config.yaml
dev:
  command: make build

build:
  command: make bindgen build
```

This is useful for non-Rust projects (Go, TinyGo, etc.) or projects with custom build steps, controlling how `wash build` and `wash dev` invoke the build.
