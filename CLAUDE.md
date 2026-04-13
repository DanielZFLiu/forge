# CLAUDE.md

## Immutable

### Commit Conventions

See `docs/commit-conventions.md`. Symbol prefix format:

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
- Scope in parens when useful: `+ (editor) block parser`
- Do not make small commits (e.g. 10 lines of change for a small fix); instead bundle edits into medium sized changes and then commit them.
- Multi-line messages for multiple changes. E.g.
```
+ (editor) undo/redo
! (editor) editor now editable when empty
```
- Verify behavior before committing
- No co-authored-by line

### Quality

Run `npm test` after every feature or refactor. Passing tests are the minimum bar.

### Testing

Tests must catch regressions. Ask: "if someone broke this, would this test catch it?" Follow `forge-tests` for coverage and organization.

### Debugging

Root-cause first. If the first fix fails, re-examine assumptions from scratch. Use the `systematic-debugging` skill.

### Process

Follow required review processes and skill workflows — don't skip steps when momentum is high.

### Style

Follow `forge-style` for code, `forge-docs` for docs.

### Review

After a major change, invoke `forge-review`.
