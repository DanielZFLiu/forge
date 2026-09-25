Here's the thing about agents writing code: they rarely write badly because they don't know better. They write badly because the context invites it. Drop an agent into a messy file and it politely matches the mess. Ask it to explain itself and it writes a paragraph above every line. Ask it for tests and you get the happy path, driven through a programmatic backdoor no user will ever take. Ask it to review and it produces twenty findings, of which six are real. Each of these skills exists to slam one of those doors shut.

| Skill | The door it closes |
|---|---|
| `forge-style` | Matching bad style. The cardinal rule is *improve what you touch*. The rest: naming, decomposition, when to abstract (same reason, not same shape), keeping a rule in one place so its copies can't drift, file and directory structure, commits, and comments on an actual budget (1-2 lines, never narrating the code, delete on sight otherwise). |
| `forge-docs` | Docs nobody can read, or that quietly went stale. What belongs in a doc and what the code already says, writing for someone reading once (plain words, every term defined where it first shows up, each thing said once), docs that survive a release, sweeping every sentence a behavior change made wrong, and changelog hygiene. If you have your own voice or style guide, that decides how the prose sounds; this one decides what gets written down. |
| `forge-tests` | Tests that can't fail. Every test answers "if someone broke this, would it fail?", and you prove the answer by watching it go red. Boundaries over happy paths, property tests with generators that actually try, real keyboard and mouse over programmatic shortcuts, requirement-driven e2e, and a miss-analysis for every bug that slips through. |
| `forge-review` | Review by vibes. A four-pass audit (bugs, design, tests, organization) where every serious finding gets reproduced or revert-probed before it's recorded, a coverage check catches whatever no pass read, and fixes land in risk tiers, test first. |

The four reference each other by name (style is the floor for tests, review leans on all three), so keep them together.

# Usage

For Claude Code, copy the four directories into your skills folder:

```bash
cp -r skills/forge-* ~/.claude/skills/
```

or in PowerShell:

```powershell
Copy-Item -Recurse skills/forge-* ~/.claude/skills/
```

Any other harness that reads a `SKILL.md` with YAML frontmatter works the same way.

# License

[MIT](./LICENSE). Use them, fork them, rewrite them in your own voice.
