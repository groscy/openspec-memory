---
name: Change
type: concept
aliases: [change-proposal]
related: [Artifact, Spec, Schema]
states: [in-progress, complete, archived]
rules: [tasks-required-before-apply]
actors: [Developer]
---
A Change is the core unit of work in OpenSpec — a named directory containing all artifacts (proposal, design, specs, tasks) that describe and implement a single coherent improvement to a project.

A change begins as `in-progress` when created via `/opsx:propose`, becomes `complete` when all tasks are done, and is `archived` by `/opsx:archive`. The change directory lives at `openspec/changes/<name>/` and is moved to `openspec/changes/archive/YYYY-MM-DD-<name>/` on archive.

Each change belongs to exactly one Schema, which defines which artifact types it contains and in what order they must be created. The tasks artifact must be complete before implementation can start.
