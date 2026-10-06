---
name: forge-style
description: Use when writing, modifying, or reviewing code, including code comments and commit messages, and especially when the surrounding code is messy enough to tempt you to match it
---

# Forge Style

How code should read and where its rules should live, in any language. Sibling skills: `forge-tests` for tests, `forge-docs` for docs, `forge-review` for audits. A project's own rules (its CLAUDE.md, a contributing guide) win where they are more specific.

## Improve what you touch

Code you add in the style of the code around it spreads that style, and the moment you're already editing a file is the cheapest time to fix it. So rename the vague name you had to decode, split the mode-parameter function you're adding a mode to, prune the comments you read on the way. The limit is your path, not the whole file: fix what your change reads or edits, and when the cleanup outgrows the change (a rename with thirty call sites, a file split), land it as its own commit so each diff stays reviewable. Renaming something exported means updating its callers in the same change, or leaving it.

## Where a rule lives

A rule that several routes must follow (every edit that joins two blocks, every path that places the caret, every caller that needs the same context) belongs in one place all of those routes already pass through. Copied to each route, it drifts: one copy learns a case the others don't, a new route arrives without it, and the bug is always "enforced at N-1 of N paths". When you find one broken copy, list every sibling route before fixing any.

Contracts climb an enforcement ladder as high as they can:

1. **Unrepresentable.** A type or a shared entry point that makes the violation impossible to write. A required parameter beats a convention to pass it.
2. **Guarded.** A check at that shared entry point that fails a test gate in development. Where the shared route can't be built yet, a test that reads the source and fails when a new route skips the rule catches path N+1 the day it's written.
3. **Documented.** Prose, only for what neither of the above can hold.

A bug fix closes the class, not the instance: find the rule the bug broke, move it up a rung if you can, and add the guard that would have caught it.

## When to abstract

Abstract when two pieces of code exist for the same reason, not when they have the same shape. The asymmetry decides it: undoing a premature abstraction inside a codebase is a mechanical job (find its call sites, inline it), while unifying copies that have drifted apart is a real one, because each copy picked up its own fixes and you have to work out which differences are bugs.

- At the second copy, ask whether both copies enforce one rule or keep one promise. If they do, extract now, before a third route copies whichever of the two it finds first.
- If they merely look alike (two loops over different data for different ends), leave them. A third copy is the tiebreaker when you can't tell.
- A wrong abstraction announces itself: each new caller adds a flag or a special case. Inline it back and let the callers diverge.
- The asymmetry flips at a published API. Inlining an export breaks its consumers, so abstractions at a public boundary wait for evidence.

## Simplicity

- Build for the cases you have, not the ones you can imagine.
- Prefer flat control flow and shallow wrapper chains.
- Delete dead code; git remembers it.
- A migration shim (a re-export for old import paths, a rename alias, a compatibility wrapper) has a deadline: when the last caller moves, delete it. "Keeps old callers working" stops being true long before anyone removes the shim.

## Naming

Names are the first layer of documentation, so name by role (`userRecords`, not `hashMap`) and call one concept by one name everywhere. If it's `user` in one module and `account` in the next, a reader has to find out whether they're the same thing. Generic names (`data`, `item`, `temp`) are fine in a scope of a few lines and nowhere else.

## Decomposition

Each function, file and module has a responsibility you can state in one short sentence. If the sentence needs "and" between two jobs that change for different reasons, split it; reading and writing one format is one job. An exported function whose behavior switches on a mode or flag argument is two functions sharing a signature; give each its own name, and let a shared core inside take a direction as data. A long orchestrator that "does one thing" in fifty sequential steps usually has its steps as the missing functions.

Logic that doesn't use a UI framework's lifecycle (no effects, no rendering, no DOM refs) belongs in a plain module the component calls, where it can be tested without mounting anything. Pass it its dependencies explicitly, and pass values that change as reads (a getter, an accessor function), never as a copy captured at construction, which goes stale the moment the source changes. If the module needs everything the component has, it isn't independent yet; leave it. A factory is a unit too: the one-sentence test applies to its closure, and a closure holding several independent pieces of state is several factories.

## File structure

A reader should see a file's shape without reading every line. Public API and main exports near the top, internals below. Types sit next to the code that uses them; a shared type lives in the lowest layer every user can import. Section dividers mark logical groups, in the language's comment syntax, and a file that needs more than a handful of them is usually several files:

```
// ── Public API ──────────────────────────
```

## Comments

Default to none. The reader is a competent developer who has never seen this repo, reading once. Two kinds earn their lines:

- A **header** (top of a file, or above a module's contract) says what the thing is for in one sentence a newcomer can read, then at most the one thing a caller must get right. A file needs one when its name and exports don't already say that.
- A **body comment** says why this line is the way it is, in one plain sentence: the non-obvious choice, the workaround, the deliberate exclusion.

Neither narrates how the code works (the code, types and names carry that), and neither argues for the choice over its alternatives; that argument was for the reviewer, and the code keeps only the conclusion. Git carries when and who, the issue tracker carries what's next, design docs carry how things fit together.

**Budget.** A comment is 1-2 lines; a header is at most about 5. A why that needs more moves to a design doc and leaves a one-line pointer.

**Contract or justification.** For a borderline block, ask which it states. A contract (what this module guarantees, what a caller must do) earns its lines even at the header budget. A justification (why this beat another design, what broke before) goes, even at two lines.

**Words.** A comment names its subject (the parser, the scroll container, the undo stack), never "this" or "the layer". Project-private words get the plain phrase instead, or a gloss of three words or fewer where a symbol's name forces them. A catalog code (`R-12`) isn't a reason; say what holds. Capitals for emphasis mean the sentence is carrying too much.

**The test.** If removing the comment wouldn't confuse a reader, delete it, including comments you didn't write in a file you're editing. Only a comment whose purpose you can't work out is worth leaving alone.

What fails the test, most often:

| Shape | Why it goes |
|---|---|
| Narrating the next line, or restating the name in prose | Says nothing the code doesn't |
| Enumerating union members, enum values or flags | Lies the day a variant is added |
| History: versions ("post-0.5"), past states ("used to"), resolved-issue citations, callers or tickets ("added for #123") | The reader needs the current contract; history lives in git. A citation of an open issue marking a known gap stays. |
| Rationale essays, multi-paragraph docstrings on internal functions | Over budget; the essay goes to a design doc, the docstring collapses to one sentence |
| A claim with no subject ("Rows before columns.") | The reader has to reconstruct what it's about |
| A TODO buried in prose | Invisible to grep; write `TODO(owner): reason` or file an issue |

```
// bad: enumerates the union, narrates the cast, cites a version
// The public interface uses `string` for op.kind; narrow to the internal
// OperationKind union here. Callers pass known kinds ('split' | 'merge' |
// 'delete'); the cast can be tightened post-0.5.4.
const kind = op.kind as OperationKind;

// fine, if a reader would otherwise wonder
// The public interface widens kind to string; OperationKind is the internal source of truth.
const kind = op.kind as OperationKind;
```

## Commits

Each commit is one logical change a reviewer can read on its own, verified before it's made.

- Symbol prefix: `+` new, `-` removal, `~` tweak, `>` larger change, `!` bug fix, `@` docs or config.
- Lowercase, no trailing period, scope in parentheses when useful: `! (parser) off-by-one in heading detection`.
- Subject lines only. A body is exceptional: 2-3 short lines the subject can't carry.
- A commit holding several changes puts one summary on line 1, a blank line, then one symbol-prefixed line per change, never a subject plus paragraphs.
- No `Co-Authored-By` line and no "Generated with" attribution.

## Directory structure

A directory should reflect a decision, not "I didn't know where else to put it". Any topology works when chosen on purpose (by feature, by layer, by data). Four questions test one:

- **What changes together lives together.** When one kind of fix keeps landing in the same handful of directories, those pieces belong in one. A feature that crosses each layer once, through that layer's registry or entry point, is layering working.
- **Who depends on whom.** Directories form a DAG, volatile code depending on stable code, and a check in the test gate holds the order; without one it decays. A runtime cycle between two directories means the boundary isn't real; merge them or redraw it. A cycle made only of type imports means a shared type sits too high; move it down.
- **Could a new reader find things from the tree alone?** Pick five behaviors at random; a newcomer should name the directory for each from the directory names. Where they can't, fix the tree before writing the index that explains it.
- **Name the concept, not the shelf.** `parser/`, `auth/`, `billing/` survive refactors. A role (`utils/`, `helpers/`, `managers/`) or a mechanism (`reactivity/`, `stores/`) is a shelf; domain plurals like `parsers/` are fine. File names follow the same rule, and a project-private word a comment would have to gloss doesn't name a file either. When a file's contents outgrow its name, rename it; many siblings sharing a prefix are a directory waiting to be named.
