# Types and Trait Implementations

## Contents

- Type Safety: newtypes, primitive obsession, bitflags, builders
- Common Traits to Implement
- Conversion Traits
- Collection Traits
- Serialization
- Object Safety for Trait Objects
- Only Smart Pointers Implement `Deref`
- Debug Output Should Never Be Empty

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

## Common Traits to Implement

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

## Conversion Traits

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

## Collection Traits

Collections should implement:
- `FromIterator` - enables `collect()`
- `Extend` - enables `extend()`
- `IntoIterator` - enables `for` loops

## Serialization

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

## Object Safety for Trait Objects

If a trait may be useful as a trait object (`dyn Trait`), ensure it's object-safe:

```rust
// Object-safe: can be used as dyn Trait
trait Drawable {
    fn draw(&self);
    fn bounds(&self) -> Rect;
}

// NOT object-safe: has generic method
trait Processor {
    fn process<T>(&self, item: T);  // Generics not allowed
}

// NOT object-safe: returns Self
trait Clonable {
    fn clone(&self) -> Self;  // Self by value not allowed
}
```

## Only Smart Pointers Implement Deref

Only implement `Deref` and `DerefMut` for smart pointer types:

```rust
// Good: Smart pointer implementing Deref
struct MyBox<T>(T);

impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T { &self.0 }
}

// Bad: Non-pointer type implementing Deref
struct Email(String);
impl Deref for Email {
    type Target = str;  // Don't do this - use AsRef instead
    fn deref(&self) -> &str { &self.0 }
}
```

Use `AsRef` instead for non-pointer types that provide access to inner data.

## Debug Output Should Never Be Empty

`Debug` implementations should always produce meaningful output:

```rust
// Bad: Empty debug output
impl fmt::Debug for MyType {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        Ok(())  // Produces nothing
    }
}

// Good: At minimum show type name
impl fmt::Debug for MyType {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        f.debug_struct("MyType").finish()
    }
}
```
