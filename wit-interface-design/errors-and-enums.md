# Errors and Enums

## Non-exhaustive enums use the `variant` + `other` pattern

WIT `enum` is closed: adding a case later is a breaking change. If the set of cases may grow, use a `variant` with a trailing `other` case:

```wit
variant error-code {
    access-denied,
    invalid-argument,
    name-unresolvable,
    temporary-resolver-failure,
    permanent-resolver-failure,
    /// A catch-all for errors not captured by the existing variants.
    /// Implementations can use this to extend the error type without
    /// breaking existing code.
    other(option<string>),
}
```

- Use `enum` **only** when the set is truly closed and fixed (e.g. `ip-address-family` ipv4/ipv6, `advice`).
- `other` carries `option<string>` for diagnostics (WASI `sockets`, `http`). For open-ended string-keyed sets, `other(string)` is used (`http` `method`, `scheme`).
- Always put `other` last.
- Consumers must handle `other` as "unknown, treat as generic failure".
- Exhaustive enums are legitimate when the set is genuinely closed: `wasi:cli/stdio` `error-code` (`io`, `illegal-byte-sequence`, `pipe`) is a deliberate exhaustive enum, as are `ip-address-family` and `advice`. Choose `enum` on purpose, and only when you are committing to never adding a case.
- Open sets of errors use `variant` + `other`: `wasi:filesystem`, `wasi:sockets` (including `ip-name-lookup`) and `wasi:http` `error-code` all do.
- `wasmcloud:secrets` `secrets-error` (`upstream`/`io`/`not-found`) has no `other` case; new APIs that may grow should add one.

## Errors

- **Prefer typed errors.** A fallible function returns `result<T, E>` where `E` is a `variant`, `enum` or `record` that names the failure modes callers can act on. It need not be called `error-code`; name it for the domain (`secrets-error`, `header-error`, `request-options-error`).
- **A `string` payload is fine for richness.** A case may carry a `string` (or `option<string>`) with diagnostic detail, as `other(option<string>)` and `secrets-error.upstream(string)` do. Don't make a bare `string` the whole error type, because callers can't branch on it. Document that detail strings are diagnostic only and not stable.
- Open sets end in `other` (see the enum rules above); closed sets may be an `enum`.
- Model cases on the relevant standard when one exists, and document the equivalent on each case (WASI does: `POSIX equivalent: EACCES`).
- Use payload-less `result` / `result<_, E>` for success-or-failure with nothing else to return.
- Include `access-denied` when a capability may be partially granted by the implementer.
