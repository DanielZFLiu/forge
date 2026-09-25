---
name: forge-tests
description: Use when writing, reviewing, or planning unit, property, or E2E tests, or adding a regression test for a bug fix
---

# Forge Tests

How to write tests that catch regressions. Test code is code, so `forge-style` applies to it too.

## A test counts when it can fail

Every test answers one question: if someone broke this behavior, would this test go red? The only proof is watching it happen. A regression test is shown failing against the unfixed code, for the reason the bug gives, before the fix lands; a new test for existing behavior is checked by breaking the behavior (revert the change, neuter the guard) and watching it fail. A test that stays green through that was never guarding anything. The same check settles "this is already covered": revert the change, and if every suite stays green, it wasn't.

## What to cover

- **Boundaries over happy paths.** The happy path usually works. Test empty input, off-by-one offsets, first and last child, state transitions, undo after a destructive action, rapid repeated input.
- **The interesting boundaries, not every permutation.** Skip tests that can't fail from a realistic code change (a constructor setting a field, a one-line lookup tested once per key). Five tests of a set-membership check are one test.
- **Test at the layer where the bug would live.** The common blind spot is a pure core tested thoroughly with tidy, hand-built inputs while the layer that produces those inputs from real events (the key handler, the dispatcher, the paste route) has no tests at all. Each layer gets tests for what only it can get wrong, and a behavior proven at one layer isn't re-proven at another.
- **Oracles check the property that matters.** An assertion that the output has the right structure can pass while derived state (a cache, an index, a round-trip) is wrong. Assert the thing a user would notice.
- **An unreachable state gets an assertion, not a test.** If the system can't produce a node with no children, a test of how a function handles one guards nothing; a dev-mode check that it never happens does. "Can't happen" is a belief, though, and a generator is how you test the belief (below).

## Property tests

Property tests find the bugs nobody thought to write an example for, and they're only as good as their generator. A generator that draws ASCII text and one construct at a time can't produce the bugs that live in surrogate pairs, combining marks, or two constructs interleaved, so the suite says nothing about them. Make generators draw the shapes real input has, boundary shapes included. When a property finds a failure, pin the shrunk counterexample as a named example test so the case survives a generator change.

## Parameterized and generated cases

When cases differ only in one input (h1 through h6), use a table or a loop; two representative cases often beat six copies. Give each generated case its own name that includes its row, so a failure says which one broke. If the project maps requirements to tests by counting, the count has to come from what the runner lists (for Playwright, `playwright test --list`), not from counting `test(` calls in the source, or every loop-driven file reads as one test and fails the mapping for no real reason.

A bug found on one of several sibling routes gets a regression test that runs every sibling route, parameterized, so the next route added without the rule fails too.

## Timing

A test that waits a fixed time for something to happen passes on a quiet machine and flakes under load. Wait for the condition itself (poll for it, or use the framework's auto-waiting assertions). A test that's green alone and red under a full parallel run is showing you a real race or an ordering dependency; record it and find the cause instead of retrying it green.

## Organizing tests

- **One concern per file.** A file covers one module, feature or behavior area. Past about 150 lines, check whether it's covering two concerns; length alone is fine if it isn't.
- **Match the house style.** Read two or three existing test files first and match imports, nesting, helpers, assertions and file naming.
- **Tests follow their source.** When a module moves, either mirror the new source path or keep test directories flat and named by concept; pick one per repo, and rename the test directory in the same commit that moves the source.

## E2E tests

E2E tests exercise the product the way a user does, and must fail when the user's experience would break.

### Simulate real user actions

Programmatic shortcuts skip the event handlers, focus management, caret placement and rendering the user actually goes through, so a test built on them can pass while the real interaction fails.

| User action | Simulate with | Not with |
|---|---|---|
| Type text | Keyboard events | An API call that sets the value |
| Place the caret | A click, then keyboard navigation | Programmatic selection |
| Select text | Shift+arrows or click-drag | A programmatic range |
| Undo | Ctrl+Z / Cmd+Z | A direct undo call |

Programmatic calls are fine for reading state to assert on; that's observation, not interaction.

A real gesture still has to land deterministically. A click at the center of a measured rect can fall on either side of a glyph by font-metric luck, so an exact assertion after it flakes by construction. Click into the neighborhood, then walk to the exact target with the keyboard, or derive the expectation from where the click actually landed.

Leave to other layers what they already prove (parser round-trips), styling details that don't signal a functional state, and internal state the user can't see.

### Requirement files

For an interactive feature, write the requirements in plain English before the tests. Draw scenarios from design docs, the changelog, how a real user would reach the feature (keyboard, mouse, both), and what could go wrong. Pure logic doesn't need one; its API contract is the requirement.

Requirement files pair one-to-one with spec files (`requirements/feature-a.md` beside `feature-a.spec.ts`): every scenario has a test and every test maps to a scenario. A requirement file past about 50 scenarios usually means the feature should split, and the spec splits with it.

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

### After fixing a bug

Add the regression scenario to the requirement file, so the next person writing tests for the feature sees it. Record a one-line miss-analysis with it (in the requirement file for E2E, as the test's header line for a unit test): what test should have caught this, and why none did. The general answers (an entry path with no tests, a generator too tame to draw the shape, a sibling route never tested as a class) are how a suite's blind spots get named instead of rediscovered.
