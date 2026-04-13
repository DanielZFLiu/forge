# Forge

Language-agnostic code quality skills and conventions for AI-assisted software engineering.

## Contents

| File | Purpose |
|---|---|
| `CLAUDE.md` | Engineering principles template — commit conventions, quality bar, review posture. Drop into your project root, or merge with an existing `CLAUDE.md` / `GEMINI.md` / `AGENTS.md`. |
| `commit-conventions.md` | Symbol-prefixed commit message format, referenced by `CLAUDE.md`. |
| `code-style.md` | Concrete formatting and comment conventions, referenced by `CLAUDE.md`. Complements `forge-style`. |
| `skills/forge-style/` | Universal code style principles: naming, decomposition, file structure, comments, commits. |
| `skills/forge-tests/` | Test quality, coverage strategy, real user interaction simulation for unit and E2E tests. |
| `skills/forge-docs/` | Documentation principles: abstraction over implementation, brevity, stability, diagram-first. |
| `skills/forge-review/` | Four-pass codebase audit methodology — bugs, design, tests, organization. |

## Usage

**Skills.** Copy the `skills/forge-*` directories into `~/.claude/skills/` (Claude Code) or the equivalent skills directory for your harness. Each skill has a `SKILL.md` with YAML frontmatter describing when it activates.

**Project conventions.** Copy `CLAUDE.md`, `commit-conventions.md`, and `code-style.md` into your project root. Merge with any existing `CLAUDE.md` if needed. Adapt the tooling references in `code-style.md` to your formatter and language.
