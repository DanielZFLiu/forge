---
name: forge-style
description: Use when writing, modifying, or reviewing code — especially when adding to files with poor naming, missing structure, or messy patterns that tempt you to match existing style
---

# Forge Style

Universal, language-agnostic code style. These principles apply to every codebase regardless of language or framework. **Sibling skills:** `forge-docs` for docs, `forge-tests` for tests, `forge-review` for structured audits.

## The Cardinal Rule

**When you touch messy code, improve what you touch.** Don't conform to bad patterns. If the surrounding code has generic names, mode-parameter functions, or no structure — fix what's in your path. You are not obligated to match existing bad style.

This means: if you're adding `deleteUser` to a file that has `doStuff(action)` and `h(email)`, you rename `doStuff` to `createUser`/`findUser`, rename `h` to `isValidEmail`, and add section dividers — not just append your function to the bottom.

**Don't rationalize inaction:**

- "The user only asked me to add, not refactor" — improving what you touch IS part of adding. A surgeon doesn't leave old gauze in the patient.
- "I might break something by renaming" — renaming local/internal functions is safe. For exported/public APIs, check callers first — but still rename if you can update them.
- "It's not my code to change" — you were given the file to modify. Improve it.

## 1. Simplicity

Don't build for hypotheticals. Abstraction is a cost — pay it only when repetition forces your hand.

- No abstraction until the third repetition
- Prefer flat control flow; prefer shallow inheritance/wrapper chains
- Delete dead code — git remembers
- Delete migration shims once the migration is done. A re-export facade whose own docstring says "stable import path for legacy callers" has an expiration trigger: when all callers moved, kill the shim. Same for rename aliases (`export { foo as fooLegacy }`) and compatibility wrappers. "Keeps old callers working" justification outlives its usefulness fast — treat it as a deadline, not a promise.
- If a function needs a paragraph to explain what it does, it's doing too much

```
// bad: premature generalization for one use case
function transformData(data, format, options, callback)

// good: do the specific thing
function csvToJson(data)
```

## 2. Naming

Names are the first layer of documentation. Describe what something is or does, not how it works.

- Name by role/purpose, not implementation (`userRecords` not `hashMap`)
- Booleans read as questions (`isVisible`, `hasChildren`, `canEdit`)
- Functions describe their action or return value (`fetchUser`, `parseConfig`)
- Avoid generic names (`data`, `item`, `temp`) unless scope is under ~5 lines
- Stay consistent — if it's `user` in one place, don't call it `account` elsewhere

```
// bad
function process(d)
  temp = d.val * 1.1
  return temp

// good
function applyTax(price)
  taxedPrice = price * 1.1
  return taxedPrice
```

## 3. Decomposition

Each unit (function, file, module) has one clear responsibility you can state in a short sentence. If you struggle to name it, it's doing too much.

- If you'd use "and" to describe a function, split it
- Prefer composable functions over long functions with flags/mode parameters
- Files group related things — not "all helpers" or "all utils"
- Don't split too early — three similar inline lines beat a premature helper called once

```
// bad: two functions taped together
function handleUser(user, mode)
  if mode == "create" ...
  if mode == "update" ...

// good
function createUser(user)
function updateUser(user)
```

### Extract pure logic out of framework components

Framework components (React, Svelte, Vue SFCs, etc.) tend to accumulate pure logic alongside render/lifecycle code. When a component contains a large block — action bundles, state machines, computed tree operations — that only reaches into framework state through props/getters, extract it as a `createX(deps): X` factory in a plain file.

Triggers:

- A component has a 200+ line object literal or logic block that doesn't use framework lifecycle (no hooks, no reactive runes, no render calls, no refs to DOM).
- The logic's only tie to the component is reading props/state and writing back.
- You want the logic unit-testable without mounting the component.

Pattern:

```
// Before: 300 lines inline in ListComponent
const listContext = {
  indentItem(i) { ...reads node, state, parent action bundles... },
  insertItemAfter(i, item) { ... },
  // ...
}

// After: factory in a plain file
export function createListContext(deps: ListContextDeps): ListContext { ... }

// Component shrinks to:
const listContext = createListContext({
  get node()  { return node; },   // getters read live reactive values
  get index() { return index; },
  state,
  parentActions,
});
```

Rules:

- **Pass reactive state via getters**, not plain values. Closures that capture `node` by value snapshot the state at factory-call time; getters re-read each invocation.
- **Deps is a contract.** Keep it narrow. If the factory needs "basically everything the component has," the logic isn't independent — leave it where it is.
- **Don't extract what uses framework lifecycle.** `$effect`, `useEffect`, `onMount`, `watch` — those stay in the component.

## 4. File Structure

A file's organization lets a reader scan its shape and find what they need without reading every line.

- Colocate related items — types near the code that uses them, not in a separate `types` file unless shared
- Section dividers for logical groupings (adapt comment syntax per language):
  `// ── Section Name ────────────────────────────`
- Public API / main exports near the top; internals below
- File headers only when the filename isn't self-explanatory

```
// ── Public API ──────────────────────────

function createEditor(config)
function destroyEditor(editor)

// ── Internal ────────────────────────────

function initBuffer(config)
function attachListeners(editor)
```

## 5. Comments

Default to no comments. Explain _why_ — non-obvious reasoning, workarounds, deliberate exclusions — never _what_. Code, types, and names carry the _what_. Commit messages and `git blame` carry _when_ and _who_. Issue trackers carry _what's next_. Design docs carry _how things fit together_. A comment earns its line only by answering something none of those can: _why did the author make this specific local choice that wouldn't be obvious from reading the code?_

**The test:** if removing the comment wouldn't confuse a reader, delete it.

**You own the signal-to-noise on your way out.** When you touch a file, prune comments that fail the test — even ones you didn't write. Matching existing over-commented style perpetuates the rot; don't. Rationalizations that look reasonable but mean "I skipped pruning":

| Excuse | Reality |
|---|---|
| "Not my comment, not my job" | You're editing the file. Its signal-to-noise is now yours. |
| "Someone might find it helpful" | If removal wouldn't confuse a reader, it's not helping. |
| "It preserves context" | Context belongs in commits/PRs/issues/design docs — not frozen in the code. |
| "Might be load-bearing" | Only if you *don't understand its purpose*. Redundant-with-the-code goes; puzzling-to-you stays. |

### Antipatterns (delete on sight)

| Antipattern | Why it rots | Fix |
|---|---|---|
| Enumerating union members, enum values, or flag lists that live in the code | The moment someone adds a variant the comment lies; readers cross-check the type anyway | Delete; reference the type by name if useful |
| Referencing past or future versions ("post-0.5.4", "as of 2024", "TODO before v2") | Ephemeral context freezes into the file; once the version ships, the line is pure noise | Delete, or — if actionable — explicit `TODO(owner): reason` / issue tracker entry |
| Narrating the next line ("// now set the flag", "// loop through users") | Zero information beyond the code itself | Delete |
| Restating the function or variable name in prose | Duplicates the name, ages on rename | Delete |
| Mentioning callers or the current task ("used by the X flow", "added for ticket #123") | Belongs in PR description / commit message; rots as callers evolve | Delete; if the caller relationship is a genuine invariant, encode it as a type or assertion |
| Multi-paragraph docstrings on internal functions | Signal-to-noise sink; the name + signature should carry the load | Collapse to one sentence or delete. Long docstrings belong on public APIs with genuinely non-obvious edges |
| TODOs buried inside descriptive prose ("...the cast can be tightened post-X.Y") | Invisible to grep, undated, unowned, visually indistinguishable from description | Explicit `TODO(owner): reason — link`, or move to an issue tracker |

### Example

```
// ❌ bad — enumerates the union, narrates the cast, references a shipped version
// The public interface uses `string` for op.kind; narrow to the
// internal OperationKind union here. Callers pass known kinds
// ('split' | 'merge' | 'delete' | 'updateContent' | 'paste' |
// 'replaceBlock'); the cast can be tightened post-0.5.4.
const kind = op.kind as OperationKind;

// ✅ good — explains the one non-obvious why, nothing else
// Public interface widens to string for ergonomics; OperationKind is the internal source of truth.
const kind = op.kind as OperationKind;

// ✅✅ often better — if the widening rationale isn't load-bearing for this line's reader, delete.
const kind = op.kind as OperationKind;
```

```
// ❌ bad — narrates readable code
// loop through users and check if active
for user in users:
  if user.isActive: ...

// ✅ good — non-obvious data quirk
// Expired trials still show as "active" in the DB; filter by lastLogin to catch real usage.
for user in users:
  if user.isActive and user.lastLogin > cutoff: ...
```

## 6. Commits

Each commit is a self-contained, reviewable unit. The message says what changed in as few words as possible.

- Symbol prefix: `+` new, `-` removal, `~` tweak, `>` larger change, `!` bugfix, `@` docs/config
- Lowercase, no period, short
- Scope in parentheses when useful: `+ (auth) session tokens`
- One commit per logical change — not per file, not per hour
- Verify behavior before committing

```
// bad
fixed stuff
Updated code

// good
! (parser) off-by-one in heading detection
+ (api) rate limiting middleware
```

## 7. Directory Structure

A directory should reflect a decision, not an accident. The enemy isn't any specific topology — Rails-style layer-slicing, Clean Architecture layering, data-oriented clustering, and kernel-style deep taxonomies all work when chosen on purpose. The enemy is "I didn't know where else to put it."

Four diagnostic questions for any directory:

- **What changes together lives together.** If fixing one thing touches six directories, the axis is wrong. The right axis — features, layers, or data — depends on your team and your domain. Pick one you can defend, and make it legible from the tree alone.
- **Who depends on whom.** Directories form a DAG. Volatile code depends on stable, not the reverse. A dependency cycle between two directories means the boundary isn't real — merge them or re-draw.
- **Could a new reader form this tree from first principles?** If the layout doesn't match the mental model a contributor would build after a week of reading the code, the tree is fighting the architecture. Fix the tree.
- **Name the concept, not the shelf.** Directories named after what they *are* (`parser/`, `auth/`, `users/`) survive refactors. Directories named after what they *do* (`managers/`, `handlers/`, `services/`, `providers/`, `utils/`) don't — architectural roles drift. Anything ending in `-ers` or `-ors` is usually a shelf, not a boundary.

```
// bad: accidents dressed as decisions
src/
  utils/        — dumping ground
  helpers/      — second dumping ground, overlaps with utils/
  managers/     — role-named; what do they manage?
  common/       — third dumping ground

// good: decisions you can defend
src/
  auth/         — everything that touches identity
  billing/      — everything that touches payments
  parser/       — everything that touches the syntax tree
```

## Quick Reference

| Principle | One-line rule |
|-----------|--------------|
| Cardinal Rule | Improve what you touch — don't match existing bad style |
| Simplicity | No abstraction until the third repetition |
| Naming | Name by role/purpose, not implementation |
| Decomposition | If you'd say "and" to describe it, split it |
| File Structure | Section dividers, public API at top, colocate types |
| Comments | Explain why, never what |
| Commits | Symbol prefix, lowercase, one logical change per commit |
| Directory Structure | A directory should reflect a decision, not an accident |

## Common Mistakes

| Mistake                                             | Fix                                                                |
| --------------------------------------------------- | ------------------------------------------------------------------ |
| Matching existing bad style when adding code        | Improve what you touch — rename, restructure, add dividers         |
| Commenting what code does instead of why            | Delete the comment or improve the code                             |
| Matching existing over-commented style when editing a file | Prune comments that fail the "why, non-obviously" test — you own signal-to-noise on your way out |
| Creating `utils`/`helpers`/`common` dumping grounds | Name the concept that lives there (`auth`, `parser`, `billing`), or move each file next to its real consumer |
| Over-documenting with JSDoc on every function       | Doc comments only where the signature doesn't tell the story       |
| Giant orchestrator function that "does one thing"   | If it's 50+ lines of sequential steps, the steps are the functions |
| Role-named directories (`managers/`, `services/`, `handlers/`, `providers/`) | Name by the noun that lives there. Anything ending in `-ers` or `-ors` is probably a shelf, not a boundary |
| Re-export shim kept "for compatibility" long after the migration completed | Delete the shim. Update the remaining callers — the "legacy" is the shim itself. |
| 200+ line block of pure logic embedded in a framework component | Extract as a `createX(deps): X` factory. Component becomes the wiring shell; factory is unit-testable in isolation. |
