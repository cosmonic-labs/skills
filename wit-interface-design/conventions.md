# Naming, Worlds, Types and Documentation

## Naming

- All identifiers are `kebab-case`. Acronyms are lowercase by default (`ip-address`); use uppercase only when the acronym requires it (`DNS-error`, `TLS-alert-received`, `HTTP-request-denied`, as in `wasi:http`).
- Functions are **verb first** (`resolve-addresses`, `read-via-stream`, `wait-for`).
- **Resource accessors must use the `get-` and `set-` prefixes** (`get-headers`, `set-status-code`, `get-directories`). Don't drop the prefix, even for cheap accessors.
- Interfaces: noun/capability names, singular where they denote one capability (`system-clock`, `random`).
- Package: `namespace:name@semver` (`wasi:clocks@0.3.1`, `wasmcloud:secrets@2.1.0`).
- **Shared types live in an interface named `types`**; operations live in capability-named interfaces that `use types.{...}`.

## Worlds

- Name the aggregate import world `imports` and bundle with `include` (`wasi:cli/imports` includes clocks, filesystem, sockets, random).
- Name the entry-point world after the role: `command` (exports `run`), `service` (exports `handler`), `middleware` (`include service; import handler`).
- Prefer small, targeted worlds per component over importing a giant world. Smaller imports means a smaller grant.
- Use named imports (`import db-password: ...`) when the same interface must be bound to several distinct instances.

## Types

- Prefer `record` for structured data, `variant` for tagged choice, `flags` for bitsets of independent booleans (`descriptor-flags`).
- **Reuse WASI types before defining your own** (see [capability-grants.md](capability-grants.md)): `wasi:clocks/types.duration` for durations and timeouts, `wasi:clocks/system-clock.instant` for points in time, `wasi:sockets/types.ip-address` for addresses. Define a unit alias only for a quantity WASI doesn't cover, and document the unit: `type price = u64; // cents`.
- Prefer `option<T>` over sentinel values. Say what `none` means in the docs (for example "the implementation chooses a default"). Make guarantees such as "never succeeds with 0 results" explicit.
- Use `list<tuple<...>>` only for small, obvious pairs; promote to a `record` when fields need names.
- Don't expose implementation-specific numbers (raw fds, pointers). Use resources.

## Documentation

- Every interface, type, case and function gets a `///` doc comment.
- State preconditions, ownership/borrowing of handles, traps vs. errors, ordering guarantees, and reference standards (`# References` with POSIX/RFC links).
- Say which side validates inputs when construction is cheap and lenient (`wasi:http` `request.new` accepts invalid combinations; `handler.handle` rejects them).
