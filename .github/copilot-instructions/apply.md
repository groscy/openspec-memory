---
description: "Implement tasks from an OpenSpec change"
agent: agent
---
<!-- Source: openspec/skills/apply/SKILL.md | Trigger: /opsx:apply -->

# OpenSpec Skill: apply

**Trigger**: `/opsx:apply`

**Usage**: Start a Copilot Chat message with `/opsx:apply` (optionally followed by a change name) to invoke this skill.

---

Implement tasks from an OpenSpec change.

**Input**: Optionally specify a change name (e.g., `/opsx:apply add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

0. **Load domain knowledge** (before any other step)

   Check if `openspec/domain/_index.yaml` exists.
   - **If yes**: read `_index.yaml` into context. Scan the change name and task descriptions for concept names; load full concept files for matching names.
   - **If no**: proceed without domain context. No error.

1. **Select the change**

   If a name is provided, use it. Otherwise infer from context, auto-select if only one active change exists, or ask the user to select from `openspec list --json`.

   Always announce: "Using change: <name>"

2. **Check status**
   ```bash
   openspec status --change "<name>" --json
   ```

3. **Get apply instructions**
   ```bash
   openspec instructions apply --change "<name>" --json
   ```
   Handle states: `blocked` → show message; `all_done` → congratulate and suggest archive; otherwise proceed.

4. **Read all context files** listed under `contextFiles` in the apply instructions output.

5. **Show current progress**: schema, N/M tasks complete, remaining tasks.

6. **Implement tasks (loop until done or blocked)**

   For each pending task:
   - Show which task is being worked on
   - Make the code changes required
   - Keep changes minimal and focused
   - Mark task complete: `- [ ]` → `- [x]`
   - Continue to next task

   Pause if: task is unclear, implementation reveals a design issue, or a blocker is encountered.

7. **On completion or pause**: show tasks completed, overall progress. If all done suggest archive. If paused explain why.

**Guardrails**
- Keep going through tasks until done or blocked
- Always read context files before starting
- Keep code changes minimal and scoped to each task
- Update task checkbox immediately after completing each task
- Pause on errors, blockers, or unclear requirements — don't guess
