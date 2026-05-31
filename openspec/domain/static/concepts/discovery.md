---
name: Discovery
type: concept
aliases: []
related: [Change, Spec]
states: [active, superseded, invalidated]
rules: [discoveries-default-active]
---
A Discovery is a domain fact surfaced during development — a behavior, constraint, or relationship that was implicit in the implementation but not previously documented in the domain KB. Discoveries are captured during `/opsx:archive` and stored as individual files under `openspec/domain/discovered/`.

Each discovery has provenance (`discovered_in`: the change name that surfaced it) and a lifecycle `status`: `active` (current and valid), `superseded` (replaced by a later change, with `superseded_by` set), or `invalidated` (found to be incorrect). Discoveries start `active` and may transition as later changes contradict or refine them.

The domain index includes all discoveries with their status labels — superseded entries show `[SUPERSEDED]` in their summary so skills can de-weight them.
