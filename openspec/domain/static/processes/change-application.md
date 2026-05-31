---
name: ChangeApplication
type: process
aliases: [apply]
related: [Change, Artifact, Skill]
actors: [Developer]
rules: [tasks-required-before-apply, domain-loaded-on-entry, concept-files-by-name-only]
---
ChangeApplication is the workflow for implementing a Change's tasks one by one, triggered by `/opsx:apply`.

Steps: (1) Load domain index + full concept files for concepts in task scope; (2) Read all context files (proposal, design, specs, tasks); (3) Implement each pending task in order, keeping changes minimal and focused; (4) Mark task complete (`- [ ]` → `- [x]`) immediately after finishing; (5) Continue until all tasks done or a blocker is encountered; (6) Report progress and suggest archive when complete.

The process uses canonical domain terminology when naming code identifiers and references domain states and rules when implementing logic that touches indexed concepts.
