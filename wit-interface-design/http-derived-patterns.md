# Patterns Derived from wasi:http@0.3.1

## Patterns derived from `wasi:http@0.3.1`

**Symmetric interfaces for composability.** Give the consumer side and provider side of a capability the same shape and types. `wasi:http` `client.send` and `handler.handle` are both `async func(request) -> result<response, error-code>`, one function per interface. A `middleware` world is `include service; import handler;`: it imports and exports the same interface, so components chain with no adapters. A `client.send` import can be linked directly to a `handler.handle` export, bypassing the network, so the caller can't tell a host from a peer component.

> Why two interfaces and not one imported twice: the WASI docs say the `client`/`handler` duplication "is currently necessary because some Component Model tooling (including WIT itself) is unable to represent a component importing two instances of the same interface." It is a workaround for the lack of `implements`, and may be resolved by 1.0. Today, when you need two instances of one interface, either duplicate the shape under role names as `wasi:http` does, or use labeled imports (`import db-password: wasmcloud:secrets/secret;`, see [capability-grants.md](capability-grants.md)) where the toolchain supports it. Don't hand-roll a different shape just to avoid duplication.

**Bodies are `stream<u8>` plus a completion `future<result<...>>`.** Model bulk or incremental data as `option<stream<u8>>` (`none` means zero-length) plus a `future<result<option<trailers>, error-code>>`. The reader gets `tuple<stream<u8>, future<result<...>>>` and awaits the future after the stream closes. A closed stream alone does not mean the transfer succeeded.

**`static new` vs `constructor`.** Use `constructor` for cheap, infallible objects (`fields`, `request-options`). When the outcome shows up later, use `new: static func(...) -> tuple<resource, future<result<_, error-code>>>`; the future resolves with the result of transmission or completion.

**Consume by move.** For operations that can happen only once, take ownership: `consume-body: static func(this: request, ...) -> ...`. The type system then enforces single use. Borrows acquired earlier (such as headers) remain valid.

**Parent/child resources.** A child resource has a lifetime tied to its parent; document that it must be dropped before the parent or moved with it (`request.get-options`). Mutability depends on how the resource was obtained: `request.get-headers` returns an immutable `fields`, while `constructor`/`from-list`/`clone` returns a mutable one. Mutators on an immutable resource fail with an `immutable` error case. Document lifetime and mutability on every resource that hands out sub-resources.

**Role aliases.** One resource, named by role in signatures: `type headers = fields; type trailers = fields;`.

**`list<u8>` where input may not be valid UTF-8.** `field-value = list<u8>` even though values "should be ASCII". Use `string` only where valid UTF-8 is guaranteed (`field-name`).

**Worlds import only what their role needs.** `wasi:http` `service` includes clocks and random, imports stdin/stdout/stderr and `client`, and exports `handler`: no filesystem, no raw sockets. A world's imports are its grant (see [capability-grants.md](capability-grants.md)).

**Prefer industry standards when modelling an API.** `wasi:http` `error-code` is grounded in the IANA HTTP Proxy Error Types, link the standard in the docs, uses payload records for structured cases (`DNS-error-payload`, `TLS-alert-received-payload`), and ends in a catch-all with `option<string>`. Do the same: derive cases, names and semantics from the relevant RFC/IANA/POSIX source rather than inventing them. Note: its catch-all is named `internal-error`; ours is `other` (see [errors-and-enums.md](errors-and-enums.md)).
