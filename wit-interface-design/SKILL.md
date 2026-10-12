---
name: wit-interface-design
description: Design and review WIT (WebAssembly Interface Types) interfaces, worlds and packages so they are capability-scoped, async, virtualizable and portable. Use this skill whenever the user is designing or reviewing a `.wit` file, adding a function, resource or interface to a WIT package, deciding what a component should import or export, designing a plugin or provider API (for example `wasmcloud:` packages), choosing between `enum` and `variant`, modeling errors, naming interfaces or resources, versioning a WIT package, or asking "how should I model this in WIT?", even if they never say "interface design". For building, composing, or running components, use webassembly-component-development instead.
license: Apache-2.0
---

# WIT Interface Design

Design the contract first, then write the code. A good WIT API reads as a **capability grant**: someone can look at a component's imports and know exactly what it is allowed to do. The rules below are modeled on the WASI 0.3.x proposals (`wasi:cli`, `clocks`, `filesystem`, `http`, `random`, `sockets`) and `wasmcloud:secrets@2.1.0`.

## Start with the four areas

Every component's contract has four parts. Settle each one before writing WIT.

| Area | Question | Where it lives in WIT |
|------|----------|-----------------------|
| **Triggers** | What events, requests or streams start the component's work? | **Exports** |
| **Responses** | What does it return, and who owns the data? | Return types, `own` vs `borrow`, `stream` / `future` |
| **Side-effects** | What can it change outside itself? | **Imports**, one per capability |
| **Errors** | Which failures does it deliberately return? | `result<T, E>` with a typed `E` |

Every component is a reactor: each export is a reaction to something (`handle` a request, `run`, `on-message`). Design components as handlers that are given work, not scripts that fetch it. The world is the whole contract: exports say what starts it, imports say what it may do. Traps mean bugs or violated preconditions; expected failures belong in the signature.

A good WIT API is strictly decoupled (names a capability, not a vendor), virtualizable, mockable (a test double is just another component implementing the interface), and portable (maps cleanly to any language: no exceptions, callbacks, pointers, OS paths, or threads).

## Core rules and why

**Imports are grants.** Write "the guest may ___" for every interface (resolve names, read the system clock, `get-directories` from preopens, reveal a secret). If the sentence needs "and", split the interface. This keeps least privilege auditable from the world alone. One capability per interface, named for the capability, not the implementation. See [capability-grants.md](capability-grants.md).

**One singular thing = a labeled import.** To give a guest one secret, import `db-password: wasmcloud:secrets/secret` with `get: async func() -> secret`, not `get(key: string)`. There is no key to forge, and the runtime can check the binding before instantiation. Also in [capability-grants.md](capability-grants.md).

**Import WASI; never reinvent it.** Clocks, random, filesystem, sockets, name lookup, HTTP and stdio already exist. Reuse its interfaces and types (`duration`, `ip-address`) so grants stay legible and components stay interoperable. See [capability-grants.md](capability-grants.md).

**Keep interfaces granular.** Split by independently grantable power (read vs write, lookup vs reveal, consumer vs producer). A guest that only needs one power shouldn't receive another.

**Async by default.** Declare `async func` unless the call is time-sensitive and must be sync (`system-clock.now`). Cheap is not a reason to be sync: changing sync to async later breaks every consumer. This applies to resource methods too.

**Every API must be virtualizable.** Design as if another guest component implements it, never "something the host provides". Document in terms of implementer and caller. Give the consumer and provider sides symmetric shapes (as `wasi:http` `client.send` and `handler.handle`) so components chain as middleware. See [async-and-resources.md](async-and-resources.md) and [http-derived-patterns.md](http-derived-patterns.md).

**Resources are singular nouns** (`descriptor`, `secret`, `tcp-socket`). Collections are `list<secret>`. Consume-by-move for one-shot operations, and document parent/child lifetimes and mutability.

**Open sets use `variant` + `other`; closed sets use `enum`.** An `enum` can never gain a case without breaking consumers, so choose it only for sets you commit to never extending (`ip-address-family`). Otherwise end a `variant` with `other(option<string>)`. See [errors-and-enums.md](errors-and-enums.md).

**Typed errors.** Return `result<T, E>` where `E` names the failure modes callers can act on. A `string` payload is fine for detail; a bare `string` error is not. Model cases on an industry standard (IANA, RFC, POSIX) when one exists. See [errors-and-enums.md](errors-and-enums.md).

**Version for compatibility.** `@since` on every interface; `@deprecated` instead of deleting; break only on a minor bump at `0.x` or a major bump at `1.0+`. See [versioning.md](versioning.md).

**Name consistently.** Kebab-case, verb-first functions, `get-` / `set-` on resource accessors, shared types in an interface named `types`, uppercase acronyms only when required (`DNS-error`). Details, worlds, types and documentation in [conventions.md](conventions.md).

## Review checklist

1. Can you state triggers (exports), responses and ownership, side-effects (imports) and errors in one sentence each?
2. Does it reimplement something WASI already provides? Import it instead.
3. Is every function `async` (except time-sensitive ones), and could another guest implement the interface?
4. Can each import be read as "the guest may ___"? Does any interface grant more than one independent power?
5. Do open enums and errors use `variant` with a trailing `other`? Are errors typed?
6. Are all resources singular nouns, with ownership and lifetimes documented?
7. Could a single-value grant be a labeled import instead of a key parameter?
8. Is `@since` on every interface, with retired items `@deprecated` rather than removed?
9. Is every interface, type, case and function documented with `///`?

## Reference files

| Topic | File |
|-------|------|
| Capability grants, labeled singular imports, WASI reuse table, granularity | [capability-grants.md](capability-grants.md) |
| Async rule, virtualization, resources, resource methods | [async-and-resources.md](async-and-resources.md) |
| `enum` vs `variant` + `other`, error design | [errors-and-enums.md](errors-and-enums.md) |
| Patterns from `wasi:http@0.3.1`: symmetric interfaces, bodies, construction, parent/child | [http-derived-patterns.md](http-derived-patterns.md) |
| Naming, worlds, types, documentation | [conventions.md](conventions.md) |
| `@since`, `@deprecated`, breaking-change policy | [versioning.md](versioning.md) |

## Related skills

- `webassembly-component-development`: building, composing and running components.
- `wash`: the `wash build` / `wash dev` toolchain.
