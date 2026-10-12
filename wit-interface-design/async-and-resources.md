# Async, Virtualization and Resources

## Async by default

Target WASI p3 (0.3.x) component-model async.

- Declare operations `async func` by default. Any call that may do I/O, wait on its implementer, cross a component boundary, or reach a remote backend must be async. Since any implementation may be another component (see "Every API must be virtualizable" below), that is every call except the time-sensitive ones below (`resolve-addresses`, `handle`, `send`, `secrets.get`, `reveal`, `wait-for`).
- Use `stream<T>` and `future<result<_, error-code>>` for bulk/incremental data and completion.
- Don't add `*-async` or `poll`/`pollable` variants; p3 replaces them.
- Don't invent callbacks, polling resources, or `wasi:io` pollable patterns (p2 style).
- Exports that handle events are `async func` too (`run: async func() -> result`, `handle: async func(request)`).
- **Sync is the exception, not the cheap-call shortcut.** "It's cheap" is not a reason to be sync. A function is sync only when it **must** be: time-sensitive, where awaiting would distort the result, e.g. `system-clock.now` and `monotonic-clock.now`. Everything else is `async`.
- When in doubt, make it async: changing sync to async later is breaking.

## Every API must be virtualizable

Design every interface as if **another guest component will implement it**, never as "something the host provides". Host vs guest is a deployment detail: any import may be satisfied by a host, a plugin, or a peer component, and any of them can sit in the middle (middleware, mock, policy wrapper, test double).

- Describe and document APIs in terms of the **implementer** and the **caller**, not "the host".
- Don't assume implementer-side state, trust or I/O. Don't rely on shared memory, ambient host identity, or host-only types.
- Symmetric consumer/provider shapes ([http-derived-patterns.md](http-derived-patterns.md)) exist for this reason.
- Because any implementer may be a different component, calls cross a component boundary and may suspend, so the async rule (above) applies to resource methods too, with no "in-memory" exemption.

## Resources are singular nouns

`resource descriptor`, `resource request`, `resource tcp-socket`, `resource secret`: never `descriptors`, `secrets`, `request-list`. A resource is one handle; collections are `list<secret>`.

- Name by what it **is** (noun), not what it does. Functions/methods carry verbs.
- Methods are verbs on the resource; constructors/`static` factories create it (`create: static func(...) -> result<tcp-socket, error-code>`).
- Drop-in the `resource` inside the interface that grants it; share across interfaces via a `types` interface and `use`.

## Resource methods and async

Resources can have async methods; this is not hypothetical.

- **WASI 0.3.1 does it.** In `wasi:filesystem`, `descriptor` methods are `async func`, including cheap-looking ones: `get-type`, `get-flags`, `stat`, `open-at`, `set-size`, `sync`. In `wasi:sockets`, `tcp-socket.connect` and `udp-socket.send`/`receive` are async. Syntax: `stat: async func() -> ...` inside the `resource` block.
- **Runtime:** a resource method is an ordinary function whose first parameter is the handle (`[method]descriptor.stat`), so async lowering and lifting apply as for any function. Wasmtime implements the Component Model async ABI in `crates/wasmtime/src/runtime/component/concurrent.rs`: an event loop per `Store`, `call_concurrent` / `run_concurrent` for calls into a guest, `func_wrap_concurrent` with an `Accessor` for host functions, and reader/producer types for `future<T>` and `stream<T>`.
- **Rule:** don't split resources into "async because host-backed" and "sync because in-memory". Per the virtualization rule, any resource may be implemented by another guest, so **all methods on a resource are `async func`** (including getters and setters), except the time-sensitive cases in 1c.
- `wasi:http` keeps `fields`, `request` and `response` methods sync. That is a property of WASI's current design, not a pattern to copy in new APIs.
- **Open question:** `request` has about 15 methods. Is that fine because granularity applies to interfaces (`client` vs `handler`) and not to resources?
