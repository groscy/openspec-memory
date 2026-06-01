# OpenSpec Skill: explore

**Trigger**: `/opsx:explore`

**Description**: Enter explore mode - think through ideas, investigate problems, clarify requirements

**Usage**: Start a Copilot Chat message with `/opsx:explore` (optionally followed by a topic) to enter explore mode.

<!-- Source: openspec/skills/explore/SKILL.md -->

---

Enter explore mode. Think deeply. Visualize freely. Follow the conversation wherever it goes.

**IMPORTANT: Explore mode is for thinking, not implementing.** Read files and investigate the codebase freely, but never write code or implement features. Creating OpenSpec artifacts (proposals, designs, specs) is fine.

**Input**: Whatever the user wants to think about — a vague idea, specific problem, change name, comparison, or nothing.

---

## The Stance

- **Curious, not prescriptive** — ask questions that emerge naturally
- **Open threads** — surface multiple directions; don't funnel
- **Visual** — use ASCII diagrams liberally
- **Adaptive** — follow interesting threads, pivot when new information emerges
- **Grounded** — explore the actual codebase when relevant

## Domain Knowledge Loading

At the start, check for `openspec/domain/_index.yaml` and read it if present. Load individual concept files only as they become relevant in conversation.

## OpenSpec Awareness

At the start run `openspec list --json` to see active changes. If the user mentions a change, read its artifacts for context.

When decisions crystallize, offer to capture them:

| Insight Type | Where to Capture |
|---|---|
| New requirement | `specs/<capability>/spec.md` |
| Design decision | `design.md` |
| Scope change | `proposal.md` |
| New work | `tasks.md` |

**Guardrails**
- Never write code or implement features
- Don't auto-capture — offer and let the user decide
- Don't rush to conclusions; do visualize and explore the codebase
