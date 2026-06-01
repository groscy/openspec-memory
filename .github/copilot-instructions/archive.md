---
description: "Archive a completed change in the experimental workflow"
agent: agent
---
<!-- Source: openspec/skills/archive/SKILL.md | Trigger: /opsx:archive -->

# OpenSpec Skill: archive

**Trigger**: `/opsx:archive`

**Usage**: Start a Copilot Chat message with `/opsx:archive` (optionally followed by a change name) to invoke this skill.

---

Archive a completed change in the experimental workflow.

**Input**: Optionally specify a change name. If omitted, run `openspec list --json` and ask the user to select. **Do NOT auto-select.**

**Steps**

1. **Select change**: prompt for selection if not provided — never guess.

2. **Check artifact completion**: `openspec status --change "<name>" --json`. Warn if any artifacts are incomplete; ask to confirm before continuing.

3. **Check task completion**: count `- [ ]` vs `- [x]` in `tasks.md`. Warn if incomplete; ask to confirm.

4. **Domain knowledge review** (skip if `openspec/domain/` does not exist):
   - Load `_index.yaml`
   - Compare completed change against domain KB; surface discovery candidates
   - For each confirmed discovery, write `openspec/domain/discovered/<name>.md` with `status: active`
   - Check active discoveries for contradictions; offer to mark superseded
   - Regenerate `_index.yaml`

5. **Delta spec sync**: if delta specs exist in `openspec/changes/<name>/specs/`:
   - Compare each with the corresponding `openspec/specs/<capability>/spec.md`
   - Show a combined diff summary
   - Offer "Sync now (recommended)" or "Archive without syncing"

6. **Perform archive**:
   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

7. **Show summary**: change name, schema, archive location, spec sync status, any warnings.

**Guardrails**
- Always prompt for change selection if not provided
- Don't block archive on warnings — inform and confirm
- Preserve `.openspec.yaml` when moving (it moves with the directory)
