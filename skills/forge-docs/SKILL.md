---
name: forge-docs
description: Use when writing, modifying, or reviewing project documentation (README, design docs, CLAUDE.md, architecture overviews, changelogs), or when a behavior change leaves prose that may now be wrong
---

# Forge Docs

What belongs in project docs, how to write it so it's read once and understood, and how to keep it true. If the project or user has a voice or style guide, it decides how prose sounds; this skill decides what documentation covers and where it lives.

## Write for one reader, reading once

The reader didn't write the code and opened the repo today. A sentence they'd have to read twice, or ask about, is a defect.

- **Plain words over the system's words.** A term that only means something inside this codebase gets replaced with the plain phrase, or defined.
- **Define every term where it first appears**, by what the thing is. A forward reference says where the definition is ("defined in Records, below"), never just "later".
- **Say each thing once, organized by topic.** Structure the doc around how the explanation flows, not around the order things were built or the outline of the doc you're replacing. Two sections about one idea become one.
- **No emphasis padding.** A second clause that only restates the weight of the first ("the parser is total; no input is ever rejected") goes, or becomes its own plain sentence that adds something.
- **A section about a method shows it.** A snippet of the call and its example output, so a reader can check the shape by reading instead of opening the type.

## Stay above the source, except where the reader needs the code

A design doc explains the system's shape (its layers, its data flow, who owns what, and why it's built that way) so a reader knows where to look before reading code. Anything that restates the source (a struct's fields, a table's columns, a pipeline narrated function by function) goes stale on the next refactor and tells the reader nothing the code doesn't.

Code in a doc earns its place when the reader will copy it or check against it: the call a consumer makes and what it returns, the config they paste, the shape of a guard they'll add. A load-bearing API gets a small reference entry: its signature, what it's for, its arguments, the common case with example output, and a pointer to the full reference. Name the entry point a reader will search for; don't narrate the internals by function name.

## Keep it stable

A doc should survive routine releases without edits. Leave out test counts, line numbers and version-specific numbers, or elide them in shown output. Describe capabilities ("supports emphasis, code spans, links"), not mechanisms that change every sprint.

Where a doc does name files and symbols, name them in a form a script can resolve (a backticked path, a path plus a symbol name), so a check in the lint gate can fail when one moves instead of a reader finding out.

## A behavior change sweeps its prose

When behavior changes, every sentence describing it gets a verdict, across every doc and every shipped source note that mentions it. Grep for the symbol finds some of them; the sentences that describe the old behavior in plain words, with nothing searchable in them, are the ones that survive a grep sweep and mislead the next reader. Do it in the commit that changes the behavior.

## Diagrams where they earn their place

A diagram beats prose for layers, data flow, ownership trees and decision logic. Use whatever renders in the repo's tooling. Mermaid can't misalign; ASCII can, so check it in a monospace render before committing.

## One doc per slice

A thin layer rarely needs its own doc; one doc covering the whole vertical slice reads better than three that each hold a fragment. A long doc opens with a section map so a reader can jump to their question.

## README

The README answers three things first: what this is, how to run it, where to go next. It links to the deeper docs rather than carrying them.

## Records

- **The changelog is the past; plans are the future.** A shipped item moves into the changelog in the same commit that ships it, in past tense. A roadmap, where a project keeps one, holds nothing already done; a changelog holds nothing speculative. The failure is a shipped feature still described as upcoming, and readers planning against it.
- **A decision lives with the contract it binds**, in the design doc for that subsystem, not in a plan that gets archived.
- **Tracked docs point only at tracked content.** A reference to a scratch note, a gitignored plan, or "§ 4.2 of the spec" that was never committed dangles the moment it ships. Give the content a tracked home first.

## Leave out

Anything the framework's own docs cover, anything `git log` or the source already answers, in-progress state and conversation context.
