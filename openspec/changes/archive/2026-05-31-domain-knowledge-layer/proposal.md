## Why

OpenSpec skills operate without any structured understanding of the project's domain — concepts, terminology, processes, and business rules live only in the developer's head or scattered across code. This causes proposals to use imprecise language, specs to miss domain constraints, and implementations to name things inconsistently. A domain knowledge layer gives every skill a shared, token-efficient vocabulary to reason from.

## What Changes

- New `openspec/domain/` directory convention — its presence enables domain awareness across all skills
- New `openspec/domain/_index.yaml` — auto-generated compact index, always loaded by skills
- New `openspec/domain/static/` — curated concepts, processes, glossary, and rules authored before or during development
- New `openspec/domain/discovered/` — facts surfaced during development, with provenance and lifecycle status
- New `/opsx:domain` skill — scaffolds static knowledge (`init`), adds individual entries (`add`), and manually syncs the index (`sync`)
- `/opsx:archive` extended — reviews completed changes for domain discoveries, captures confirmed ones, regenerates `_index.yaml` before archiving
- `/opsx:explore`, `/opsx:propose`, `/opsx:apply` extended — load `_index.yaml` on entry, load specific concept files by name when relevant to current change scope

## Capabilities

### New Capabilities

- `domain-index`: Compact YAML index of all domain knowledge entries, auto-generated from frontmatter across static and discovered files. Always loaded by skills; never hand-edited.
- `domain-static-knowledge`: Curated, stable domain entries — concepts, processes, glossary terms, and rules — stored as markdown files with structured frontmatter and prose body.
- `domain-discovered-knowledge`: Facts found during development, each with provenance (which change surfaced it), lifecycle status (`active | superseded | invalidated`), and links to related static concepts.
- `domain-skill`: The `/opsx:domain` skill for initializing, adding to, and syncing the domain knowledge base.
- `domain-awareness`: The protocol used by existing skills to load and apply domain knowledge — index always, concept files by name, frontmatter before body.

### Modified Capabilities

## Impact

- New directory: `openspec/domain/` (no changes to existing `openspec/changes/` or `openspec/specs/` structure)
- New skill file: `opsx:domain`
- Modified skill files: `opsx:archive`, `opsx:explore`, `opsx:propose`, `opsx:apply`
- No breaking changes to existing changes or spec files
- Token overhead per interaction: ~600-800 tokens (index only) when domain/ exists; zero when it does not
