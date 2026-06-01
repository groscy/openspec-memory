# OpenSpec Skill: propose

**Trigger**: `/opsx:propose`

**Description**: Propose a new change - create it and generate all artifacts in one step

**Usage**: Start a Copilot Chat message with `/opsx:propose` (optionally followed by a change name or description) to invoke this skill.

<!-- Source: openspec/skills/propose/SKILL.md -->

---

Propose a new change - create the change and generate all artifacts in one step.

Creates: proposal.md (what & why), design.md (how), tasks.md (implementation steps).

**Input**: The argument after `/opsx:propose` is the change name (kebab-case) or a description of what to build.

**Steps**

0. **Load domain knowledge**: read `openspec/domain/_index.yaml` if it exists. Scan change description for concept names; flag rule conflicts before finalizing.

1. **If no input provided**: ask "What change do you want to work on? Describe what you want to build or fix." Derive a kebab-case name from the description.

2. **Create the change**:
   ```bash
   openspec new change "<name>"
   ```

3. **Get artifact build order**:
   ```bash
   openspec status --change "<name>" --json
   ```

4. **Create artifacts in dependency order** until all `applyRequires` artifacts are `done`:
   - Get instructions: `openspec instructions <artifact-id> --change "<name>" --json`
   - Use the `template` field as structure; apply `context` and `rules` as constraints (do NOT copy them into the file)
   - Ask the user if context is critically unclear

5. **Show final status**: `openspec status --change "<name>"`

**Output**: Change name, list of artifacts created, "Run `/opsx:apply` to start implementing."

**Guardrails**
- Create ALL artifacts needed for implementation
- Always read dependency artifacts before creating a new one
- If a change with that name already exists, ask if user wants to continue it or create a new one
