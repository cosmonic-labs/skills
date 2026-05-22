# API Design and Forward Compatibility

## Contents

- Functions Should Not Take Out-Parameters
- Expose Intermediate Results
- Constructors Are Static Methods
- Smart Pointers Don't Add Inherent Methods
- Use Generics to Minimize Assumptions
- Validate Function Arguments
- Use Private Fields (Forward Compatibility)
- Sealed Traits

## Functions Should Not Take Out-Parameters

```rust
// Bad: out-parameter
fn get_values(output: &mut Vec<i32>) { }

// Good: return the value
fn get_values() -> Vec<i32> { }
```

## Expose Intermediate Results

Allow callers to avoid duplicate work:

```rust
// Bad: forces re-parsing
fn is_valid(s: &str) -> bool { }
fn parse(s: &str) -> Result<Value, Error> { }

// Good: parse once, check validity from result
fn parse(s: &str) -> Result<Value, Error> { }
// Caller can check is_ok() for validity
```

## Constructors Are Static Methods

```rust
impl MyType {
    // Primary constructor
    pub fn new() -> Self { }

    // Alternative constructors
    pub fn with_capacity(cap: usize) -> Self { }
    pub fn from_parts(a: A, b: B) -> Self { }
}
```

## Smart Pointers Don't Add Inherent Methods

Types implementing `Deref` should not add methods that might conflict with the target type.

## Use Generics to Minimize Assumptions

Accept generic parameters instead of concrete types when possible:

```rust
// Bad: Requires specific type
fn read_config(file: File) -> Config { }

// Good: Accepts any reader
fn read_config(reader: impl Read) -> Config { }

// Good: Accepts any string-like type
fn greet(name: impl AsRef<str>) {
    println!("Hello, {}", name.as_ref());
}
```

## Validate Function Arguments

Functions should validate their inputs rather than trusting callers:

```rust
// Bad: Trusts caller to validate
fn get_element(slice: &[u8], index: usize) -> u8 {
    slice[index]  // Panics on invalid index
}

// Good: Returns Option for invalid input
fn get_element(slice: &[u8], index: usize) -> Option<u8> {
    slice.get(index).copied()
}

// Good: Use types that enforce validity
fn process_nonempty(items: NonEmpty<Item>) { }
```

## Use Private Fields

Structs should have private fields to allow future changes without breaking users:

```rust
// Bad: Public fields lock in representation
pub struct Point {
    pub x: f64,
    pub y: f64,
}

// Good: Private fields with accessors
pub struct Point {
    x: f64,
    y: f64,
}

impl Point {
    pub fn new(x: f64, y: f64) -> Self { Self { x, y } }
    pub fn x(&self) -> f64 { self.x }
    pub fn y(&self) -> f64 { self.y }
}
```

## Sealed Traits

Use sealed traits to prevent downstream implementations while allowing use:

```rust
mod private {
    pub trait Sealed {}
}

/// This trait is sealed and cannot be implemented outside this crate.
pub trait MyTrait: private::Sealed {
    fn method(&self);
}

// Only types in this crate can implement MyTrait
impl private::Sealed for MyType {}
impl MyTrait for MyType {
    fn method(&self) { }
}
```
