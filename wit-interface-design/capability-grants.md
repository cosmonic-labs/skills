# Capability Grants, WASI Reuse and Granularity

## Interfaces are capability grants

Every `import` in a world is a permission. Read the imports aloud as sentences:

| Import | Reads as |
|--------|----------|
| `wasi:sockets/ip-name-lookup` | the guest **may** resolve names (`resolve-addresses`) |
| `wasi:clocks/system-clock` | the guest **may** read the system clock |
| `wasi:filesystem/preopens` | given preopens, the guest **may** `get-directories` |
| `wasmcloud:secrets/reveal` | the guest **may** reveal a secret |
| `wasi:random/insecure` | the guest **may** get insecure random bytes |

Design consequences:

- **One capability per interface.** If two operations should be granted independently, they belong in separate interfaces. WASI splits `random` / `insecure` / `insecure-seed`, and `stdin` / `stdout` / `stderr`, for this reason.
- **Split read from reveal/mutate.** Obtaining a handle and exercising the dangerous power on it are different grants (e.g. holding a `secret` vs. revealing it).
- **Name interfaces after the capability** (`ip-name-lookup`, `system-clock`, `preopens`), not after the implementation (`dns-client-v2`, `host-utils`).
- **Don't bundle unrelated powers** into a catch-all (`utils`, `common`, `host`). A guest that needs one function would be granted all.
- **Test:** can you write the sentence "the guest may ___" for the interface in one clause? If it needs "and", split it.
- Keep the world the unit of review: a reader should learn what a component can do from its `import` list alone.

### Singular resource access via labeled imports

To give a guest one specific secret (or config value, bucket, etc.), import a **labeled instance** of an interface that has a zero-argument getter, instead of passing a key string. This is how `wasmcloud:secrets@2.1.0` does it (source: `wasmCloud/wasmCloud` `wit/secrets/wit/world.wit`):

```wit
package wasmcloud:secrets@2.1.0;

/// Dynamic lookup: for components whose keys are only known at runtime.
interface store {
  resource secret;
  get: async func(key: string) -> result<secret, secrets-error>;
}

/// Reveal is split from lookup so an implementer or policy layer can audit/restrict it separately.
interface reveal {
  use store.{secret, secret-value};
  reveal: async func(s: borrow<secret>) -> secret-value;
}

/// A single secret, identified by the label it is imported under.
interface secret {
  use store.{secret};
  /// No `key` parameter: which secret this is comes entirely from the label.
  get: async func() -> secret;
}
```

```wit
world my-service {
  import db-password: wasmcloud:secrets/secret;
  import wasmcloud:secrets/reveal;
}
```

Why this is better than `store.get(key)`:

- **Least privilege:** the guest holds exactly one secret and cannot ask for another; there is no key to forge.
- **Static check:** the label is part of the component's type, so the linker or runtime verifies every labeled secret has a binding from the import list alone, before instantiation, instead of discovering `not-found` at runtime.
- **Auditable grant:** the world lists which secrets and whether the guest may `reveal` them.
- Keep the dynamic `store` interface only for components whose key set is genuinely unknown until runtime.
- Apply the same shape to other singular things (a config value, a bucket, a queue): `interface bucket { get: async func() -> bucket; }` imported as `import uploads: ...`.

## Depend on WASI, never reinvent

If a capability already exists in WASI (wasip3 / 0.3.x), **import it; do not define your own.** Clocks, randomness, filesystem, sockets, name lookup, HTTP client/server, stdio, config:

| Need | Import |
|------|--------|
| wall-clock time | `wasi:clocks/system-clock` |
| elapsed time / sleep / timeout | `wasi:clocks/monotonic-clock` (`wait-for`, `wait-until`) |
| random bytes | `wasi:random/random` (or `insecure` / `insecure-seed`) |
| DNS | `wasi:sockets/ip-name-lookup` |
| TCP/UDP | `wasi:sockets/types` |
| outbound HTTP / handling HTTP | `wasi:http/client`, `wasi:http/handler` |
| files | `wasi:filesystem/types` + `preopens` |
| logs/stdio | `wasi:cli/stdout`, `stderr` |

- Reuse WASI **types** too (`use wasi:clocks/types.{duration}`, `wasi:sockets/types.{ip-address}`) rather than redefining `duration`, `ip-address`, `headers`.
- Pin the WASI version your package depends on (`wasi:clocks@0.3.1`) and note it in the package docs.
- If a function takes a timestamp or timeout, take WASI's `duration` or `instant`; do not invent `timeout-ms: u64`.
- Only define a new interface for a capability WASI does not cover (e.g. `wasmcloud:secrets`). Prefer contributing a gap upstream.
- Don't wrap a WASI interface with a thin pass-through of your own. Grants then stop being legible: the world should say `import wasi:clocks/system-clock`, not `import my:app/time`.

## Granular, fine-grained interfaces

- Small interfaces, few functions each. Split by *independently grantable power*, not by topic.
- Prefer many narrow imports over one wide one: `wasi:random` has `random`, `insecure`, `insecure-seed`; `wasmcloud:secrets` has `store`, `reveal`, `secret`; `wasi:cli` has `stdin`, `stdout`, `stderr` as three interfaces.
- Separate: read vs write, lookup vs reveal, observe vs control, single-instance vs dynamic-keyed, consumer vs producer.
- A guest that only reads should not be able to import a write function by importing the same interface.
- Keep shared types in a `types` interface so narrow interfaces can `use` them without being bundled.
- Aggregate with a `world` (`include`), never by merging interfaces.
- Granularity is about **what can be granted**, not about chatty calls. Keep each operation within a capability coarse enough to avoid per-field round trips across a component boundary (batch, return records or streams), but split capabilities into separate interfaces.
