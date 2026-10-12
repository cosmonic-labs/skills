# Async in WebAssembly Components

## Contents

- Native async on wasip3
- Rust example: an HTTP handler with a streamed body
- Concurrency model
- wasip2 fallback: tokio and wstd

## Native async on wasip3

On `wasm32-wasip3` async comes from the component model itself, not from a runtime library.

| WIT | Rust (via `wasip3` / `wit-bindgen`) |
|-----|-------------------------------------|
| `async func` export | `async fn` in the `Guest` impl |
| `async func` import | an `async fn` you `.await` |
| `stream<T>` | `wit_stream::new()` returns a writer and a reader |
| `future<T>` | `wit_future::new(...)` returns a writer and a reader |
| background work | `wit_bindgen::spawn_local(async move { ... })` |

Add the bindings with `cargo add wasip3`. Build with `wash build` or `cargo build --target wasm32-wasip3 --release`.

Note: the `wasip3` crate README still says the Rust compiler has no `wasm32-wasip3` target. That is stale; the target is Tier 2.

## Rust example: HTTP handler with a streamed body

Adapted from the `wasip3` crate's `http-proxy` example:

```rust
use wasip3::http::types::{ErrorCode, Fields, Request, Response};
use wasip3::{wit_bindgen, wit_future, wit_stream};

wasip3::http::service::export!(Example);

struct Example;

impl wasip3::exports::http::handler::Guest for Example {
    async fn handle(_request: Request) -> Result<Response, ErrorCode> {
        let (mut body_tx, body_rx) = wit_stream::new();
        let (body_result_tx, body_result_rx) = wit_future::new(|| Ok(None));
        let (response, _transmit) =
            Response::new(Fields::new(), Some(body_rx), body_result_rx);
        drop(body_result_tx);

        // Write the body in the background; the response is returned immediately.
        wit_bindgen::spawn_local(async move {
            let remaining = body_tx.write_all(b"Hello, WASI!".to_vec()).await;
            assert!(remaining.is_empty());
        });
        Ok(response)
    }
}
```

Points to notice:

- The handler is a plain `async fn`; the host drives it.
- The body is a `stream<u8>` plus a completion `future`, so the caller learns whether the transfer succeeded after the stream closes.
- Work that must outlive the handler's return goes in `spawn_local`.

## Concurrency model

- A component instance is **single-threaded and cooperative**. Tasks interleave at `.await` points; there is no parallelism inside one instance.
- Blocking the instance (a long CPU loop, a blocking call) stalls every task in it. Keep CPU-heavy work short or yield.
- Parallelism comes from the host running many instances, not from threads in a component.
- Don't use `std::thread` (unsupported on wasip3 today) or `rayon`.

## wasip2 fallback: tokio and wstd

These target WASI 0.2 and the `wasm32-wasip2` target. Use them only when you must stay on wasip2. For wasip3, use native async above. Support for either on wasip3 was not verified when this file was written; check their docs before depending on it.

**tokio (wasip2):** supports `wasm32-wasip2` networking from 1.50+, built with `RUSTFLAGS="--cfg tokio_unstable"`. Single-threaded runtime only (`flavor = "current_thread"`). Does not work: `rt-multi-thread`, `tokio::process`, `tokio::fs`, `tokio::net::lookup_host`.

**wstd (wasip2):** a minimal async standard library for Wasm components and WASI 0.2 (`wstd::http`, `wstd::net`, `wstd::io`, `wstd::time`). It defines its own I/O traits, so don't mix wstd and tokio in one component.

Don't mix a wasip2 async runtime into a wasip3 component.
