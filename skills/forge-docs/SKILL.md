---
name: forge-docs
description: Use when writing, modifying, or reviewing project documentation — README, design docs, CLAUDE.md, architecture overviews, changelogs. Applies principles for abstraction level, stability, conciseness, and diagram-first communication
---

# Forge Docs

Documentation covers architecture, not implementation. Readers use docs to grasp the system's shape fast — then read code for details.

**Sibling skill:** `forge-style` covers code style. This skill covers documentation style.

## Principles

### 1. Abstraction Over Implementation

Describe layers, data flows, responsibilities — not function signatures or file contents.

- No code snippets unless architecturally significant and very unlikely to change
- No function/method signatures — the source is authoritative for those
- No column-level schema tables — describe what the tables represent, not their columns
- If it duplicates the source, it will go stale. Stay above the implementation.

### 2. Brevity

5 words over a paragraph. Table over prose. Bullet over sentence.

- Cut filler words and hedging
- One idea per sentence
- If you need "and" to describe it, split it

### 3. Stability

Docs should survive minor version bumps without edits.

- No exact test counts, line numbers, or version-specific numbers
- Describe capabilities ("supports emphasis, code spans, links"), not mechanisms ("calls scanBacktickSpans then processEmphasis")
- No internal function names — describe what the system does, not what the functions are called
- If it changes every sprint, it doesn't belong in docs

### 4. Diagrams First

A diagram replaces 50 lines of prose.

| Subject | Diagram type |
|---------|-------------|
| Architecture layers | Vertical stack |
| Data flow | Directed graph |
| Component relationships | Tree |
| Decision logic | Flowchart |
| Entity relationships | ER diagram |

Use ASCII, Mermaid, or Graphviz — whatever renders in the repo's tooling.

### 5. README = Landing Page

Three questions, answered fast:

1. What is this?
2. How do I run it?
3. Where next?

Link to deeper docs. No essays.

### 6. Roadmap vs. Changelog

Roadmap is forward-looking. Changelog is the record. They are separate documents for a reason — keep them separate.

- When a milestone ships, move its entry out of the roadmap into the changelog **in the same commit that ships the feature**. If you can't hold that discipline, the roadmap rots.
- Roadmap contains nothing already done.
- Changelog contains nothing still speculative.
- Language matching status: roadmap uses "will", "planned"; changelog uses past tense and version numbers.

The failure mode is roadmap entries that describe shipped features as if they're upcoming. Readers trust the wrong document and plan against stale information.

## What NOT to Document

- Patterns discoverable from framework docs
- Anything derivable from `git log`, `git blame`, or reading the source
- Temporary state, in-progress work, conversation context
- Implementation details the code already expresses

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Pasting code (structs, interfaces, configs) | Describe shape in words: "holds user identity, settings cache, and DB pool" |
| Listing file paths in tables | Describe directory roles: "core/ has parser, serializer, types" |
| Listing function signatures or command APIs | Describe capabilities: "CRUD for sources, search, settings" |
| Prose where a diagram works | Draw the diagram |
| Hardcoded counts or versions | Qualitative description, or omit entirely |
| Separate docs for thin layers | One doc covering the full vertical slice |
| Describing internal pipeline stages by function name | Describe what the pipeline does, not what the functions are called |
| "It should be noted that..." | Delete. Say the thing. |
| Shipped milestone still listed in the roadmap | Move to changelog in the same commit that ships the feature. Roadmap is forward-looking only. |
| Changelog promises features that haven't shipped | The changelog records what happened. Speculative entries belong in the roadmap until they land. |
