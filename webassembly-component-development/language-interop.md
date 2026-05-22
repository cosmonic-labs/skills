# Language Interoperability

## Cross-Language Composition

**Example:** Python library used from Rust

```wit
// python-ml-lib/wit/world.wit
package ml:inference;

interface model {
    predict: func(features: list<float32>) -> list<float32>;
}

world ml-component {
    export model;
}
```

```rust
// Rust component using Python ML library
wit_bindgen::generate!({
    world: "app",
    path: "wit",
});

impl Guest for MyApp {
    fn process(data: Vec<f32>) -> Vec<f32> {
        ml::inference::model::predict(&data)
    }
}
```

## Supported Languages

- **Rust** - First-class support via `wash build`
- **Python** - Via `componentize-py`
- **JavaScript/TypeScript** - Via `componentize-js` (experimental, uses SpiderMonkey)
- **Go** - Via TinyGo 0.33+ with native WASI P2 support, plus Go Component SDK
- **C/C++** - Via WASI SDK
- **C#** - Experimental support

## Language Compatibility Considerations

**Data Types:**
- WIT provides common types that map to each language
- Complex types (records, variants, lists) are automatically converted
- Strings are always UTF-8

**Performance:**
- Cross-language calls have overhead (serialization/deserialization)
- Keep interfaces coarse-grained to minimize calls
- Pass handles/resources instead of large data when possible

**Error Handling:**
- Use WIT `result` types for cross-language error propagation
- Each language maps `result<T, E>` to its native error handling
