---
name: openspec-domain
description: Manage the OpenSpec domain knowledge base. Use when the user wants to initialize a domain KB, add domain concepts/processes/rules, or sync the domain index.
license: MIT
compatibility: Requires openspec CLI.
metadata:
  author: openspec
  version: "1.0"
  generatedBy: "1.3.1"
---

Manage the OpenSpec domain knowledge base.

**Input**: A subcommand:
- `init` — scaffold the domain KB from scratch via interview + optional codebase read
- `add <type> <name>` — add a single concept, process, rule, or actor entry
- `sync` — regenerate `_index.yaml` from all current frontmatter

If no subcommand is provided, show the subcommands above and stop.

---

## Domain Directory Structure

```
openspec/domain/
├── _index.yaml              # Auto-generated compact index (NEVER hand-edit)
├── static/
│   ├── concepts/            # Domain entities and objects (one file per concept)
│   ├── processes/           # Workflows and multi-step flows (one file per process)
│   ├── glossary.md          # Terminology definitions
│   └── rules.md             # Business constraints and invariants
└── discovered/              # Facts surfaced during development (one file per discovery)
```

The presence of `openspec/domain/` enables domain awareness across all OpenSpec skills automatically.

---

## File Schemas

### Concept / Process Frontmatter

Every file in `static/concepts/` and `static/processes/` requires these frontmatter fields:

```yaml
---
name: Invoice                       # string, canonical concept name (PascalCase)
type: concept                       # concept | process | rule | actor
aliases: [bill, billing-doc]        # array of alternate names (may be empty [])
related: [Customer, Payment]        # array of other concept names (may be empty [])
# Optional fields:
states: [draft, pending, paid]      # valid state names for this concept
rules: [approved-before-payment]    # rule IDs from rules.md that apply
actors: [AccountsPayable]           # actor names relevant to this concept
---
Prose definition. Include: what it is, edge cases, key relationships, known states.
```

Required fields: `name`, `type`, `aliases`, `related`.
Optional fields: `states`, `rules`, `actors`.

### Discovery Frontmatter

Every file in `discovered/` requires:

```yaml
---
name: invoice-disputed-state         # kebab-case string matching the filename
type: discovery                      # must be the literal string "discovery"
discovered_in: add-invoice-dispute   # name of the change that surfaced this fact
date: 2026-05-31                     # ISO 8601 date when this was discovered
relates_to: [Invoice]                # array of static concept names (may be empty [])
status: active                       # active | superseded | invalidated
superseded_by: null                  # set to change name when status becomes superseded
---
Description of the discovery: what was observed, why it matters, what it implies for the domain.
```

Required fields: `name`, `type`, `discovered_in`, `date`, `relates_to`, `status`.
Optional field: `superseded_by` (required when `status: superseded`).

### glossary.md Format

```markdown
### Net Terms
Payment period after invoice date (e.g., Net 30).
```

One `### Term` heading per entry. The line immediately after the heading is the one-line definition.

### rules.md Format

```markdown
### Rule: approved-before-payment
An invoice must be approved before payment can be recorded.
```

One `### Rule: <id>` heading per entry. The line immediately after is the one-line constraint.

### _index.yaml Schema

```yaml
concepts:
  - name: Invoice
    aliases: [bill, billing-doc]
    related: [Customer, Payment]
    states: [draft, pending, approved, paid]
    rules: [approved-before-payment]
    def: One-line definition (first sentence of prose body)
    file: static/concepts/invoice.md
processes: []
glossary:
  - term: Net Terms
    def: Payment period after invoice date (e.g., Net 30)
rules:
  - id: approved-before-payment
    def: An invoice must be approved before payment can be recorded
discoveries:
  - name: invoice-disputed-state
    relates_to: [Invoice]
    status: active
    discovered_in: add-invoice-dispute-flow
    date: 2026-05-31
    def: Invoices can enter a disputed state before resolution
    file: discovered/invoice-disputed-state.md
```

Top-level keys: `concepts`, `processes`, `glossary`, `rules`, `discoveries`. All keys present even when empty.

---

## Index Generation Procedure

Use this procedure whenever a sync is needed (called by `sync`, `init`, and `add`):

1. **Scan concept files**: read all `*.md` files under `openspec/domain/static/concepts/`
2. **Scan process files**: read all `*.md` files under `openspec/domain/static/processes/`
3. **Parse glossary**: read `openspec/domain/static/glossary.md`; extract each `### <Term>` heading and the first non-blank line following it
4. **Parse rules**: read `openspec/domain/static/rules.md`; extract each `### Rule: <id>` heading and the first non-blank line following it
5. **Scan discovery files**: read all `*.md` files under `openspec/domain/discovered/`
6. For each scanned `.md` file: extract the YAML frontmatter block (between `---` delimiters)
7. **Validate** each file (see Validation section); skip files that fail with a warning
8. **Build index entries**: `def` = first sentence of prose body; for superseded discoveries append `[SUPERSEDED]`, for invalidated append `[INVALIDATED]`
9. **Write** `openspec/domain/_index.yaml` with all four top-level keys; use `[]` for empty categories
10. Report: "Index written: N concepts, M processes, P glossary terms, Q rules, R discoveries"

### Validation Rules

Concept/process files — required: `name`, `type`, `aliases`, `related`
- Missing required field → warn `⚠ Skipping <file>: missing required field '<field>'` and skip
- Malformed YAML → warn `⚠ Skipping <file>: malformed frontmatter` and skip

Discovery files — required: `name`, `type`, `discovered_in`, `date`, `relates_to`, `status`
- Missing required field → warn and skip
- `status` not in `[active, superseded, invalidated]` → warn and skip
- `status: superseded` with null/missing `superseded_by` → warn but include with `[SUPERSEDED]` label

---

## Subcommand: init

1. **Check for existing domain** — if `openspec/domain/` exists: warn and ask to reinitialize or cancel
2. **Interview** — ask user to describe their domain; extract concepts, processes, glossary terms, rules, actors
3. **Optional codebase read** — if `src/`, `models/`, `schemas/`, or `routes/` exist, offer to scan and enrich the draft
4. **Show draft summary** — list all extracted entries; ask for confirmation or revisions; do NOT write files until confirmed
5. **Create directories** — `openspec/domain/static/concepts/`, `.../processes/`, `openspec/domain/discovered/`
6. **Write static files** — concept `.md` files, process `.md` files, `glossary.md`, `rules.md`
7. **Generate index** — run Index Generation Procedure; write `_index.yaml`
8. **Configure project skills** — check if `openspec/skills/` exists in the current project:
   - **If yes**: run `openspec install skills` to generate adapter files for all skills found there; capture the list of skill names and target environments from the command output
   - **If no**: warn "No `openspec/skills/` directory found — skipping skill configuration" and continue
9. **Report** — list created domain KB files and entry counts; if skill configuration ran, also list the configured skill names and target environments

---

## Subcommand: add

**Usage**: `add <type> <name>` where `<type>` is `concept`, `process`, `rule`, or `actor`

1. **Parse arguments** — extract type and name; ask if missing
2. **Check domain exists** — error if `openspec/domain/` not found
3. **Check duplicates** — load `_index.yaml`, check for name collision (case-insensitive); if found: warn and ask to update or cancel
4. **Determine file path** — concept/actor → `static/concepts/<kebab>.md`; process → `static/processes/<kebab>.md`; rule → append to `rules.md`
5. **Interview** — gather aliases, related concepts, states, rules (type-appropriate questions)
6. **Write file** — create `.md` with frontmatter + prose body (or append rule to `rules.md`)
7. **Regenerate index** — run Index Generation Procedure
8. **Report** — "✓ Added `<type>` `<name>` to `<path>`; Index regenerated (N concepts, ...)"

---

## Subcommand: sync

1. **Check domain exists** — error if `openspec/domain/` not found
2. **Snapshot current index** — read existing `_index.yaml` to compare; if missing/malformed, treat as empty and warn
3. **Run index generation** — execute Index Generation Procedure
4. **Compare before/after** — count added, updated, removed entries
5. **Report** — list changes and final counts; if no changes: "Index is already up to date"

---

## Edge Cases

- **No domain directory on add/sync**: error "No domain KB found. Run `/opsx:domain init` first."
- **Empty static/ subdirectories**: emit empty arrays. No error.
- **Malformed `_index.yaml`**: warn and regenerate from source files; never abort.
- **Malformed frontmatter in concept file**: warn and skip that file; continue with valid files.
- **`status: superseded` without `superseded_by`**: warn but still include entry with `[SUPERSEDED]` label.
- **Large domain (100+ concepts)**: warn about token cost when adding would push past ~100 entries.
