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

- [OpenSpec CLI](https://github.com/fission-ai/openspec) v1.3.1 or later
- Node.js v18 or later (required by the OpenSpec CLI)
- One of: [Claude Code](https://claude.ai/code) or GitHub Copilot Chat

## Installation

### 1. Install the OpenSpec CLI

```bash
npm install -g @fission-ai/openspec
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

### 4. Set up for your AI environment

#### Claude Code

Run the domain init command once to scaffold the domain KB and configure the OpenSpec skills:

```
/opsx:domain init
```

This will interview you about your domain, create all domain KB files, and write the skill adapter files to `.claude/commands/opsx/`. After init, the slash commands are active:

```
/opsx:domain   /opsx:propose   /opsx:apply   /opsx:explore   /opsx:archive
```

#### GitHub Copilot Chat

The `.github/copilot-instructions/` directory in this repo contains ready-to-use instruction files for each skill. Copy it to your project:

```bash
cp -r .github/copilot-instructions <your-project>/.github/
```

Then invoke skills in Copilot Chat by starting a message with the trigger, e.g.:

```
/opsx:propose add user authentication
/opsx:apply
/opsx:domain init
```

Copilot will load the matching instruction file and follow the workflow.

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
├── .github/
│   └── copilot-instructions/  # Copilot Chat adapter files (copy to your project)
│       ├── apply.md
│       ├── archive.md
│       ├── domain.md
│       ├── explore.md
│       └── propose.md
└── .claude/
    └── commands/opsx/         # Written by /opsx:domain init (gitignored)
```

> **Note**: `.github/copilot-instructions/` contains committed adapter files for Copilot Chat — copy them to your project. `.claude/commands/opsx/` is written by `/opsx:domain init` and is gitignored. The canonical skill content for both lives in `openspec/skills/<name>/SKILL.md`.

## How domain awareness works

Skills follow a strict load order to keep token costs low:

1. **`_index.yaml` always** — loaded on every skill entry when `openspec/domain/` exists (~600–800 tokens)
2. **Concept files by name** — loaded only when a concept name appears in the current change or task scope
3. **Prose body only in apply** — `propose` and `explore` read frontmatter only; `apply` reads the full file

If `openspec/domain/` does not exist, all skills proceed without domain context and without error. Zero setup friction for projects that don't need it.

## Token budget

A 30-concept domain fits in approximately 600–800 tokens as YAML (the index). Individual concept files add ~200–400 tokens each when loaded. The index is loaded on every skill invocation; concept files are loaded on demand.

When the index grows past ~100 entries, `domain add` will warn you to consider archiving inactive concepts.
