---
name: archive
description: "Archive a completed change in the experimental workflow"
triggers:
  - /opsx:archive
agents: [claude, copilot, intellij]
args: "[change-name]"
version: "1.0.0"
---

Archive a completed change in the experimental workflow.

**Input**: Optionally specify a change name after `/opsx:archive` (e.g., `/opsx:archive add-auth`). If omitted, check if it can be inferred from conversation context. If vague or ambiguous you MUST prompt for available changes.

**Steps**

1. **If no change name provided, prompt for selection**

   Run `openspec list --json` to get available changes. Use the **AskUserQuestion tool** to let the user select.

   Show only active changes (not already archived).
   Include the schema used for each change if available.

   **IMPORTANT**: Do NOT guess or auto-select a change. Always let the user choose.

2. **Check artifact completion status**

   Run `openspec status --change "<name>" --json` to check artifact completion.

   Parse the JSON to understand:
   - `schemaName`: The workflow being used
   - `artifacts`: List of artifacts with their status (`done` or other)

   **If any artifacts are not `done`:**
   - Display warning listing incomplete artifacts
   - Prompt user for confirmation to continue
   - Proceed if user confirms

3. **Check task completion status**

   Read the tasks file (typically `tasks.md`) to check for incomplete tasks.

   Count tasks marked with `- [ ]` (incomplete) vs `- [x]` (complete).

   **If incomplete tasks found:**
   - Display warning showing count of incomplete tasks
   - Prompt user for confirmation to continue
   - Proceed if user confirms

   **If no tasks file exists:** Proceed without task-related warning.

4. **Domain knowledge review** (skip this entire step if `openspec/domain/` does not exist)

   a. **Load domain index**

      Read `openspec/domain/_index.yaml`. If it is missing or malformed YAML, warn "⚠ Domain index unreadable — skipping domain review." and skip to step 5.

   b. **Read change artifacts for comparison**

      Read the change's `tasks.md` and any specs under `openspec/changes/<name>/specs/`. Use these to understand what the completed change introduced or changed.

   c. **Surface discovery candidates**

      Compare the completed change's implementation against the domain KB:
      - Look for behaviors, facts, or constraints in the implementation that are absent from or not reflected in any existing concept, rule, or active discovery
      - Look for facts that are implied by the implementation but not explicit in the domain KB

      For each candidate, propose it with a concise description:

      ```
      ## Domain Discovery Candidates

      Found 2 potential discoveries from this change:

      1. **invoice-grace-period**: Invoices have a 5-day grace period after the due date before entering overdue state — not reflected in current domain KB.
         Capture this? (yes / no)

      2. **payment-partial-allowed**: Partial payments are accepted and tracked separately — not currently in the domain KB.
         Capture this? (yes / no)
      ```

      If no candidates are found: skip the capture prompt and continue. Do NOT force candidates.

   d. **Capture confirmed discoveries**

      For each candidate the user confirms:
      - Ask for a brief description if not already clear
      - Create the discovery file at `openspec/domain/discovered/<kebab-name>.md`:

        ```markdown
        ---
        name: invoice-grace-period
        type: discovery
        discovered_in: <change-name>
        date: <today ISO 8601>
        relates_to: [Invoice]
        status: active
        superseded_by: null
        ---
        <description of the discovery>
        ```

      For candidates the user declines: skip — no file created.

   e. **Check for contradictions with active discoveries**

      Scan all `active` discoveries in the index. For each one, check whether the completed change explicitly contradicts or supersedes it (e.g., the change replaces a behavior the discovery documented).

      For each contradiction found:
      ```
      ⚠ This change may contradict an active discovery:

      Discovery: invoice-grace-period (discovered in: add-overdue-tracking)
      Current behavior per discovery: Invoices enter overdue immediately at due date
      What this change does: Introduces a 5-day grace period

      Mark this discovery as superseded? (yes / no)
      ```

      If yes: update the discovery file's `status` to `superseded` and `superseded_by` to `<change-name>`.
      If no: leave the discovery unchanged.

      If no contradictions found: continue silently.

   f. **Regenerate domain index**

      Run the Index Generation Procedure (as defined in `/opsx:domain`):
      - Scan all files under `openspec/domain/static/` and `openspec/domain/discovered/`
      - Validate frontmatter; warn and skip invalid files
      - Write updated `openspec/domain/_index.yaml`
      - Report: "Domain index regenerated (N concepts, M processes, P rules, Q discoveries)"

5. **Assess delta spec sync state**

   Check for delta specs at `openspec/changes/<name>/specs/`. If none exist, proceed without sync prompt.

   **If delta specs exist:**
   - Compare each delta spec with its corresponding main spec at `openspec/specs/<capability>/spec.md`
   - Determine what changes would be applied (adds, modifications, removals, renames)
   - Show a combined summary before prompting

   **Prompt options:**
   - If changes needed: "Sync now (recommended)", "Archive without syncing"
   - If already synced: "Archive now", "Sync anyway", "Cancel"

   If user chooses sync, use Task tool (subagent_type: "general-purpose", prompt: "Use Skill tool to invoke openspec-sync-specs for change '<name>'. Delta spec analysis: <include the analyzed delta spec summary>"). Proceed to archive regardless of choice.

5. **Perform the archive**

   Create the archive directory if it doesn't exist:
   ```bash
   mkdir -p openspec/changes/archive
   ```

   Generate target name using current date: `YYYY-MM-DD-<change-name>`

   **Check if target already exists:**
   - If yes: Fail with error, suggest renaming existing archive or using different date
   - If no: Move the change directory to archive

   ```bash
   mv openspec/changes/<name> openspec/changes/archive/YYYY-MM-DD-<name>
   ```

6. **Display summary**

   Show archive completion summary including:
   - Change name
   - Schema that was used
   - Archive location
   - Spec sync status (synced / sync skipped / no delta specs)
   - Note about any warnings (incomplete artifacts/tasks)

**Output On Success**

```
## Archive Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** ✓ Synced to main specs
**Domain:** ✓ Index regenerated (N concepts, M processes, Q rules, R discoveries)

All artifacts complete. All tasks complete.
```

**Output On Success (No Delta Specs)**

```
## Archive Complete

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** No delta specs
**Domain:** ✓ Index regenerated (or "No domain KB")

All artifacts complete. All tasks complete.
```

**Output On Success With Warnings**

```
## Archive Complete (with warnings)

**Change:** <change-name>
**Schema:** <schema-name>
**Archived to:** openspec/changes/archive/YYYY-MM-DD-<name>/
**Specs:** Sync skipped (user chose to skip)

**Warnings:**
- Archived with 2 incomplete artifacts
- Archived with 3 incomplete tasks
- Delta spec sync was skipped (user chose to skip)

Review the archive if this was not intentional.
```

**Output On Error (Archive Exists)**

```
## Archive Failed

**Change:** <change-name>
**Target:** openspec/changes/archive/YYYY-MM-DD-<name>/

Target archive directory already exists.

**Options:**
1. Rename the existing archive
2. Delete the existing archive if it's a duplicate
3. Wait until a different date to archive
```

**Guardrails**
- Always prompt for change selection if not provided
- Use artifact graph (openspec status --json) for completion checking
- Don't block archive on warnings - just inform and confirm
- Preserve .openspec.yaml when moving to archive (it moves with the directory)
- Show clear summary of what happened
- If sync is requested, use the Skill tool to invoke `openspec-sync-specs` (agent-driven)
- If delta specs exist, always run the sync assessment and show the combined summary before prompting
