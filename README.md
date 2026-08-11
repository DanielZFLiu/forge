# Forge

Four skills that try to keep AI-assisted code from reading like AI-assisted code. I wrote them for my own agents while building [aragonite](https://github.com/voithos-labs/aragonite), where a small army of subagents has shipped code, docs, tests, and reviews under these exact rules [^1]. They are language agnostic: the examples are pseudocode, the principles port.

Here's the thing about agents writing code: they rarely write badly because they don't know better. They write badly because the context invites it. Drop an agent into a messy file and it politely matches the mess. Ask it to explain itself and it writes a paragraph above every line. Ask it for tests and you get the happy path, driven through a programmatic backdoor no user will ever take. Ask it to review and it produces twenty findings, of which six are real [^2]. Each of these skills exists to slam one of those doors shut.

| Skill | The door it closes |
|---|---|
| `forge-style` | Matching bad style. The cardinal rule is *improve what you touch*: naming, decomposition, file and directory structure, commits, and a comment discipline with an actual budget (why-only, 1-2 lines, delete on sight otherwise). |
| `forge-docs` | Docs that restate the source. Architecture over implementation, brevity, stability rules so docs survive version bumps, diagrams first, and roadmap/changelog hygiene. |
| `forge-tests` | Tests that can't fail. Every test answers "if someone broke this, would it fail?": edge cases over happy paths, real keyboard/mouse simulation over programmatic shortcuts, requirement-driven e2e, and a miss-analysis habit for every bug that slips through. |
| `forge-review` | Review by vibes. A four-pass audit methodology (bugs, design, tests, organization) where every finding is verified by reproduction or revert-probe before it's recorded, fixes land in risk tiers, and multiple agents can work one checkout without eating each other's commits. |

The four reference each other by name (style is the floor for tests, review leans on all three), so keep them together.

# Usage

For Claude Code, copy the four directories into your skills folder:

```
cp -r skills/forge-* ~/.claude/skills/
```

Any other harness that reads a `SKILL.md` with YAML frontmatter works the same way; the frontmatter `description` says when each skill should activate. If you dispatch subagents, name the skills in the brief and have the agent invoke them before touching anything. That one sentence in your prompt is most of the value of this repo.

# Why trust these

Because they aren't aspirational. Nearly every rule in here was paid for: a review that shipped a false "covered by existing tests" claim funded the revert-probe rule, two deterministic-looking test failures that turned out to be sub-pixel click rounding funded the gesture-determinism rule, and the concurrent-writers git discipline was learned across four real collisions in one evening [^3]. When a rule stops earning its keep, it gets deleted; the skills stay short on purpose.

# License

Copyright (c) 2026 DanielZFLiu. [MIT](./LICENSE): use them, fork them, rewrite them in your own voice, no strings.

[^1]: the number that matters: aragonite's test suite is ~144k lines and its comment density went *down* while agents wrote most of it. That is what these skills are for.

[^2]: the other fourteen cost you an afternoon each. Verification discipline is cheaper.

[^3]: same evening, four different agents, four different ways to trip over a shared git index. The skill now knows all four.
