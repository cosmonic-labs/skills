# Contributing to Claude Skills Marketplace

Thank you for your interest in contributing!

## What is a Skill?

A skill is a markdown file (`SKILL.md`) that provides Claude Code with specialized knowledge and guidance for a specific domain. Skills help Claude give better, more accurate responses when working in that area.

## Skill Structure

Each skill lives in its own directory with a single `SKILL.md` file:

```
your-skill-name/
└── SKILL.md
```

### SKILL.md Format

Your `SKILL.md` must include YAML frontmatter followed by markdown content:

```yaml
---
name: your-skill-name
description: A concise description of what expertise this skill provides (one sentence).
license: Apache-2.0
tags:
  - tag1
  - tag2
  - tag3
---

# Your Skill Title

Content goes here...
```

### Required Frontmatter Fields

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Skill identifier (kebab-case, matches directory name) | `rust-development` |
| `description` | One-sentence description of the skill's expertise | `Expert in Rust best practices...` |
| `license` | License identifier (must be Apache-2.0 for this repo) | `Apache-2.0` |
| `tags` | List of 3-5 relevant tags for discoverability | `[rust, development, best-practices]` |

### Tag Guidelines

- Use lowercase, hyphenated tags
- Include 3-5 tags per skill
- Use existing tags when applicable:
  - Languages: `rust`, `python`, `go`, `javascript`
  - Domains: `webassembly`, `wasi`, `component-model`, `wasmcloud`
  - Types: `debugging`, `troubleshooting`, `best-practices`, `tooling`, `architecture`
- Create new tags only when necessary

## Content Guidelines

### What Makes a Good Skill?

1. **Focused scope** - Covers a specific domain or tool deeply
2. **Actionable guidance** - Provides concrete recommendations, not just concepts
3. **Code examples** - Includes practical code snippets where applicable
4. **Common issues** - Addresses frequently encountered problems and solutions
5. **Best practices** - Codifies expert knowledge and patterns

### Content Structure

A well-structured skill typically includes:

```markdown
# Skill Title

Brief overview of what this skill covers.

## When to Use

Describe scenarios where this skill applies.

## Core Concepts

Essential knowledge and terminology.

## Common Patterns

Recommended approaches with code examples.

## Common Issues and Solutions

Problems users frequently encounter.

## Best Practices

Key recommendations summarized.

## Resources (optional)

Links to official documentation or references.
```

### Writing Style

- Write in a direct, instructional tone
- Use code blocks for all code examples
- Prefer concrete examples over abstract explanations
- Include both "do" and "don't" examples where helpful
- Keep sections scannable with clear headings

## Review Criteria

Pull requests will be reviewed for:

1. **Relevance** - Does the skill fill a gap in the marketplace?
2. **Quality** - Is the content accurate and well-written?
3. **Structure** - Does it follow the required format?
4. **Scope** - Is it focused enough to be useful?
5. **Originality** - Does it provide value beyond existing skills?

## Updating Existing Skills

To improve an existing skill:

1. Fork and clone the repository
2. Make your changes to the existing `SKILL.md`
3. Submit a pull request with a clear description of the improvements

## License

By contributing, you agree that your contributions will be licensed under the Apache-2.0 license.

## Questions?

Open an issue if you have questions about contributing or need guidance on skill development.
