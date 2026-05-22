# Documentation

## Requirements

- Every public item must have documentation
- Crate-level docs (`//!` in lib.rs) should provide overview and examples
- Include examples that compile and run

## Function Documentation Sections

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

## Example Guidelines

- Use `?` for error handling, not `unwrap()` or `try!`
- Hide boilerplate with `# ` prefix
- Ensure examples compile with `cargo test --doc`

## Hide Implementation Details

Use `#[doc(hidden)]` to hide internal items from documentation:

```rust
// Public but hidden from docs
#[doc(hidden)]
pub mod __internal {
    // Implementation details needed by macros
}

// Hide re-exports of internal types
#[doc(hidden)]
pub use crate::internal::Helper;
```

Don't expose helper types, internal modules, or implementation details in public documentation.
