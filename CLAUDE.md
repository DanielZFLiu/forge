# CLAUDE.md

## Transient
Put codebase specific information here.

## Immutable

The forge-\* skills (review, style, docs, tests) are the canonical guidelines for this codebase, you MUST read through ALL of them at the start of the session, even if the user didn't ask you to. Apply them whenever they're relevant — in every code, docs, test, and review action.

### Commit Conventions

See `docs/commit-conventions.md`. Symbol prefix format:

| Symbol | Meaning                |
| ------ | ---------------------- |
| `+`    | New feature            |
| `-`    | Removal                |
| `~`    | Small tweak            |
| `>`    | Normal to large change |
| `!`    | Bug fix                |
| `@`    | Docs/config            |

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

### Comments

Default to none. Explain _why_ (non-obvious choices), never _what_. The test: if removing the comment wouldn't confuse a reader, delete it. When you touch a file, you own its comment signal-to-noise — prune comments that fail the test even if you didn't write them; don't match existing over-commented style. Antipatterns (enumerating types, referencing past/future versions, narrating the next line, multi-paragraph docstrings on internal functions) live in `forge-style` § Comments.

### Subagents

Any dispatched subagent (dispatched via the `Agent` tool) must read `forge-style`, `forge-docs`, and `forge-tests` at the start of its task and enforce them in every file it touches — for example, improving pre-existing verbose comments when they fall in its path. Subagent briefs must name these three skills explicitly.

Unless the task is trivial, any dispatched subagent must be opus 4.7.

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
