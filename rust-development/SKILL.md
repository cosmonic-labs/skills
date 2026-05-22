---
name: rust-development
description: Expert Rust development guidance covering naming conventions, type safety, error handling, documentation, and best practices based on official Rust guidelines. Use this skill when writing new Rust code, reviewing or modifying existing Rust code, designing Rust APIs or libraries, refactoring Rust codebases, or whenever the user mentions Rust, Cargo, clippy, rustfmt, traits, lifetimes, or `.rs` files.
license: Apache-2.0
---

# Rust Development

This skill enforces new code or code changes to conform to proper Rust guidelines.

## When to Use

Activate this skill when:
- Writing new Rust code
- Reviewing or modifying existing Rust code
- Designing Rust APIs, libraries, or applications
- Refactoring Rust codebases

## Required Gates (Before Submitting Code)

All code must pass these checks:

```bash
cargo +nightly fmt              # format
cargo clippy -- -D warnings     # lint with no warnings
cargo test                      # tests pass
cargo doc --no-deps             # docs build
```

Code that fails clippy or is not properly formatted should not be committed. See [project-and-tooling.md](project-and-tooling.md) for required lints and `rustfmt.toml` config.

## Reference Files

Load the relevant file when working on the matching topic:

| Topic | File |
|-------|------|
| Naming conventions (RFC 430), case rules, conversion methods (`as_`/`to_`/`into_`), getter/iterator method conventions | [naming.md](naming.md) |
| Type safety (newtypes, primitive obsession, bitflags, builders) and trait implementations (`Clone`, `Debug`, `From`/`TryFrom`, `Deref` rules, object safety, Serde behind feature flags) | [types.md](types.md) |
| Error type requirements, `thiserror` for libraries, `anyhow` for applications, propagating with `?`, `let ... else` for early returns | [error-handling.md](error-handling.md) |
| Public-item docs, `# Arguments`/`# Errors`/`# Panics`/`# Safety`/`# Examples` sections, `#[doc(hidden)]` for implementation details | [documentation.md](documentation.md) |
| Minimizing `unsafe`, thread safety, `Drop` must not fail, avoiding allocations, iterators over indexing, `&str` over `String`, `Cow` | [safety-and-performance.md](safety-and-performance.md) |
| API design (no out-parameters, intermediate results, constructors as static methods, generics over concrete types, input validation) and forward compatibility (private fields, sealed traits) | [api-design.md](api-design.md) |
| `Cargo.toml` metadata, module organization, feature flags, test organization, property-based testing, required clippy lints, `rustfmt.toml`, pre-commit checks | [project-and-tooling.md](project-and-tooling.md) |

## Guidelines Sources

These guidelines are based on:
- [Microsoft Rust Guidelines](https://microsoft.github.io/rust-guidelines/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [The Rust Programming Language Book](https://doc.rust-lang.org/book/)
- Rust community best practices

## Code Review Checklist

### Required
- [ ] Code passes `cargo clippy -- -D warnings`
- [ ] Code is formatted with `cargo +nightly fmt`
- [ ] Tests pass with `cargo test`

### Naming ([naming.md](naming.md))
- [ ] Follows RFC 430 case conventions
- [ ] Conversions use `as_`/`to_`/`into_` correctly
- [ ] No `get_` prefix on simple getters
- [ ] Iterator methods follow conventions

### Types ([types.md](types.md))
- [ ] Uses newtypes for distinct concepts
- [ ] Avoids bool/Option parameters with unclear meaning
- [ ] Implements appropriate standard traits
- [ ] Error types implement `Error + Send + Sync`

### Safety ([safety-and-performance.md](safety-and-performance.md))
- [ ] Minimizes `unsafe` code
- [ ] Documents safety invariants
- [ ] Validates inputs at boundaries

### Documentation ([documentation.md](documentation.md))
- [ ] All public items documented
- [ ] Examples compile and demonstrate usage
- [ ] Error/panic conditions documented

### Performance ([safety-and-performance.md](safety-and-performance.md))
- [ ] Avoids unnecessary allocations
- [ ] Uses iterators appropriately
- [ ] Takes references where ownership isn't needed

### Testing ([project-and-tooling.md](project-and-tooling.md))
- [ ] Unit tests for core logic
- [ ] Integration tests for public API
- [ ] Edge cases covered
