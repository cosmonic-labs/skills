---
name: wash
description: "wasmCloud Shell (wash) CLI for building, running, and managing WebAssembly components — scaffolding projects, hot-reload development, WIT dependency management, and component deployment. Use when working with wasmCloud, wash commands, WebAssembly components, WIT definitions, or component plugins."
license: Apache-2.0
metadata:
  tags:
    - wasmcloud
    - cli
    - tooling
    - development
    - webassembly
---

# wash - wasmCloud Shell

## Prerequisites

- `wash` version 2.0.0 or later (including release candidates like 2.0.0-rc.X)
- Verify version with: `wash --version`

## Important Notes

**Do not use `cargo component` commands** (like `cargo component build`) when working with wasmCloud projects. Always use `wash` commands instead:
- Use `wash build` or `wash dev` for building components
- For plugins, build with `wash build --skip-fetch` (since wasmcloud:wash interface isn't published)

## Core Commands

### wash dev

Hot-reload development loop — rebuilds your component automatically on file changes:

```bash
wash dev
# Expected: "watching for changes..." output, component accessible at localhost
```

### wash wit update

Resolves mismatched WIT (WebAssembly Interface Types) definitions:

```bash
wash wit update
# Fixes version mismatches in wit/deps/ — re-run wash build after
```

## Common Workflows

### Starting a New Project

1. `wash new component hello --template-name hello-world-rust` — scaffold from a template
2. `wash build` — compile to a valid WebAssembly component
3. Verify: check for `.wasm` output in `build/`
4. `wash dev` — start hot-reload development loop

### Fixing WIT Definition Conflicts

If you encounter errors about mismatched WIT definitions:

1. `wash wit update` — synchronize WIT dependencies
2. Review changes in `wit/deps/`
3. `wash build` — verify the build succeeds
4. If the error persists, check specific WIT versions in `wit/deps/` and pin compatible versions
