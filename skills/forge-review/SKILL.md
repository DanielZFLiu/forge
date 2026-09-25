---
name: forge-review
description: Use when the user asks for a thorough code review, codebase audit, post-release quality check, or a review before a new development phase; its standard for verifying findings and fixes also applies when reviewing any bug fix
---

# Forge Review

A method for reviewing a codebase so the review finds what's there, proves it, and misses as little as possible: four passes by concern, every serious finding reproduced, every fix shown red first.

Reviewing by concern rather than by file keeps severities from blurring together and keeps whole categories of problem from going unasked. The fixes follow `forge-style`, and what counts as a good test is in `forge-tests`.

## Phase 1: Orientation

1. **Read the project's docs**: README, changelog, design docs, style guide, commit conventions. Know what the system does and what just shipped. Skip what you already read this session.
2. **Inventory every artifact, not just source**: sizes per module, the largest files, the dependency graph, and the non-source surface (stylesheets, build and test configs, CI, packaging, scripts, dev harnesses). A pass structure scoped from the source tree alone silently skips the rest: a map that counts only the main language's files reports a stylesheet or config directory as empty, and whatever is wrong in it goes unread.
3. **Mine git history for hotspots.** Churn and bug-fix density per file are the cheapest risk signal there is; weight depth toward them and toward recent feature work.
4. **Read the known-issues ledger first.** Tell reviewers what not to re-report, and treat each open entry as something to verify: confirm it still holds, sharpen its repro, or design its fix.
5. **Record a baseline**: the commit, a clean tree, and a green run of the project's gate. Without it, a red after a fix can't be told apart from a pre-existing failure or a flake.

## Phase 2: Four passes

Run the passes in order. Within a pass, run review agents in parallel on disjoint file scopes, as many as the machine tolerates, and never run an agent that writes to the tree beside one running repo-wide gates; contention on a shared tree produces failures that aren't real and cost real time to chase.

Match the passes to the ask. Going wider buries the findings the user wanted under ones they didn't.

| Ask | Passes |
|---|---|
| "Ready to ship?", full audit, before the next phase | All four |
| "Bugs in module X" | Pass 1, scoped to X |
| "Are the docs stale?" | Pass 2 |
| "Are the tests catching regressions?" | Pass 3 |
| "How's the directory structure?" | Pass 4 |

If the ask is ambiguous, confirm the scope with the user.

### Pass 1: Logical bugs

Race conditions, null handling, wrong state transitions, missing error handling at boundaries, broken control flow, logic that contradicts the design docs. Two shapes deserve a deliberate look because they recur:

- **Stale derived state.** For every cache, memo, invalidation key or subscription set, does the key include every input the output depends on? Is anything captured at construction that should be read live? When two paths back one feature, does one read a snapshot while the other reads the live source?
- **Sibling-path parity.** A rule enforced at N-1 of N entry paths is the most common serious bug. When you find one instance, enumerate every sibling path before reporting.

Assign the adversarial lens to someone explicitly, because functional review doesn't ask what hostile or pathological input does: malformed and boundary input at every parse, decode and offset boundary; Unicode (surrogate pairs, combining marks, case folds that change length) wherever there's index arithmetic; quadratic or exponential shapes in scanners; unbounded recursion; user-supplied regex; injection and unsafe URL schemes. Prefer empirical probes (a throwaway test, a timed adversarial string) to reasoning. The review states its verdict ("adversarial surface: reviewed / none / N/A because ...") even when security stakes are low, since a reviewer reading for correct behavior on normal input can't see the failures that only hostile input produces.

Split the modules by risk: the largest and most interconnected in one batch, the smaller and isolated in another.

### Pass 2: Design and documentation

Boundary violations, leaky abstractions, logic in the wrong layer, contract violations, library integrations that depart from the library's documented approach, and single-source-of-truth violations (the design root of Pass 1's stale derived state). A rule copied at several routes is a design finding even when every copy is currently correct: it belongs at one shared entry point (`forge-style` § Where a rule lives), and the copies are where the next parity bug will come from.

Docs: stale design docs, misleading comments, and comments that fail `forge-style` § Comments. Judge docs against `forge-docs`.

Name design smells by their canonical names (Feature Envy, Shotgun Surgery, Flag Argument, Data Clump, ...) so findings from different reviewers aggregate. Prefer the project's own smell reference if it keeps one.

### Pass 3: Test quality

Tests that check implementation instead of behavior, features with no coverage, tests that would stay green if the feature broke, missing edge and error paths, test and production environments that differ, duplicated test helpers. `forge-tests` has the criteria.

**Miss-analysis.** For each bug Pass 1 confirmed: what test should have existed, in which file, and why did the suite miss it? Then generalize. The recurring answers are that entry and dispatch layers have no tests while the pure cores behind them are over-tested, that property generators are tame by construction, that a sibling-path rule is never asserted as a class, and that oracles check structure but not derived state. The generalized patterns are the finding, more than the individual gaps.

### Pass 4: Organization

Files with genuine responsibility sprawl (length alone isn't a problem; a 400-line file with one responsibility and clear sections is fine), missing section dividers or headers where a file needs one, premature abstractions, inconsistent naming, messy directories. Audit directories with `forge-style` § Directory structure's four questions and name which question each finding fails.

Run this pass last, so decomposition proposals account for the bugs the earlier passes found.

### Agent prompt

```
Your scope: [files, with line counts]
What to look for: [this pass's concerns]
Context: [what the system does, recent changes, known patterns, known issues not to re-report]
Instructions: Read every line. Report each finding with severity (critical/important/minor),
file:line, description, and code evidence. If a file is clean, say so.
```

### Verify findings

Agents produce false positives, and reading the code is the floor, not the bar.

- **A Critical or Important finding is reproduced**, with a throwaway failing test or a probe script, or confirmed by a neuter-probe (disable the allegedly broken guard and the claimed failure must appear). A finding that can't produce a concrete failure scenario isn't Critical or Important.
- **A coverage claim is revert-checked.** "Pinned by existing tests" is disproven by reverting the change and watching every suite stay green, and it's a claim that fails that check more often than it sounds.
- **Generate dynamic evidence.** Run the software, drive the feature, use the project's debug tooling. Unit suites pass while integration boundaries are broken.
- Record findings that fail verification as falsified, with the disproof, so the next reviewer doesn't re-derive them.

### Coverage reconciliation

Verification controls false positives; nothing so far controls false negatives, and reviews mostly fail by omission. Before recording, walk the Phase 1 inventory and mark which pass covered each artifact class, with a one-line reason for every exclusion. A class no pass read is a scoping hole to close now.

## Phase 3: Record

Group findings by theme, not only by pass: five findings that are one structural gap go under one heading, so the fix targets the gap instead of five symptoms.

- **Critical**: correctness bugs, data loss.
- **Important**: design violations, docs that mislead, tests that don't guard their feature.
- **Minor**: style, naming, cosmetics.

Critical and Important carry their concrete failure scenario (this input and state, this wrong outcome).

Route every finding to exactly one place (a fix tier in this review, a named future task, or the project's issue tracker) and record where. An unrouted finding is a silent discard. The findings document is a working artifact; commit it only if the project keeps review records, since the tracker is where open findings live.

## Phase 4: Tiered fixes

Fix by risk tier, not all at once:

1. Docs and small code fixes with no behavior change.
2. Targeted bug fixes with clear boundaries.
3. Structural fixes across several files.
4. Major refactors and decompositions, the riskiest in an isolated worktree.

Consolidate edits by file within a tier, and commit per tier. After each tier run build, tests and format, then commit. A few habits keep the gate honest:

- **Never pipe the gate** (`npm test | tail`): the pipe's exit code replaces the gate's. Capture output to a file and check the exit yourself.
- **Derive each fix's gate list from the files it touched**, not from its theme. A batch "about" module Y that edits module X runs X's suites, or X's red tests ship unseen.
- **Every bug fix lands test-first**: the regression test shown red on the pre-fix code, the failure quoted, then the fix, then green.
- **Fix the class.** Find the rule the bug broke and fix every route it applies to, not the instance you saw.

Dispatching fixes to other agents:

- **State a diagnosis as a hypothesis**, name the acceptance signal (usually the failing test going green unmodified), and let the fixer falsify it. A confident root cause from the reviewer can still be wrong, and a fixer told to verify first is the one who catches it.
- **Hand mechanical work down** once it's specified to exact paths (a stale-doc sweep, an import rewrite across many files, a rename with many importers); review the diff and commit. Keep your own attention for bug fixes, structural decisions and decomposition shape.
- **Re-review fixes.** Critical and Important fix waves go back through a reviewer, or the controller re-verifies them to the same reproduce and revert standard. A green build proves nothing about whether the fix addressed the finding.
