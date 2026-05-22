# Component Composition and Building

## Contents

- Composition Patterns: Simple Dependency, Adapter, Middleware, Facade
- Designing Component Boundaries
- Building Components (`wash build`, raw `cargo`)
- Composition Tools: `wac`, wasmCloud
- Build Configuration with `.wash/config.yaml`

## Composition Patterns

### 1. Simple Dependency

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

### 2. Adapter Pattern

Adapting one interface to another for compatibility.

```wit
world adapter {
    import old-api/interface;
    export new-api/interface;
}
```

### 3. Middleware Pattern

Components that intercept and process data between other components.

```wit
world http-logger {
    import wasi:http/incoming-handler;
    export wasi:http/outgoing-handler;
    // Logs all HTTP requests/responses
}
```

### 4. Facade Pattern

Single component providing simplified access to multiple components.

```wit
world application-facade {
    import database/client;
    import cache/client;
    import auth/service;
    export api/handler;
}
```

## Designing Component Boundaries

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

## Building Components

If the project has a `.wash/config.yaml`, use `wash` to build and develop:

```bash
wash build              # Build a valid WebAssembly component
wash dev                # Development with hot-reload
wash build --skip-fetch
```

`wash build` automatically:
- Compiles to the correct `wasm32-wasip2` target
- Handles WIT dependency fetching and binding generation
- Respects `.wash/config.yaml` build configuration

For non-wash projects, you can target wasip2:

```bash
cargo build --target wasm32-wasip2 --release
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

### wasmCloud Composition

wasmCloud handles runtime linking automatically via wRPC over NATS. Components are wired together using link definitions and deployed via `wash`.

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
