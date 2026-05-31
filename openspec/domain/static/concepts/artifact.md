---
name: Artifact
type: concept
aliases: []
related: [Change, Schema]
states: [pending, done]
---
An Artifact is a generated document within a Change — examples include `proposal.md`, `design.md`, `tasks.md`, and `specs/**/*.md`. Each artifact has a type ID (e.g., `proposal`, `tasks`), an output path, and a status of either `pending` or `done`.

Artifacts are created in dependency order as defined by the Schema. Some artifacts depend on others being `done` first (e.g., `tasks` depends on `design` and `specs`). The set of artifacts required before `/opsx:apply` can run is defined by the Schema's `applyRequires` list.

Skills use `openspec status --change <name> --json` to check artifact completion and `openspec instructions <id> --change <name> --json` to get creation instructions for each artifact type.
