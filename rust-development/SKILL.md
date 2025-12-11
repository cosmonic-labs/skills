# Rust Development

This skill enforces new code or code changes to conform to proper Rust guidelines.

## When to Use

Activate this skill when:
- Writing new Rust code
- Reviewing or modifying existing Rust code
- Designing Rust APIs, libraries, or applications
- Refactoring Rust codebases

## Guidelines Sources

These guidelines are based on:
- [Microsoft Rust Guidelines](https://microsoft.github.io/rust-guidelines/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [The Rust Programming Language Book](https://doc.rust-lang.org/book/)
- Rust community best practices

---

## Naming Conventions (RFC 430)

### Case Rules

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

### Important Rules

- Acronyms count as one word: use `Uuid`, not `UUID`; `HttpRequest`, not `HTTPRequest`
- In `snake_case`, single-letter words only appear last: `btree_map`, not `b_tree_map`
- Avoid `-rs` or `-rust` suffixes in crate names
- Feature names should be direct: use `std`, not `use-std`

### Conversion Methods

| Prefix | Cost | Ownership | Example |
|--------|------|-----------|---------|
| `as_` | Free | borrowed → borrowed | `as_str()`, `as_bytes()` |
| `to_` | Expensive | borrowed → owned | `to_string()`, `to_vec()` |
| `into_` | Variable | owned → owned (non-Copy) | `into_inner()`, `into_vec()` |

The `mut` qualifier goes where it would in the return type: `as_mut_slice()`, not `as_slice_mut()`.

### Getter Methods

- Omit the `get_` prefix: use `first()`, not `get_first()`
- Reserve `get` for single, obvious values like `Cell::get`

### Iterator Methods

Collections should provide:
```rust
fn iter(&self) -> Iter<'_, T>       // borrowed iteration
fn iter_mut(&mut self) -> IterMut<'_, T>  // mutable iteration
fn into_iter(self) -> IntoIter<T>  // owned iteration
```

Iterator type names must match their producing methods.

---

## Type Safety

### Use Newtypes for Distinctions

Create wrapper types to distinguish values with the same underlying type:

```rust
// Bad: easy to confuse
fn set_dimensions(width: u32, height: u32) { }

// Good: compiler catches mistakes
struct Width(u32);
struct Height(u32);
fn set_dimensions(width: Width, height: Height) { }
```

### Avoid Primitive Obsession

Use custom types instead of `bool`, `u8`, or `Option` when meaning is unclear:

```rust
// Bad: what do these bools mean?
Widget::new(true, false)

// Good: self-documenting
enum Size { Small, Large }
enum Shape { Round, Square }
Widget::new(Size::Small, Shape::Round)
```

### Bitflags for Multiple Flags

Use the `bitflags` crate for independent flags:

```rust
use bitflags::bitflags;

bitflags! {
    struct Permissions: u32 {
        const READ = 0b001;
        const WRITE = 0b010;
        const EXECUTE = 0b100;
    }
}
```

### Builders for Complex Construction

For types with many optional parameters:

```rust
struct Config {
    timeout: Duration,
    retries: u32,
    // ...
}

struct ConfigBuilder { /* ... */ }

impl ConfigBuilder {
    fn new() -> Self { /* ... */ }
    fn timeout(mut self, timeout: Duration) -> Self { /* ... */ }
    fn retries(mut self, retries: u32) -> Self { /* ... */ }
    fn build(self) -> Result<Config, Error> { /* ... */ }
}
```

---

## Trait Implementations

### Common Traits to Implement

Types should eagerly implement applicable standard traits:

| Trait | When to Implement |
|-------|-------------------|
| `Clone` | Almost always |
| `Copy` | For small, trivially copyable types |
| `Debug` | Always for public types |
| `Default` | When a sensible default exists |
| `PartialEq`, `Eq` | For comparable types |
| `PartialOrd`, `Ord` | For orderable types |
| `Hash` | For types used as map keys |
| `Display` | For user-facing output |
| `Send`, `Sync` | When thread-safe |

### Conversion Traits

```rust
// Implement From for infallible conversions
impl From<&str> for MyString { }

// Implement TryFrom for fallible conversions
impl TryFrom<&str> for MyValidatedString {
    type Error = ValidationError;
}

// Implement AsRef/AsMut for borrowing
impl AsRef<str> for MyString { }

// Never implement Into/TryInto directly - they're derived automatically
```

### Collection Traits

Collections should implement:
- `FromIterator` - enables `collect()`
- `Extend` - enables `extend()`
- `IntoIterator` - enables `for` loops

### Serialization

Implement Serde traits behind a feature flag:

```toml
[features]
serde = ["dep:serde"]

[dependencies]
serde = { version = "1", features = ["derive"], optional = true }
```

```rust
#[derive(Debug, Clone)]
#[cfg_attr(feature = "serde", derive(serde::Serialize, serde::Deserialize))]
pub struct MyType { }
```

---

## Error Handling

### Error Type Requirements

Error types must:
- Implement `std::error::Error`
- Implement `Send + Sync`
- Implement `Display` with lowercase message, no trailing punctuation
- Never use `()` as an error type

```rust
use std::error::Error;
use std::fmt;

#[derive(Debug)]
pub struct MyError {
    kind: ErrorKind,
    source: Option<Box<dyn Error + Send + Sync>>,
}

impl fmt::Display for MyError {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "operation failed: {}", self.kind)  // lowercase, no period
    }
}

impl Error for MyError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        self.source.as_ref().map(|e| e.as_ref() as _)
    }
}
```

### Use `thiserror` for Library Errors

```rust
use thiserror::Error;

#[derive(Debug, Error)]
pub enum MyError {
    #[error("failed to read file: {path}")]
    ReadFile { path: String, source: std::io::Error },

    #[error("invalid configuration")]
    InvalidConfig,
}
```

### Use `anyhow` for Application Errors

```rust
use anyhow::{Context, Result};

fn main() -> Result<()> {
    let config = read_config()
        .context("failed to read configuration")?;
    Ok(())
}
```

### Propagate with `?`, Not `unwrap()`

```rust
// Bad
let file = File::open(path).unwrap();

// Good
let file = File::open(path)?;

// Good with context
let file = File::open(path)
    .with_context(|| format!("failed to open {}", path.display()))?;
```

---

## Documentation

### Requirements

- Every public item must have documentation
- Crate-level docs (`//!` in lib.rs) should provide overview and examples
- Include examples that compile and run

### Function Documentation Sections

```rust
/// Brief description of what this function does.
///
/// More detailed explanation if needed.
///
/// # Arguments
///
/// * `arg1` - Description of first argument
///
/// # Returns
///
/// Description of return value.
///
/// # Errors
///
/// Returns `Err` if:
/// - condition one occurs
/// - condition two occurs
///
/// # Panics
///
/// Panics if:
/// - some invariant is violated
///
/// # Safety
///
/// This function is unsafe because:
/// - caller must ensure X
///
/// # Examples
///
/// ```
/// use my_crate::my_function;
///
/// let result = my_function(42)?;
/// assert_eq!(result, 84);
/// # Ok::<(), my_crate::Error>(())
/// ```
pub fn my_function(arg1: i32) -> Result<i32, Error> { }
```

### Example Guidelines

- Use `?` for error handling, not `unwrap()` or `try!`
- Hide boilerplate with `# ` prefix
- Ensure examples compile with `cargo test --doc`

---

## Memory and Safety

### Minimize `unsafe`

- Avoid `unsafe` when safe alternatives exist
- Isolate `unsafe` code in small, well-documented functions
- Document all safety invariants

```rust
/// Returns a reference to the element at `index`.
///
/// # Safety
///
/// Caller must ensure `index < self.len()`.
pub unsafe fn get_unchecked(&self, index: usize) -> &T {
    // SAFETY: caller guarantees index is in bounds
    unsafe { self.data.get_unchecked(index) }
}
```

### Prefer Owned Types for Thread Safety

- Return owned types when the data might be used across threads
- Document `Send` and `Sync` bounds explicitly

### Avoid Interior Mutability Unless Necessary

- Prefer `&mut self` methods over `Cell`/`RefCell`
- When using interior mutability, document why it's needed

---

## Performance

### Avoid Unnecessary Allocations

```rust
// Bad: allocates unnecessarily
fn greet(name: &str) -> String {
    format!("Hello, {}!", name.to_string())
}

// Good: reuses the input
fn greet(name: &str) -> String {
    format!("Hello, {name}!")
}
```

### Use Iterators Over Indexing

```rust
// Bad: bounds checking on each access
for i in 0..vec.len() {
    process(vec[i]);
}

// Good: optimized iteration
for item in &vec {
    process(item);
}
```

### Prefer `&str` Over `String` in Parameters

```rust
// Bad: forces caller to allocate
fn process(s: String) { }

// Good: accepts both String and &str
fn process(s: &str) { }

// Good: generic over string types
fn process(s: impl AsRef<str>) { }
```

### Use `Cow` for Conditional Ownership

```rust
use std::borrow::Cow;

fn process(input: &str) -> Cow<'_, str> {
    if needs_modification(input) {
        Cow::Owned(modify(input))
    } else {
        Cow::Borrowed(input)
    }
}
```

---

## API Design

### Functions Should Not Take Out-Parameters

```rust
// Bad: out-parameter
fn get_values(output: &mut Vec<i32>) { }

// Good: return the value
fn get_values() -> Vec<i32> { }
```

### Expose Intermediate Results

Allow callers to avoid duplicate work:

```rust
// Bad: forces re-parsing
fn is_valid(s: &str) -> bool { }
fn parse(s: &str) -> Result<Value, Error> { }

// Good: parse once, check validity from result
fn parse(s: &str) -> Result<Value, Error> { }
// Caller can check is_ok() for validity
```

### Constructors Are Static Methods

```rust
impl MyType {
    // Primary constructor
    pub fn new() -> Self { }

    // Alternative constructors
    pub fn with_capacity(cap: usize) -> Self { }
    pub fn from_parts(a: A, b: B) -> Self { }
}
```

### Smart Pointers Don't Add Inherent Methods

Types implementing `Deref` should not add methods that might conflict with the target type.

---

## Project Structure

### Cargo.toml Metadata

```toml
[package]
name = "my-crate"
version = "0.1.0"
edition = "2021"
rust-version = "1.70"  # MSRV
authors = ["Author Name <email@example.com>"]
description = "Brief description of the crate"
documentation = "https://docs.rs/my-crate"
repository = "https://github.com/user/my-crate"
license = "MIT OR Apache-2.0"
keywords = ["keyword1", "keyword2"]
categories = ["category1"]

[dependencies]
# Use specific versions or version ranges
serde = { version = "1.0", features = ["derive"] }

[dev-dependencies]
# Test-only dependencies
```

### Module Organization

```
src/
├── lib.rs          # Public API, re-exports
├── error.rs        # Error types
├── types.rs        # Core types
├── internal/       # Private implementation
│   ├── mod.rs
│   └── helpers.rs
└── tests/          # Integration test helpers
```

### Feature Flags

- Use feature flags for optional functionality
- Name features clearly without placeholder words
- Document features in Cargo.toml and README

```toml
[features]
default = []
serde = ["dep:serde"]
async = ["dep:tokio"]
```

---

## Testing

### Test Organization

```rust
// Unit tests in the same file
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_basic_functionality() {
        // ...
    }
}

// Integration tests in tests/ directory
// tests/integration_test.rs
```

### Test Naming

```rust
#[test]
fn function_name_condition_expected_result() {
    // e.g., parse_valid_input_returns_value
}
```

### Property-Based Testing

Consider using `proptest` or `quickcheck` for property-based tests:

```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn roundtrip_serialization(value: MyType) {
        let serialized = serialize(&value);
        let deserialized = deserialize(&serialized)?;
        prop_assert_eq!(value, deserialized);
    }
}
```

---

## Clippy and Formatting

### Required Clippy Lints

Enable these in `lib.rs` or `main.rs`:

```rust
#![warn(
    clippy::all,
    clippy::pedantic,
    clippy::nursery,
    missing_docs,
    rust_2018_idioms,
)]

#![allow(
    clippy::module_name_repetitions,  // if needed
)]
```

### Rustfmt Configuration

Create `rustfmt.toml`:

```toml
edition = "2021"
max_width = 100
use_small_heuristics = "Default"
```

### Run Before Committing

```bash
cargo +nightly fmt --check
cargo clippy -- -D warnings
cargo test
cargo doc --no-deps
```

---

## Checklist for Code Review

### Naming
- [ ] Follows RFC 430 case conventions
- [ ] Conversions use `as_`/`to_`/`into_` correctly
- [ ] No `get_` prefix on simple getters
- [ ] Iterator methods follow conventions

### Types
- [ ] Uses newtypes for distinct concepts
- [ ] Avoids bool/Option parameters with unclear meaning
- [ ] Implements appropriate standard traits
- [ ] Error types implement `Error + Send + Sync`

### Safety
- [ ] Minimizes `unsafe` code
- [ ] Documents safety invariants
- [ ] Validates inputs at boundaries

### Documentation
- [ ] All public items documented
- [ ] Examples compile and demonstrate usage
- [ ] Error/panic conditions documented

### Performance
- [ ] Avoids unnecessary allocations
- [ ] Uses iterators appropriately
- [ ] Takes references where ownership isn't needed

### Testing
- [ ] Unit tests for core logic
- [ ] Integration tests for public API
- [ ] Edge cases covered
