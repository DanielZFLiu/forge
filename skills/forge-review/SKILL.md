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
2. **Inventory ALL artifacts, not just source** — file counts and sizes per module, largest files, dependency graph — AND the non-source surface: stylesheets, build/test configs, CI workflows, packaging manifests, scripts, dev harnesses. A pass structure scoped from the source tree alone silently excludes these; count them in the map so the scoping step must consciously assign or exclude each. (A map built by counting only `.ts`/`.svelte` files once reported a stylesheet directory as "0 files" — six Important findings were hiding there.)
3. **Mine git history for hotspots** — churn and bug-fix density per file are the cheapest risk signal available; weight review depth toward them and toward recent feature commits.
4. **Ingest the project's own known-issues ledger** (issues tracker, deferral log, TODO registry) FIRST: tell reviewers what not to re-report, and treat each entry as a verification target — confirm it still holds, sharpen its repro, or deliver a fix design.
5. **Record the baseline** — baseline commit, clean working tree, and a pre-review green run of the project's gate. Without the green run, "build+test after each fix tier" cannot distinguish your breakage from pre-existing failures or flakes.
6. **Learn the style** — read the project's code style guide and commit conventions.

Skip what you already know from the current session — don't re-read docs you've just worked through.

## Phase 2: Four Review Passes

Run passes sequentially. Within each pass, run review agents in parallel with STRICTLY DISJOINT file scopes (a dedicated review agent type if the harness has one; a general agent with edit rights when the review's mandate includes fixing trivia on the spot). Bound concurrency to what the machine tolerates, and never run agents that WRITE to the tree concurrently with an agent running repo-wide gates — shared-tree contention produces phantom failures that cost real investigation time.

**When multiple agents WRITE to one shared checkout concurrently**, disjoint file scopes are necessary but not sufficient — the git index and refs are still shared. The rules that keep it safe (each learned from a real collision): commit with an explicit pathspec (`git commit -- <your files>`), never bare `git commit` against a shared index a sibling may have staged into; never `--amend` (the tip may be a sibling's commit by the time the amend lands — recover with `reset --soft`, land a second commit, never rebase away a sibling's hash mid-flight); never `git stash`/`git checkout --` on shared state — prove red-first by neutering your own new code instead; verify `git show --name-only` after committing; and treat a red in a sibling's scope as presumed-phantom — re-run it in isolation before investigating.

### Scoped Reviews

The four-pass structure exists so nothing is systematically missed in a **full** review. It's not a checklist to walk every time. Match the passes to the ask:

| Ask | Passes to run |
|---|---|
| "Is this ready to ship?" / "Full audit" / "Before next phase" | All four |
| "Check for bugs in module X" | Pass 1, scoped to that module |
| "Are the docs stale?" / "Design doc review" | Pass 2 only |
| "Are the tests catching regressions?" | Pass 3 only |
| "How's the directory structure?" / "Organize before v1" | Pass 4 only |

Going wider than the ask produces findings the user didn't want, buries the findings they did want, and drains the review budget on low-value passes. Reconfirm the scope with the user if the ask is ambiguous — don't guess.

### Pass 1: Logical Bugs

Look for: race conditions, null/undefined mishandling, incorrect state transitions, missing error handling at boundaries, broken control flow, logic that contradicts design docs, stale derived state (a cache/memo/invalidation key — or dependency/subscription set — that omits an input its output depends on; the same logical value duplicated across a build-time snapshot and its live source so the two can drift; a value captured at construction time that should be resolved live), execution path parity gaps (feature works in one code path but is entirely missing in another — a rule enforced at N−1 of N sibling entry paths is the single most common shape; when you find one instance, enumerate ALL the siblings).

**Adversarial input is its own lens — assign it explicitly.** Functional-correctness review does not ask what hostile or pathological input does; someone must: malformed/boundary input at every parse/decode/offset seam, Unicode (surrogate pairs, combining marks, length-changing case folds) at any index arithmetic, quadratic/exponential blowup shapes in scanners, unbounded recursion, user-supplied regex, injection and unsafe-scheme surfaces. Prefer empirical probes (throwaway tests, timed adversarial strings) over reasoning from code. Even where security is low-stakes, the review must state "adversarial surface: reviewed / none / N/A because…" rather than leave it implicit. (One dedicated adversarial agent found a hand-typable byte-corruption Critical plus three Importants that four sibling bug-pass agents structurally could not have seen.)

**Key question for any cached or duplicated value:** does the cache/memo/invalidation key include every input the output depends on? Is anything baked in at build/mount time that should instead resolve live from a source that changes? When two code paths back the same feature, does one read a snapshot while its sibling reads live — so only the snapshot path can go stale?

**Split by risk:**

- Batch A: highest-risk modules (largest, most complex, most interconnected)
- Batch B: lower-risk modules (smaller, more isolated)

### Pass 2: Design + Documentation

Look for: tier/boundary violations, leaky abstractions, business logic in the wrong layer, single-source-of-truth violations (the same logical value duplicated across a build-time snapshot and its live source — the design root of the stale-derived-state correctness bug in Pass 1), contract violations, SDK/library integration patterns that deviate from the library's documented approach, stale design docs, outdated version references, misleading comments, comment signal-to-noise sinks (enumerations of types/unions that already live in the code, references to past/future versions, narration of readable code, multi-paragraph docstrings on internal functions — see `forge-style` § Comments antipatterns). Evaluate doc quality against `forge-docs` criteria.

When flagging a design smell, prefer its canonical name (Feature Envy, Data Clump, Shotgun Surgery, Flag Argument, Temporary Field, Status Variable, …) over an ad-hoc label — consistent vocabulary makes findings aggregable across reviewers. Catalog: [luzkan.github.io/smells](https://luzkan.github.io/smells/); some projects keep a curated local reference (e.g. `docs/research/code-smells.md`) — prefer that when present.

### Pass 3: Testing Quality

Look for: tests that verify implementation details instead of behavior, missing coverage for new features, tests that would pass even if the feature regressed, missing edge case and error path tests, test/production environment parity gaps, test helper duplication.

**Key question per test:** "If someone broke this feature, would this test catch it?" Use `forge-tests` criteria for what counts as a good test.

**The miss-analysis (mandatory when Pass 1 confirmed bugs):** for EACH confirmed bug, answer — what test should have existed, in which file, and WHY did the suite miss it — then generalize the patterns. Recurring answers in practice: pure cores over-tested while the entry/dispatch layers feeding them have zero tests; property-generator inputs tame by construction (all-ASCII, no cross-construct interleaving); sibling-path parity never asserted as a class; oracles checking structure but never derived-state validity. The generalized patterns, not the individual gaps, are the finding.

### Pass 4: Organizational Issues

Look for: files too long with genuine responsibility sprawl, missing section dividers, missing file header comments, premature abstractions, inconsistent file naming conventions, messy directory structures.

**REQUIRED for the directory audit:** Load `forge-style` and apply Section 7's four diagnostic questions explicitly — *what changes together lives together*, *who depends on whom*, *could a new reader form this tree from first principles*, *name the concept not the shelf*. Cite the specific diagnostic each finding violates. Do not audit directories from general intuition or paraphrase forge-style rules from memory — read the section and quote it.

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

Agents produce false positives. Reading the code is the floor, not the bar:

- **Critical/Important findings: reproduce or revert-probe.** Reproduce empirically (a throwaway failing test, a probe script), or verify the claim survives a neuter-probe (disable the allegedly-buggy guard → the documented failure must appear). A finding that cannot produce a concrete failure scenario is not Critical/Important — severity labels require one.
- **Never accept a coverage claim without a revert-check.** "Pinned by existing tests" is disproven by reverting the change and watching every suite stay green — this exact claim has failed review under exactly this check. The same rule binds fix reports: a fix's regression test must be shown red against the pre-fix code.
- **Generate dynamic evidence; don't just wait for it.** Run the software, drive the feature, use the project's debug/probe tooling. Unit suites pass while integration boundaries are broken.
- Findings that survive verification get recorded; the ones that don't get recorded as falsified WITH the disproof, so the next reviewer doesn't re-derive them.

### Coverage reconciliation (before recording — the false-negative check)

Verification above controls false positives; nothing so far controls false NEGATIVES. Before Phase 3, reconcile: walk the Phase-1 artifact inventory and mark, per artifact class, which pass covered it — and justify every exclusion in one line. An artifact class no pass read (stylesheets, CI config, dev harness, generated data) is a scoping hole to close now, not after the report ships. Reviews rarely fail by wrong findings; they fail silently by omission.

## Phase 3: Record Findings

Write findings to a tracking document (e.g., `docs/code-review-findings.md`).

**Group by theme, not just by pass.** If five findings are all instances of the same structural gap (e.g., "pipeline path missing features the single-session path has"), group them under one heading. This makes the root cause visible and helps prioritize: fix the structural issue, not five symptoms.

Severity levels (Critical/Important require the concrete failure scenario from verification — inputs/state → wrong outcome):

- **Critical** — correctness bugs, data loss risks
- **Important** — design violations, stale docs that mislead, tests that don't guard features
- **Minor** — style, naming, cosmetic

**Route every finding to exactly one destination** — a fix in this review's tiers, a named future task's brief, or a tracker/issues entry — and record which. A findings doc without routing is a silent discard; "still open" is only acceptable with the routing attached.

Commit the findings doc so nothing is lost (or keep it wherever the project keeps review records). After all fixes are applied, keep the doc as a record — don't delete it.

## Phase 4: Tiered Fixes

**Do not fix everything at once.** Group by risk and fix tier by tier:

- **Tier 1: Docs + small code fixes** — stale docs, wrong field names, import ordering. No behavior change. Safe, fast.
- **Tier 2: Targeted bug fixes** — small code changes with clear boundaries. Build + test after.
- **Tier 3: Structural fixes** — larger changes that touch multiple files or add infrastructure. Build + test after each.
- **Tier 4: Major refactors** — file decompositions, architectural changes. Use isolated worktrees for the riskiest ones.

**After each tier:** `build`, `test`, `format`, then commit. Never pipe the gate command's output (`npm test | tail`) — the pipe's exit code masks the gate's; capture to a file and check the exit explicitly. Derive each fix's gate list from the FILES it touched, not from subsystem intuition — a batch that edits module X must run X's suite even when the batch is "about" module Y (a fix batch once shipped two silently-red tests because its gate list came from its theme, not its diff).

**Every bug fix lands test-first** — the regression test shown failing on the pre-fix code (quote the failure), then the fix, then green. A fix without a red-first pin is a claim, not a fix.

**Frame fix-dispatch diagnoses as hypotheses to verify, not conclusions to implement.** The dispatch says "hypothesis — verify before fixing," names the acceptance signal (usually the failing test going green unmodified), and licenses the fixer to falsify. A controller's confident root-cause diagnosis has been empirically wrong while the code was right — a verify-first fixer proved the failing assertion encoded stale semantics instead, and "fixed" nothing.

**Fixes get re-reviewed.** Critical/Important fix waves go back through the reviewer (or the controller re-verifies with the same reproduce/revert standard). Build+test after a tier proves nothing about whether the fix addressed the finding.

**Dispatch subagents for mechanical fixes.** Once a fix is specified down to exact paths — a stale-doc sweep, an import-path rewrite across 10+ files, a file rename with N importers — the work is mechanical. Hand it to a subagent with the precise list; review the diff; commit. Don't grind through Read+Edit calls yourself when the scope is nailed down. Reserve your own attention for the judgment tiers (bug fixes, structural decisions, decomposition shape).

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
| Mapping only the source tree | Inventory every artifact class (CSS, configs, CI, scripts, harnesses) — unmapped classes become unreviewed classes |
| Mixing bug/design/org concerns in one pass | One concern type per pass ensures nothing is systematically missed |
| No adversarial-input lens anywhere | Assign it explicitly; state the adversarial verdict even when it's "N/A" |
| Recording agent findings without verifying | Read the code, then reproduce or revert-probe anything Critical/Important |
| Accepting "covered by existing tests" | Revert the change; if the suites stay green, the claim was false |
| Skipping the coverage reconciliation | False negatives are silent — justify every exclusion against the inventory |
| Confirmed bugs not fed back into Pass 3 | Each one gets a miss-analysis: which test should have caught it, and why didn't it |
| Fix gate lists chosen by theme, not by diff | Run the suites of the files actually touched |
| Piping the gate command | The pipe's exit code masks the gate's — capture and check explicitly |
| Fix dispatches stating diagnoses as fact | Frame as hypothesis-to-verify; the fixer may falsify it |
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
