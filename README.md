<p align="center">
  <a href="https://cosmonic.com">
    <img src="https://cosmonic.com/images/cosmonic-logo-purple.svg" alt="Cosmonic" width="300" />
  </a>
</p>

<h1 align="center">Cosmonic Skills Marketplace</h1>

<p align="center">
  <strong>Claude Code skills for WebAssembly, wasmCloud, and cloud-native development</strong>
</p>

<p align="center">
  <a href="https://github.com/cosmonic-labs/skills/blob/main/LICENSE"><img src="https://img.shields.io/badge/license-Apache--2.0-685BC7?style=flat-square" alt="License" /></a>
  <a href="https://cosmonic.com"><img src="https://img.shields.io/badge/Built%20by-Cosmonic-685BC7?style=flat-square" alt="Built by Cosmonic" /></a>
  <a href="https://wasmcloud.com"><img src="https://img.shields.io/badge/Powered%20by-wasmCloud-00C389?style=flat-square" alt="Powered by wasmCloud" /></a>
</p>

---

## Available Skills

| Skill | Description | Tags |
|-------|-------------|------|
| [wash](./wash) | wasmCloud Shell CLI tool for building and managing WebAssembly components | `wasmcloud` `cli` `tooling` |
| [webassembly-component-development](./webassembly-component-development) | Comprehensive guide to WebAssembly components: WASI, composition patterns, language interop, runtime compatibility, and troubleshooting | `webassembly` `component-model` `wasi` |
| [rust-development](./rust-development) | Expert Rust development guidance covering naming conventions, type safety, error handling, and best practices | `rust` `development` `best-practices` |
| [brand-guidelines](./brand-guidelines) | Official Cosmonic and wasmCloud brand colors, typography, and visual identity standards | `branding` `design` `cosmonic` `wasmcloud` |

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

### wasmCloud Development
- **[wash](./wash)** - Master the wasmCloud Shell CLI tool for building, running, and managing WebAssembly components and wasmCloud applications

### WebAssembly & Component Model
- **[webassembly-component-development](./webassembly-component-development)** - Complete guide covering:
  - WASI fundamentals (Preview1 vs Preview2)
  - Component composition patterns
  - Language interoperability
  - Runtime compatibility
  - Testing strategies
  - Debugging and troubleshooting

### Rust Development
- **[rust-development](./rust-development)** - Comprehensive Rust coding standards and best practices

### Brand & Design
- **[brand-guidelines](./brand-guidelines)** - Official Cosmonic and wasmCloud visual identity standards including colors, typography, and usage guidelines

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines on submitting new skills.

## License

Apache-2.0

---

<p align="center">
  <sub>Built with 💜 by <a href="https://cosmonic.com">Cosmonic</a> — Sandboxing agentic AI with WebAssembly</sub>
</p>
