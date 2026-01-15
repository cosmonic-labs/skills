# Claude Skills Marketplace

A collection of Claude Code skills for WebAssembly, wasmCloud, and Rust development.

## Available Skills

| Skill | Description | Tags |
|-------|-------------|------|
| [rust-development](./rust-development) | Expert Rust development guidance covering naming conventions, type safety, error handling, and best practices | `rust` `development` `best-practices` |
| [webassembly-component-development](./webassembly-component-development) | Comprehensive guide to WebAssembly components: WASI, composition patterns, language interop, runtime compatibility, and troubleshooting | `webassembly` `component-model` `wasi` |
| [wash](./wash) | wasmCloud Shell CLI tool for building and managing WebAssembly components | `wasmcloud` `cli` `tooling` |

## Installation

### Option 1: Install as Marketplace (Recommended)

Install all skills from this marketplace using the Claude Code CLI:

```bash
/plugin marketplace add cosmonic-labs/skills
```

This adds the entire marketplace and all its skills to your Claude Code environment.

### Option 2: Project-Level Installation

Add individual skills to your project's `.claude/settings.json`:

```json
{
  "skills": [
    "https://github.com/cosmonic-labs/skills/tree/main/rust-development",
    "https://github.com/cosmonic-labs/skills/tree/main/webassembly-component-development"
  ]
}
```

### Option 3: User-Level Installation

Add individual skills to `~/.claude/settings.json` to make them available across all projects:

```json
{
  "skills": [
    "https://github.com/cosmonic-labs/skills/tree/main/rust-development"
  ]
}
```

### Option 4: Local Clone

Clone this repository and reference skills locally:

```bash
git clone https://github.com/cosmonic-labs/skills.git ~/skills
```

Then add to your settings:

```json
{
  "skills": [
    "~/skills/rust-development",
    "~/skills/webassembly-component-development"
  ]
}
```

## Skill Categories

### Rust Development
- **rust-development** - Comprehensive Rust coding standards and best practices

### WebAssembly & Component Model
- **webassembly-component-development** - Complete guide covering:
  - WASI fundamentals (Preview1 vs Preview2)
  - Component composition patterns
  - Language interoperability
  - Runtime compatibility
  - Testing strategies
  - Debugging and troubleshooting

### wasmCloud
- **wash** - Master the wasmCloud CLI tool

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on submitting new skills.

## License

Apache-2.0
