# Rust Naming Conventions (RFC 430)

## Case Rules

| Item | Convention | Example |
|------|------------|---------|
| Modules | `snake_case` | `my_module` |
| Types & Traits | `UpperCamelCase` | `MyStruct`, `MyTrait` |
| Enum variants | `UpperCamelCase` | `Some`, `None` |
| Functions & Methods | `snake_case` | `do_something` |
| Macros | `snake_case!` | `my_macro!` |
| Statics & Constants | `SCREAMING_SNAKE_CASE` | `MAX_SIZE` |
| Type parameters | Single uppercase letter | `T`, `U`, `E` |
| Lifetimes | Single lowercase letter with `'` | `'a`, `'b` |

## Important Rules

- Acronyms count as one word: use `Uuid`, not `UUID`; `HttpRequest`, not `HTTPRequest`
- In `snake_case`, single-letter words only appear last: `btree_map`, not `b_tree_map`
- Avoid `-rs` or `-rust` suffixes in crate names
- Feature names should be direct: use `std`, not `use-std`

## Conversion Methods

| Prefix | Cost | Ownership | Example |
|--------|------|-----------|---------|
| `as_` | Free | borrowed → borrowed | `as_str()`, `as_bytes()` |
| `to_` | Expensive | borrowed → owned | `to_string()`, `to_vec()` |
| `into_` | Variable | owned → owned (non-Copy) | `into_inner()`, `into_vec()` |

The `mut` qualifier goes where it would in the return type: `as_mut_slice()`, not `as_slice_mut()`.

## Getter Methods

- Omit the `get_` prefix: use `first()`, not `get_first()`
- Reserve `get` for single, obvious values like `Cell::get`

## Iterator Methods

Collections should provide:
```rust
fn iter(&self) -> Iter<'_, T>       // borrowed iteration
fn iter_mut(&mut self) -> IterMut<'_, T>  // mutable iteration
fn into_iter(self) -> IntoIter<T>  // owned iteration
```

Iterator type names must match their producing methods.
