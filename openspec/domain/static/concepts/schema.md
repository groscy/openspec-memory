---
name: Schema
type: concept
aliases: [workflow-schema]
related: [Artifact, Change]
---
A Schema defines the workflow type for a Change — it specifies which Artifact types exist, their dependencies, templates, and which artifacts must be complete before implementation can start (`applyRequires`).

The current schema is `spec-driven`, which requires: proposal → design + specs (parallel) → tasks. The `tasks` artifact is in `applyRequires`, meaning it must be `done` before `/opsx:apply` can proceed.

Schema configuration lives in the OpenSpec package at `schemas/<name>/schema.yaml` and its artifact templates at `schemas/<name>/templates/`. Projects select a schema via `openspec/config.yaml` (`schema: spec-driven`). Skills query the schema via `openspec status --json` and `openspec instructions <artifact> --json`.
