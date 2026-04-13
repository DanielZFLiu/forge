# CLAUDE.md

Reusable engineering principles for AI-assisted development. Drop into your project's root, or merge with an existing `CLAUDE.md` / `GEMINI.md` / `AGENTS.md`.

## Commit Conventions

See `commit-conventions.md`. Symbol prefix format:

| Symbol | Meaning |
| ------ | ------- |
| `+` | New feature |
| `-` | Removal |
| `~` | Small tweak |
| `>` | Normal to large change |
| `!` | Bug fix |
| `@` | Docs/config |

Rules:
- Lowercase, no period, short
- Scope in parens when useful: `+ (auth) session tokens`
- Bundle related small edits into a medium-sized commit — don't fragment a cohesive change across tiny commits
- Multi-line messages for multiple distinct changes
- Verify behavior before committing
- No co-authored-by line

## Quality

Run the full test suite after every feature or refactor. Passing tests are the minimum bar.

## Testing

Tests must catch regressions. Ask: "if someone broke this, would this test catch it?" Follow `forge-tests` for coverage and organization.

## Debugging

Root-cause first. If the first fix fails, re-examine assumptions from scratch.

## Process

Follow required review processes and skill workflows — don't skip steps when momentum is high.

## Style

Follow `forge-style` for code principles and `code-style.md` for concrete formatting rules. Follow `forge-docs` for documentation.

## Review

After a major change, invoke `forge-review`.
