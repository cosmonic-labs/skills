# Memory, Safety, and Performance

## Contents

- Minimize `unsafe`
- Thread Safety: owned types, interior mutability
- Destructors Must Not Fail
- Avoid Unnecessary Allocations
- Use Iterators Over Indexing
- Prefer `&str` Over `String` in Parameters
- Use `Cow` for Conditional Ownership

## Minimize `unsafe`

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

## Prefer Owned Types for Thread Safety

- Return owned types when the data might be used across threads
- Document `Send` and `Sync` bounds explicitly

## Avoid Interior Mutability Unless Necessary

- Prefer `&mut self` methods over `Cell`/`RefCell`
- When using interior mutability, document why it's needed

## Destructors Must Not Fail

`Drop` implementations must not panic or fail:

```rust
impl Drop for Resource {
    fn drop(&mut self) {
        // Bad: Can panic
        self.file.flush().unwrap();

        // Good: Log error but don't panic
        if let Err(e) = self.file.flush() {
            eprintln!("warning: failed to flush: {e}");
        }
    }
}
```

If cleanup can fail, provide an explicit `close()` method that returns `Result`.

## Avoid Unnecessary Allocations

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

## Use Iterators Over Indexing

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

## Prefer `&str` Over `String` in Parameters

```rust
// Bad: forces caller to allocate
fn process(s: String) { }

// Good: accepts both String and &str
fn process(s: &str) { }

// Good: generic over string types
fn process(s: impl AsRef<str>) { }
```

## Use `Cow` for Conditional Ownership

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
