---
name: ChangeProposal
type: process
aliases: [propose]
related: [Change, Artifact, Schema, Developer]
actors: [Developer]
---
ChangeProposal is the workflow for creating a new Change with all required Artifacts in one step, triggered by `/opsx:propose`.

Steps: (1) User describes what they want to build; (2) Skill derives a kebab-case change name; (3) `openspec new change <name>` scaffolds the change directory; (4) Artifacts are created in dependency order — each artifact's instructions come from `openspec instructions <id> --json`; (5) Final status is shown and user is prompted to run `/opsx:apply`.

The process reads domain context (index + matching concept frontmatter) before generating artifacts, and flags rule conflicts before finalizing the proposal.
