# Project Structure, Testing, and Tooling

## Contents

- Cargo.toml Metadata
- Module Organization
- Feature Flags
- Test Organization
- Test Naming
- Property-Based Testing
- Required Clippy Lints
- Rustfmt Configuration
- Pre-Commit Checks

## Cargo.toml Metadata

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

## Module Organization

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

## Feature Flags

- Use feature flags for optional functionality
- Name features clearly without placeholder words
- Document features in Cargo.toml and README

```toml
[features]
default = []
serde = ["dep:serde"]
async = ["dep:tokio"]
```

## Test Organization

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

## Test Naming

```rust
#[test]
fn function_name_condition_expected_result() {
    // e.g., parse_valid_input_returns_value
}
```

## Property-Based Testing

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

## Required Clippy Lints

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

## Rustfmt Configuration

Create `rustfmt.toml`:

```toml
edition = "2021"
max_width = 100
use_small_heuristics = "Default"
```

## Required: Run Before Committing

All code must pass these checks before being submitted:

```bash
# Format code (required)
cargo +nightly fmt

# Check for lint errors (required - must pass with no warnings)
cargo clippy -- -D warnings

# Run tests
cargo test

# Verify docs build
cargo doc --no-deps
```

Code that fails clippy or is not properly formatted should not be committed.
