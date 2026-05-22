# Testing, Debugging, and Performance

## Contents

- Testing: host harness, mocks, wash integration, feature flags, env setup
- Debugging workflow, common error patterns, tools
- Performance: minimize cross-component calls, streaming, batching

## Testing Strategies

### 1. Lightweight Host Harness

```rust
use wasmtime::{Engine, Store, component::Component};

#[cfg(test)]
mod tests {
    #[test]
    fn test_component() -> Result<()> {
        let engine = Engine::default();
        let mut store = Store::new(&engine, ());
        let component = Component::from_file(&engine, "component.wasm")?;
        // Instantiate and test...
        Ok(())
    }
}
```

### 2. Mock Components

```rust
wit_bindgen::generate!({
    world: "mock-database",
    exports: {
        "database:client/query": MockDatabase,
    }
});

struct MockDatabase;

impl database::client::Query for MockDatabase {
    fn execute(query: String) -> Result<Vec<Row>, Error> {
        Ok(vec![/* test rows */])
    }
}
```

### 3. Integration Tests with wash

```bash
wash dev  # Start development environment
curl http://localhost:8080/api/test  # Test the component's HTTP interface
```

### 4. Feature Flags for Host-Specific Code

```rust
#[cfg(target_family = "wasm")]
mod wasm_impl {
    pub fn get_data() -> Vec<u8> {
        wasi::filesystem::read("/data/file.bin")
    }
}

#[cfg(not(target_family = "wasm"))]
mod native_impl {
    pub fn get_data() -> Vec<u8> {
        vec![1, 2, 3, 4] // Test data
    }
}
```

### 5. Environment Setup

```rust
use wasmtime_wasi::{WasiCtxBuilder, Dir};

#[test]
fn test_with_filesystem() -> Result<()> {
    let test_dir = tempfile::tempdir()?;
    std::fs::write(test_dir.path().join("test.txt"), "test data")?;

    let wasi = WasiCtxBuilder::new()
        .env("TEST_MODE", "true")?
        .preopened_dir(
            Dir::open_ambient_dir(test_dir.path(), ambient_authority())?,
            "/data",
        )?
        .build();

    // Test component...
    Ok(())
}
```

## Debugging

### Debugging Workflow

1. **Check target compatibility:**
   ```bash
   rustc --print target-list | grep wasi
   ```

2. **Fetch WIT dependencies:**
   ```bash
   # WIT deps are fetched automatically by wash build
   # For manual management, use the wkg CLI
   wkg wit fetch
   ```

3. **Inspect component imports:**
   ```bash
   wasm-tools component wit your-component.wasm
   ```

4. **Check for threading issues:**
   - Review dependencies for `rayon`, `tokio` (threaded), `std::thread`

### Common Error Patterns

| Error | Cause | Solution |
|-------|-------|----------|
| "operation not supported on this platform" | Threading issue | Remove threaded dependencies |
| "unknown import: wasi:cli/..." | Runtime doesn't support interface | Update runtime or remove dependency |
| WIT version mismatch | Version conflict | Re-fetch deps with `wkg wit fetch` or rebuild with `wash build` |
| Component instantiation failed | Adapter/runtime incompatibility | Check adapter version |

### Tools

```bash
# Inspect component structure
wasm-tools print component.wasm

# Extract WIT interfaces
wasm-tools component wit component.wasm

# Validate component
wasm-tools validate component.wasm

# Check component info (wasmCloud)
wash inspect component.wasm
```

## Performance Optimization

### Minimize Cross-Component Calls

```rust
// Bad: Multiple fine-grained calls
let name = user_service.get_name(id)?;
let email = user_service.get_email(id)?;

// Good: Single coarse-grained call
let user = user_service.get_user(id)?;
```

### Use Streaming for Large Data

```wit
interface data-processor {
    resource stream {
        read-chunk: func() -> option<list<u8>>;
    }
    process-stream: func(input: stream) -> result<_, error>;
}
```

### Batch Operations

```rust
// Bad: Many individual calls
for item in items {
    database.insert(item)?;
}

// Good: Single batch operation
database.insert_batch(items)?;
```
