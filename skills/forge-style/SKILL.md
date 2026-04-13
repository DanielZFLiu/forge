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

Comments explain _why_, never _what_. If the code needs a comment to explain what it does, improve the code first.

- Explain non-obvious reasoning, workarounds, deliberate exclusions
- Don't narrate readable code — no `// increment counter` above `counter++`
- Doc comments only for public APIs and non-obvious edge cases — not every function
- Link to issues/tickets when working around a bug

```
// bad: narrates the obvious
// loop through users and check if active
for user in users
  if user.isActive ...

// good: explains a non-obvious data quirk
// expired trials still show as "active" in the DB;
// filter by lastLogin to catch actual usage
for user in users
  if user.isActive and user.lastLogin > cutoff ...
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

## Common Mistakes

| Mistake                                             | Fix                                                                |
| --------------------------------------------------- | ------------------------------------------------------------------ |
| Matching existing bad style when adding code        | Improve what you touch — rename, restructure, add dividers         |
| Commenting what code does instead of why            | Delete the comment or improve the code                             |
| Creating `utils`/`helpers`/`common` dumping grounds | Group by domain: "authentication," "parsing," not "miscellaneous"  |
| Over-documenting with JSDoc on every function       | Doc comments only where the signature doesn't tell the story       |
| Giant orchestrator function that "does one thing"   | If it's 50+ lines of sequential steps, the steps are the functions |
