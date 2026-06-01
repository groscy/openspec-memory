---
name: domain
description: "Manage the OpenSpec domain knowledge base - initialize, add entries, and sync the index"
triggers:
  - /opsx:domain
agents: [claude, copilot, intellij]
args: "<init|add <type> <name>|sync>"
version: "1.0.0"
---

Manage the OpenSpec domain knowledge base.

**Input**: A subcommand after `/opsx:domain`:
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

### Purchase Order
A buyer-issued document authorizing a seller to deliver goods at an agreed price.
```

One `### Term` heading per entry. The line immediately after the heading is the one-line definition.

### rules.md Format

```markdown
### Rule: approved-before-payment
An invoice must be approved before payment can be recorded.

### Rule: no-duplicate-invoice-numbers
Invoice numbers must be unique within a billing period.
```

One `### Rule: <id>` heading per entry. The line immediately after is the one-line constraint.

### _index.yaml Schema

```yaml
concepts:
  - name: Invoice
    aliases: [bill, billing-doc]
    related: [Customer, Payment]
    states: [draft, pending, approved, paid]    # omit if absent in frontmatter
    rules: [approved-before-payment]            # omit if absent in frontmatter
    actors: []                                  # omit if absent in frontmatter
    def: One-line definition (first sentence of prose body)
    file: static/concepts/invoice.md
processes:
  - name: InvoiceApproval
    related: [Invoice, Approver]
    def: Multi-step flow for approving invoices above threshold
    file: static/processes/invoice-approval.md
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

Top-level keys: `concepts`, `processes`, `glossary`, `rules`, `discoveries`.
Each key maps to an array; emit empty arrays (`[]`) when no entries exist.

---

## Index Generation Procedure

Use this procedure whenever a `sync` is needed (called by `sync`, `init`, and `add`):

1. **Scan concept files**: read all `*.md` files under `openspec/domain/static/concepts/`
2. **Scan process files**: read all `*.md` files under `openspec/domain/static/processes/`
3. **Parse glossary**: read `openspec/domain/static/glossary.md`; extract each `### <Term>` heading (term = heading text) and the first non-blank line following it (def)
4. **Parse rules**: read `openspec/domain/static/rules.md`; extract each `### Rule: <id>` heading (id = text after "Rule: ") and the first non-blank line following it (def)
5. **Scan discovery files**: read all `*.md` files under `openspec/domain/discovered/`
6. For each scanned `.md` file: extract the YAML frontmatter block (between `---` delimiters)
7. **Validate** each file (see Validation section below); skip files that fail, warn the user
8. **Build index entries**:
   - `def` field: use the first sentence of the prose body if non-empty, otherwise synthesize from the name
   - For discoveries with `status: superseded`: append `[SUPERSEDED]` to def
   - For discoveries with `status: invalidated`: append `[INVALIDATED]` to def
9. **Write** `openspec/domain/_index.yaml` with all four top-level keys
10. If any category has no valid entries, emit `key: []` (never omit the key)
11. If `glossary.md` or `rules.md` do not exist, emit `glossary: []` and `rules: []`

### Validation Rules

For concept/process files — required fields: `name`, `type`, `aliases`, `related`
- Missing any required field → warn `⚠ Skipping <file>: missing required field '<field>'` and skip
- Invalid YAML frontmatter → warn `⚠ Skipping <file>: malformed frontmatter` and skip

For discovery files — required fields: `name`, `type`, `discovered_in`, `date`, `relates_to`, `status`
- Missing any required field → warn and skip
- `status` not in `[active, superseded, invalidated]` → warn `⚠ Skipping <file>: invalid status '<value>'` and skip
- `status: superseded` with null/missing `superseded_by` → warn but still include with `[SUPERSEDED]` label

After generation: report "Index written: N concepts, M processes, P glossary terms, Q rules, R discoveries"

---

## Subcommand: init

Scaffold the full domain KB from scratch.

### Step 1: Check for existing domain

Check if `openspec/domain/` already exists.

- **If yes**: warn "Domain KB already exists at `openspec/domain/`." Ask: "Reinitialize (overwrites existing files) or cancel?" Stop if user cancels.
- **If no**: proceed.

### Step 2: Interview the user

Ask:
> "Describe your project's domain in plain language. What are the main concepts, processes, terminology, and business rules? (e.g., 'An invoicing system where customers submit purchase orders, finance approves them, and payments are tracked')"

From their description, extract:
- **Concepts**: named domain entities (nouns — Invoice, Customer, Order, Payment)
- **Processes**: multi-step workflows (verb phrases — invoice approval, order fulfillment)
- **Glossary terms**: domain-specific terminology needing definition
- **Rules**: business constraints and invariants
- **Actors**: people or systems involved (Finance, AccountsPayable, ExternalAPI)

Ask follow-up questions as needed to fill in aliases, states, and relationships.

### Step 3: Optional codebase read

Check for source code directories (`src/`, `lib/`, `app/`, `models/`, `schemas/`, `routes/`).

- **If found**: offer "I found source files — scan them to enrich the draft concepts?"
  - If yes: read relevant model/schema/route files; infer additional concepts, states, and relationships; merge with interview-derived content
  - If no: proceed with interview content only
- **If not found**: skip this step.

### Step 4: Show draft and confirm

Display a formatted draft summary:

```
## Domain KB Draft

**Concepts** (N):
- Invoice: aliases [bill], states [draft, pending, approved, paid], related [Customer, Payment]
- Customer: aliases [], related [Invoice]

**Processes** (N):
- InvoiceApproval: related [Invoice, Approver, Finance]

**Glossary** (N):
- Net Terms: Payment period after invoice date
- Purchase Order: Buyer-issued authorization for a seller to deliver goods

**Rules** (N):
- approved-before-payment: An invoice must be approved before payment can be recorded

**Actors captured as concepts**: Finance, AccountsPayable
```

Ask: "Does this look right? Confirm to create files, or describe any revisions."

Incorporate any revisions and show the draft again. Only write files after explicit confirmation.

### Step 5: Create directory structure

Create these directories (create parent dirs as needed):
- `openspec/domain/`
- `openspec/domain/static/`
- `openspec/domain/static/concepts/`
- `openspec/domain/static/processes/`
- `openspec/domain/discovered/`

### Step 6: Write static files

**`openspec/domain/static/glossary.md`** — one `### Term` + definition per glossary entry:

```markdown
### Net Terms
Payment period after invoice date (e.g., Net 30).
```

**`openspec/domain/static/rules.md`** — one `### Rule: <id>` + constraint per rule:

```markdown
### Rule: approved-before-payment
An invoice must be approved before payment can be recorded.
```

**Concept files** at `openspec/domain/static/concepts/<kebab-name>.md`:

```markdown
---
name: Invoice
type: concept
aliases: [bill, billing-doc]
related: [Customer, Payment]
states: [draft, pending, approved, paid]
rules: [approved-before-payment]
---
An Invoice is a financial document issued to a customer requesting payment for goods or services rendered.

It belongs to a single Customer and must be approved before payment is recorded. Valid states are: draft, pending, approved, paid. A disputed state may occur before resolution.
```

**Process files** at `openspec/domain/static/processes/<kebab-name>.md` (same structure, `type: process`).

**Actor files** at `openspec/domain/static/concepts/<kebab-name>.md` (same structure, `type: actor`).

### Step 7: Generate index

Run the Index Generation Procedure and write `openspec/domain/_index.yaml`.

### Step 8: Configure OpenSpec skills

Check if `openspec/skills/` exists in the project.

- **If yes**: for each directory `openspec/skills/<name>/` containing a `SKILL.md`:
  1. Read the SKILL.md file
  2. Parse the frontmatter block to extract `name`, `description`, and the first entry in `triggers`
  3. Extract the prose body (everything after the closing `---` of the frontmatter)
  4. Write `.claude/commands/opsx/<name>.md` with this content:
     ```markdown
     ---
     description: "<description from frontmatter>"
     ---
     <!-- Configured by /opsx:domain init — source: openspec/skills/<name>/SKILL.md -->
     <!-- Trigger: <trigger> -->

     <prose body>
     ```
  5. Skip any SKILL.md that fails frontmatter validation; warn and continue

- **If no**: skip this step silently.

### Step 9: Report

```
## Domain KB Initialized

Created:
- N concept files in openspec/domain/static/concepts/
- M process files in openspec/domain/static/processes/
- openspec/domain/static/glossary.md (P terms)
- openspec/domain/static/rules.md (Q rules)
- openspec/domain/_index.yaml (auto-generated index)

Skills configured:
- .claude/commands/opsx/apply.md
- .claude/commands/opsx/archive.md
- .claude/commands/opsx/domain.md
- .claude/commands/opsx/explore.md
- .claude/commands/opsx/propose.md

All OpenSpec skills will now load domain context automatically.
Next: /opsx:domain add concept <name>  to add more entries
      /opsx:domain sync                to regenerate the index after direct edits
```

If no `openspec/skills/` directory was found, omit the "Skills configured" section.

---

## Subcommand: add

Add a single entry to the domain KB and update the index.

**Usage**: `/opsx:domain add <type> <name>`
Where `<type>` is: `concept`, `process`, `rule`, or `actor`

### Step 1: Parse and validate arguments

Extract `<type>` and `<name>` from input. If either is missing, ask for them.
Normalize `<name>` to kebab-case for filename, PascalCase for `name` frontmatter field.

### Step 2: Check for domain directory

If `openspec/domain/` does not exist: error "No domain KB found. Run `/opsx:domain init` first." Stop.

### Step 3: Check for duplicates

Load `openspec/domain/_index.yaml` (if it exists).
Check if any entry across `concepts`, `processes`, `rules`, `discoveries` has the same name (case-insensitive).

- **Duplicate found**: warn "⚠ An entry named `<name>` already exists (type: <existing-type>)."
  Ask: "Update the existing entry, or cancel?"
  - Update: proceed using the existing file path; interview for revised content
  - Cancel: stop

### Step 4: Determine target file path

- `concept` → `openspec/domain/static/concepts/<kebab-name>.md`
- `process` → `openspec/domain/static/processes/<kebab-name>.md`
- `actor` → `openspec/domain/static/concepts/<kebab-name>.md` (with `type: actor`)
- `rule` → append to `openspec/domain/static/rules.md` (no separate file)

### Step 5: Interview for content

Ask targeted questions based on type:

- **concept/actor**: What does it represent? What are its aliases? What concepts is it related to? What are its valid states (if stateful)? What rules apply to it?
- **process**: What steps does it involve? Which concepts does it operate on? Which actors participate?
- **rule**: What is the constraint in one sentence? What does it enforce or prevent?

### Step 6: Write the file

For `concept`, `process`, `actor`: write a new `.md` file with frontmatter and prose body.
For `rule`: append `### Rule: <id>` + one-line constraint to `openspec/domain/static/rules.md`.

### Step 7: Regenerate index

Run the Index Generation Procedure and rewrite `openspec/domain/_index.yaml`.

### Step 8: Report

```
✓ Added concept `Invoice` to openspec/domain/static/concepts/invoice.md
✓ Index regenerated (N concepts, M processes, P glossary terms, Q rules, R discoveries)
```

---

## Subcommand: sync

Regenerate `_index.yaml` from all current frontmatter and report what changed.

### Step 1: Check for domain directory

If `openspec/domain/` does not exist: error "No domain KB found. Run `/opsx:domain init` first." Stop.

### Step 2: Snapshot current index

Read `openspec/domain/_index.yaml` if it exists. Record all entry names as the "before" set.
If the file is missing or malformed: treat "before" as empty; warn "⚠ Previous index missing or malformed — regenerating from source files."

### Step 3: Run index generation

Execute the Index Generation Procedure.

### Step 4: Compare before/after

Count entries added (new names), updated (same name, different content), and removed (names gone from current files).

### Step 5: Report

```
## Domain Index Synced

Changes:
- Added: 2 (Invoice, Customer)
- Updated: 1 (approved-before-payment)
- Removed: 0

Index now contains: N concepts, M processes, P glossary terms, Q rules, R discoveries
```

If no changes: "Index is already up to date. (N concepts, M processes, P terms, Q rules, R discoveries)"

---

## Edge Cases

- **No domain directory on any subcommand except `init`**: error "No domain KB found. Run `/opsx:domain init` first."
- **Empty `static/` subdirectories**: produce index with empty arrays for that category. No error.
- **Malformed `_index.yaml`**: warn and regenerate from source files; never abort.
- **Malformed frontmatter in a concept file**: warn "⚠ Skipping `<file>`: malformed frontmatter" and continue with valid files.
- **Missing optional fields**: include only present optional fields in the index entry.
- **`status: superseded` without `superseded_by`**: warn and include the entry with `[SUPERSEDED]` label.
- **Large domain (100+ concepts)**: when `domain add` would push the index past approximately 100 entries, warn "⚠ Index is getting large (~N entries). Consider archiving inactive concepts to keep token cost low."
