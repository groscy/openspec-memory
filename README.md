# openspec-memory

An [OpenSpec](https://github.com/fission-ai/openspec) project workspace that extends the core workflow with a **domain knowledge layer** — a structured, file-based knowledge base that gives every OpenSpec skill a shared vocabulary of concepts, processes, and rules to reason from.

## What it adds

By default OpenSpec skills have no structured understanding of your project's domain. Proposals use imprecise language, specs miss domain constraints, and implementations name things inconsistently. The domain knowledge layer fixes this by introducing:

| Addition | Description |
|---|---|
| `openspec/domain/` | Convention-based directory — its presence activates domain awareness in all skills |
| `openspec/domain/_index.yaml` | Auto-generated compact YAML index loaded by every skill on entry (~600–800 tokens) |
| `openspec/domain/static/` | Curated concepts, processes, glossary terms, and rules authored before or during development |
| `openspec/domain/discovered/` | Facts surfaced during development, with provenance and lifecycle status (`active \| superseded \| invalidated`) |
| `/opsx:domain` skill | Scaffolds the domain KB (`init`), adds individual entries (`add`), and resyncs the index (`sync`) |

Existing skills are extended to load the domain index on entry and use canonical terminology in their output:

- `/opsx:explore` — loads `_index.yaml`, offers to pull concept files as they appear in conversation
- `/opsx:propose` — loads `_index.yaml`, scans change description for concept names, flags rule conflicts before finalizing
- `/opsx:apply` — loads `_index.yaml`, reads full concept files (frontmatter + body) for concepts in task scope
- `/opsx:archive` — reviews completed changes for domain discoveries, captures confirmed ones, detects contradictions with active discoveries, regenerates `_index.yaml`

## Requirements

- [Claude Code](https://claude.ai/code) — CLI or desktop app
- [OpenSpec CLI](https://github.com/fission-ai/openspec) v1.3.1 or later
- Node.js v18 or later (required by the OpenSpec CLI)

## Installation

### 1. Install the OpenSpec CLI

```bash
npm install -g @fission-ai/openspec
```

Verify:

```bash
openspec --version
```

### 2. Clone this repository

```bash
git clone <repo-url> openspec-memory
cd openspec-memory
```

### 3. Initialize OpenSpec in your project (if starting fresh)

The `openspec/` directory is already present in this repo. If you are adopting these skills into a different project, run:

```bash
openspec init
```

### 4. Install skills for your environment

Skills are stored in `openspec/skills/` and must be installed for your AI environment before first use. Run:

```bash
openspec install skills
```

This generates adapter files for all supported environments. To target a specific environment:

```bash
openspec install skills --env claude      # Claude Code
openspec install skills --env copilot     # GitHub Copilot Chat
openspec install skills --env intellij    # JetBrains IntelliJ OpenSpec plugin
openspec install skills --env all         # all of the above (default)
```

#### Claude Code

After running `openspec install skills --env claude`, the skills are available as slash commands:

```
/opsx:domain
/opsx:propose
/opsx:apply
/opsx:explore
/opsx:archive
```

Claude Code picks up `.claude/commands/opsx/` from the project root automatically. To verify, open Claude Code in this directory and run `/opsx:domain` — you should see the available subcommands.

#### GitHub Copilot Chat

After running `openspec install skills --env copilot`, adapter files are written to `.github/copilot-instructions/`. In Copilot Chat, use the trigger from the file header to invoke each skill (e.g., `/opsx:propose`). Copilot will load the skill instructions and follow the workflow.

#### JetBrains IntelliJ

If you have the OpenSpec IntelliJ plugin installed, it reads `openspec/skills/` automatically on project open and registers each skill as an IDE action. You can also run:

```bash
openspec install skills --env intellij
```

This writes `.idea/runConfigurations/opsx_<name>.xml` run configs so skills appear in the Run menu without a plugin update.

## Usage

### Initialize a domain knowledge base

Run this once per project to scaffold the domain KB from scratch:

```
/opsx:domain init
```

Claude will interview you about your domain, optionally scan your codebase, show a draft for confirmation, and write all files.

### Add individual entries

```
/opsx:domain add concept Invoice
/opsx:domain add process InvoiceApproval
/opsx:domain add rule no-duplicate-invoice-numbers
/opsx:domain add actor AccountsPayable
```

### Sync the index after direct edits

If you edit concept files directly, regenerate `_index.yaml`:

```
/opsx:domain sync
```

### Normal OpenSpec workflow

The domain KB integrates transparently into the standard workflow:

```
/opsx:propose   → creates a change with domain-aware artifacts
/opsx:apply     → implements tasks using canonical domain terminology
/opsx:archive   → captures domain discoveries, updates the index
```

## Project structure

```
openspec-memory/
├── openspec/
│   ├── config.yaml            # Schema: spec-driven
│   ├── skills/                # Canonical skill source (tracked in git)
│   │   ├── README.md          # Layout and frontmatter schema reference
│   │   ├── apply/SKILL.md
│   │   ├── archive/SKILL.md
│   │   ├── domain/SKILL.md
│   │   ├── explore/SKILL.md
│   │   └── propose/SKILL.md
│   ├── specs/                 # Capability specs
│   │   ├── domain-awareness/
│   │   ├── domain-discovered-knowledge/
│   │   ├── domain-index/
│   │   ├── domain-skill/
│   │   └── domain-static-knowledge/
│   └── domain/                # Domain KB for the OpenSpec domain itself
│       ├── _index.yaml
│       ├── static/
│       │   ├── concepts/      # Change, Artifact, Spec, Skill, Schema, Discovery
│       │   ├── processes/     # ChangeProposal, ChangeApplication, ChangeArchival
│       │   ├── glossary.md
│       │   └── rules.md
│       └── discovered/
└── .claude/
    └── commands/opsx/         # Generated Claude adapters (run: openspec install skills)
```

> **Note**: `.claude/commands/opsx/` is generated output. The canonical skill source is `openspec/skills/`. Run `openspec install skills` to regenerate adapters after modifying skills.

## How domain awareness works

Skills follow a strict load order to keep token costs low:

1. **`_index.yaml` always** — loaded on every skill entry when `openspec/domain/` exists (~600–800 tokens)
2. **Concept files by name** — loaded only when a concept name appears in the current change or task scope
3. **Prose body only in apply** — `propose` and `explore` read frontmatter only; `apply` reads the full file

If `openspec/domain/` does not exist, all skills proceed without domain context and without error. Zero setup friction for projects that don't need it.

## Token budget

A 30-concept domain fits in approximately 600–800 tokens as YAML (the index). Individual concept files add ~200–400 tokens each when loaded. The index is loaded on every skill invocation; concept files are loaded on demand.

When the index grows past ~100 entries, `domain add` will warn you to consider archiving inactive concepts.
