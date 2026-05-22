# Error Handling

## Contents

- Error Type Requirements
- `thiserror` for library errors
- `anyhow` for application errors
- Propagating with `?`
- `let ... else` for Early Returns

## Error Type Requirements

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

## Use `thiserror` for Library Errors

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

## Use `anyhow` for Application Errors

```rust
use anyhow::{Context, Result};

fn main() -> Result<()> {
    let config = read_config()
        .context("failed to read configuration")?;
    Ok(())
}
```

## Propagate with `?`, Not `unwrap()`

```rust
// Bad
let file = File::open(path).unwrap();

// Good
let file = File::open(path)?;

// Good with context
let file = File::open(path)
    .with_context(|| format!("failed to open {}", path.display()))?;
```

## Use `let ... else` for Early Returns

Extract values and exit early to keep the happy path unindented:

```rust
// Bad: Nested conditionals
fn process(value: Option<String>) -> Result<(), Error> {
    if let Some(s) = value {
        if !s.is_empty() {
            // happy path buried in nesting
            do_work(&s)?;
        }
    }
    Ok(())
}

// Good: let-else keeps happy path flat
fn process(value: Option<String>) -> Result<(), Error> {
    let Some(s) = value else {
        return Ok(());
    };

    if s.is_empty() {
        return Ok(());
    }

    do_work(&s)?;
    Ok(())
}
```

Benefits:
- Keeps the main logic at the top level of indentation
- Makes early exit conditions explicit
- Reduces rightward drift in complex functions
