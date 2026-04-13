---
name: forge-review
description: Use when the user asks for a thorough code review, codebase audit, post-release quality check, or before starting a new development phase — not for single PR reviews or quick style checks
---

# Forge Review

Structured codebase review methodology. Four passes, verified findings, tiered fixes.

## Overview

**Core principle:** Review by concern type, not by file. Separate passes for bugs, design, testing, and organization prevent mixing severity levels and missing entire categories of issues.

## Phase 1: Orientation

Before reviewing anything, build a mental model:

1. **Read project docs** — README, changelog, roadmap, design docs, style guides. Understand what the system does, how it's organized, and what just shipped.
2. **Map the codebase** — file counts and sizes per module, identify the largest files, understand the dependency graph.
3. **Learn the style** — read the project's code style guide and commit conventions.

Skip what you already know from the current session — don't re-read docs you've just worked through.

## Phase 2: Four Review Passes

Run passes sequentially. Within each pass, use parallel `feature-dev:code-reviewer` agents (max 3 at a time). These are better than Explore agents for review — they filter by confidence and report only high-priority issues.

### Pass 1: Logical Bugs

Look for: race conditions, null/undefined mishandling, incorrect state transitions, missing error handling at boundaries, broken control flow, logic that contradicts design docs, execution path parity gaps (feature works in one code path but is entirely missing in another).

**Split by risk:**

- Batch A: highest-risk modules (largest, most complex, most interconnected)
- Batch B: lower-risk modules (smaller, more isolated)

### Pass 2: Design + Documentation

Look for: tier/boundary violations, leaky abstractions, business logic in the wrong layer, contract violations, SDK/library integration patterns that deviate from the library's documented approach, stale design docs, outdated version references, misleading comments. Evaluate doc quality against `forge-docs` criteria.

### Pass 3: Testing Quality

Look for: tests that verify implementation details instead of behavior, missing coverage for new features, tests that would pass even if the feature regressed, missing edge case and error path tests, test/production environment parity gaps, test helper duplication.

**Key question per test:** "If someone broke this feature, would this test catch it?" Use `forge-tests` criteria for what counts as a good test.

### Pass 4: Organizational Issues

Look for: files too long with genuine responsibility sprawl, missing section dividers, missing file header comments, premature abstractions, inconsistent file naming conventions, messy directory structures.

**Run this pass last** — decomposition proposals should account for bugs found in earlier passes. Don't flag files that are long but cohesive — length alone isn't a problem.

### Agent prompt template

Each agent gets a prompt with:

```
**Your scope:** [specific files with line counts]
**What to look for:** [pass-specific concerns from above]
**Context:** [what the system does, recent changes, known patterns]
**Instructions:** Read every line. Report findings with severity (critical/important/minor),
exact file:line, description, and code evidence. If a file is clean, say so explicitly.
```

### Verify findings

Agents produce false positives. Before recording a finding, read the actual code to confirm. When you have E2E evidence (from testing the feature live), use it — unit tests can pass while a feature is broken at the integration boundary.

## Phase 3: Record Findings

Write findings to a tracking document (e.g., `docs/code-review-findings.md`).

**Group by theme, not just by pass.** If five findings are all instances of the same structural gap (e.g., "pipeline path missing features the single-session path has"), group them under one heading. This makes the root cause visible and helps prioritize: fix the structural issue, not five symptoms.

Severity levels:

- **Critical** — correctness bugs, data loss risks
- **Important** — design violations, stale docs that mislead, tests that don't guard features
- **Minor** — style, naming, cosmetic

Commit the findings doc so nothing is lost. After all fixes are applied, keep the doc as a record — don't delete it.

## Phase 4: Tiered Fixes

**Do not fix everything at once.** Group by risk and fix tier by tier:

- **Tier 1: Docs + small code fixes** — stale docs, wrong field names, import ordering. No behavior change. Safe, fast.
- **Tier 2: Targeted bug fixes** — small code changes with clear boundaries. Build + test after.
- **Tier 3: Structural fixes** — larger changes that touch multiple files or add infrastructure. Build + test after each.
- **Tier 4: Major refactors** — file decompositions, architectural changes. Use isolated worktrees for the riskiest ones.

**After each tier:** `build`, `test`, `format`, then commit.

## Quick Reference

| Phase | What | Output |
|-------|------|--------|
| 1. Orientation | Read docs, map codebase, learn style | Mental model |
| 2. Four Passes | Bugs → Design → Testing → Organization | Verified findings |
| 3. Record | Theme-grouped findings doc with severity | Committed tracking doc |
| 4. Tiered Fixes | Tier 1-4, build+test after each | Clean commits per tier |

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Jumping into code without reading docs first | Phase 1 exists for a reason — orientation prevents false positives |
| Mixing bug/design/org concerns in one pass | One concern type per pass ensures nothing is systematically missed |
| Recording agent findings without verifying | Read the actual code — agents fabricate file paths and line numbers |
| Fixing everything at once | Group by risk tier, build+test after each tier |
| Flagging long files as problems | Length alone isn't an issue — only flag genuine responsibility sprawl |
| Skipping test quality review | A passing test suite means nothing if the tests don't guard behavior |

## Key Principles

- **Verify before recording.** Read the actual code. Check E2E behavior when available.
- **Consolidate by file before fixing.** Edit once per file, not once per finding.
- **Long-term solutions, not patches.** Fix the structural gap, not the symptom.
- **Don't fix what isn't broken.** A 400-line file with clear sections and one responsibility is fine.
- **Parallel where independent, sequential where dependent.**
- **Commit per tier, not per finding.**

**REQUIRED COMPANION:** Follow `forge-style` when making fixes.
