Here's the thing about agents writing code: they rarely write badly because they don't know better. They write badly because the context invites it. Drop an agent into a messy file and it politely matches the mess. Ask it to explain itself and it writes a paragraph above every line. Ask it for tests and you get the happy path, driven through a programmatic backdoor no user will ever take. Ask it to review and it produces twenty findings, of which six are real. Each of these skills exists to slam one of those doors shut.

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

Any other harness that reads a `SKILL.md` with YAML frontmatter works the same way. 
