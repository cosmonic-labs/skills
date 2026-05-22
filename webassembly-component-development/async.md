# Async in WebAssembly Components

## Contents

- Tokio on wasip2
- wstd (lightweight Wasm-native async stdlib)
- When to use which

## Tokio on wasip2

As of tokio 1.50+, tokio supports `wasm32-wasip2` networking. This means you can use tokio's async runtime and networking APIs in WebAssembly components.

**Requirements:**
- tokio 1.50+
- Build with `RUSTFLAGS="--cfg tokio_unstable"` (required until stabilized)

```toml
# Cargo.toml
[dependencies]
tokio = { version = "1.50", features = ["rt", "net", "macros", "time", "io-util", "sync"] }
```

```bash
# Build with wash (tokio_unstable flag required)
RUSTFLAGS="--cfg tokio_unstable" wash build
```

**What works on wasip2:**
- `tokio::net::TcpListener`, `TcpStream` — full async TCP networking
- `tokio::time` — timers and delays
- `tokio::sync` — channels, mutexes, etc.
- `tokio::io` — async I/O utilities
- Single-threaded runtime (`rt` feature)

**What does NOT work yet:**
- `tokio::net::lookup_host` — DNS lookups require multithreading workaround
- `tokio::fs` — async file I/O (may work with future WASIp3 threading)
- Multi-threaded runtime (`rt-multi-thread`) — WASI has no threading support yet
- `tokio::process` — no process spawning in WASI

```rust
#[tokio::main(flavor = "current_thread")]
async fn main() {
    let listener = tokio::net::TcpListener::bind("0.0.0.0:8080").await.unwrap();
    loop {
        let (stream, _addr) = listener.accept().await.unwrap();
        tokio::spawn(async move {
            handle_connection(stream).await;
        });
    }
}
```

## wstd

[wstd](https://github.com/bytecodealliance/wstd) is a lightweight, WebAssembly-native async standard library built specifically for WASI 0.2. It provides a minimal async runtime, HTTP, networking, and other APIs purpose-built for the Wasm component model.

```toml
# Cargo.toml
[dependencies]
wstd = "0.6"
```

**When to use wstd over tokio:**
- You want a minimal, Wasm-native dependency with no `tokio_unstable` flag required
- You need `wstd::http` client/server APIs (higher-level than raw TCP)
- You want `wstd::rand` for random number generation in Wasm
- You prefer a purpose-built Wasm library over a general-purpose runtime

| Module | Purpose | Example Types |
|--------|---------|---------------|
| `wstd::http` | HTTP client and server | `Client`, `Server` |
| `wstd::io` | Async I/O abstractions | Read, Write traits |
| `wstd::net` | Async TCP networking | `TcpListener` |
| `wstd::time` | Async time interfaces | `Duration`, `Instant` |
| `wstd::rand` | Random number generation | `thread_rng()` |
| `wstd::runtime` | Async event loop | Task spawning |

```rust
#[wstd::main]
async fn main() {
    let client = wstd::http::Client::new();
    let response = client.get("https://example.com").await.unwrap();
    println!("Status: {}", response.status());
}
```

```rust
#[wstd::http_server]
async fn handle(request: wstd::http::IncomingRequest) -> wstd::http::Response {
    wstd::http::Response::new(200, "Hello, World!")
}
```

**Note:** wstd defines its own I/O traits separate from tokio — do not mix wstd and tokio in the same component.
