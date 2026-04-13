---
name: forge-tests
description: Use when writing, reviewing, or planning unit tests or E2E tests — covers test quality, coverage strategy, file organization, and real user interaction simulation. Addresses common agent pitfalls like happy-path-only coverage, trivial tests, monolithic files, and programmatic shortcuts that bypass real user behavior
---

# Forge Tests

Tests catch regressions. Every test should answer: "if someone broke this specific behavior, would this test fail?" Test code is still code — follow `forge-style` for naming, decomposition, and structure.

## Shared Principles

### Coverage That Matters

- **Edge cases over happy paths.** The happy path usually works. Test the boundaries: empty input, off-by-one, state transitions, concurrent operations, undo after a destructive action.
- **No trivial tests.** Don't test that a constructor sets a field, that `true === true`, or that a function called with valid input doesn't throw. If the test can't fail from a realistic code change, delete it.
- **No duplicate coverage.** If unit tests already prove round-trip serialization, E2E tests shouldn't re-test it. Each layer tests what it uniquely can.

### File Organization

- **One concern per file.** A test file covers one module, one feature, or one behavior area. Not "everything about headings" (710 lines) — split into "heading conversion", "heading split/merge", "heading navigation".
- **Target: under 150 lines per file.** If a file exceeds this, it's probably covering multiple concerns. Split it.
- **Parameterize repetitive cases.** If 6 tests differ only in a single input value (h1 through h6), use a loop or table-driven pattern. Two representative cases beat six copy-pasted ones.

### Before Writing Tests

```
Does a requirement file exist for this feature?
  ├─ Yes → follow it
  └─ No → is this E2E / integration for an interactive feature?
       ├─ Yes → write the requirement file first
       └─ No (unit test for pure logic) → requirements are implicit in the API contract
```

## Unit Tests

Unit tests verify logic in isolation. They're fast, deterministic, and focused.

- **Test the interesting boundaries**, not every permutation. A merge function needs tests for: eligible pairs, ineligible pairs, edge cases (first/last child, empty content, container with one child left after merge). It doesn't need a separate test for every possible kind pair.
- **Don't test what can't regress independently.** If `isBlockEditable` is a one-line lookup, one test suffices. Don't write 5 tests for 5 block types — the implementation is a set membership check.
- **Don't test defensive cases that can't happen.** If the editor never produces a node with `undefined` children, don't test how the function handles `undefined` children.
- **Match existing conventions.** Before writing, read 2-3 existing test files. Match: import style, describe/it nesting, helper patterns, assertion style, file naming.

## E2E Tests

E2E tests verify behavior through the same interface users use. The test must fail if the user's experience would break.

### Simulate Real User Actions

This is the most important rule. Agents default to programmatic shortcuts. Users don't.

| User action | Correct simulation | Wrong simulation |
|---|---|---|
| Type text | Keyboard events (per-character or string) | Programmatic API call that sets value directly |
| Click at position | Simulated click then keyboard navigation | Programmatic cursor/selection placement |
| Select text | Shift+arrow keys or click-drag | Programmatic range selection |
| Undo | Keyboard shortcut (Ctrl+Z / Cmd+Z) | Programmatic undo call |

**Why this matters:** Programmatic shortcuts bypass event handlers, focus management, cursor positioning, and rendering pipelines. A test using programmatic APIs can pass while the real user interaction fails.

**Exception:** Use programmatic calls for state queries (reading values, checking counts) — that's assertion, not interaction.

### Requirement-Driven Coverage

For interactive features, write a plain-English requirement file before writing tests. Derive scenarios from:
- Design docs and architecture specs
- Existing behavior described in changelogs
- How a real user would interact (keyboard, mouse, combined)
- What could go wrong (edge cases, rapid input, undo, state boundaries)

#### Requirement file organization

Requirement files follow the same rules as test files: **one concern per file, 1:1 mapping with test files.**

```
tests/
  requirements/
    feature-a.md       ← scenarios for feature-a test file
    feature-b.md       ← scenarios for feature-b test file
    ...
  feature-a.spec.ts   ← implements feature-a.md
  feature-b.spec.ts   ← implements feature-b.md
  ...
```

- **One requirement file per test file.** Never one monolithic file for all features.
- **Target: under 50 scenarios per file.** If a requirement file exceeds this, the feature is too broad — split both the requirements and the test file.
- If a requirement file gets too big, that signals the test file should also be split. They stay in lockstep.

#### Requirement file format

```
# Feature: [name]

## Happy paths
- [scenario]: [expected outcome]

## Edge cases
- [scenario]: [expected outcome]

## User interactions
- [physical action pattern]: [expected outcome]

## Error cases
- [scenario]: [expected outcome]
```

The test file implements each scenario. If a scenario is in the requirements, there's a test for it. If a test exists, it maps to a requirement.

#### After fixing a bug

When a bug is fixed and a regression test is added, update the corresponding requirement file with the new scenario. Requirements are living documents — they grow as bugs are found and fixed. A regression test without a requirement entry will be forgotten by the next person writing tests for that feature.

### What E2E Should NOT Test

- Things unit tests already cover (parser round-trip, metadata extraction)
- CSS styling details (font size, color) unless they indicate a functional state
- Internal state that isn't visible to the user

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| 700-line test file | Split by behavior area. Target <150 lines. |
| 6 identical tests for h1-h6 | Parameterize: `for (const level of [1, 3, 6])` |
| E2E uses programmatic API for all input | Use real keyboard/mouse simulation for interactions |
| Tests only the happy path | Add: empty input, boundary offsets, undo after action, rapid state changes |
| Round-trip tests in E2E that duplicate unit tests | Delete them. E2E tests user-facing behavior. |
| Tests for impossible states | Delete. Only test states the system can actually reach. |
| No existing test files read before writing | Read 2-3 existing tests first. Match conventions. |
| Jumped straight to writing tests | Write or locate requirements first for E2E/integration. |
| One giant requirements file for all features | Split: one requirement file per test file, in a `requirements/` directory. |
| Bug fix + regression test but requirements not updated | Add the regression scenario to the requirement file. Requirements are living documents. |
