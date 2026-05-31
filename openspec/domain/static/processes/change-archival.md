---
name: ChangeArchival
type: process
aliases: [archive]
related: [Change, Spec, Discovery]
actors: [Developer]
rules: [index-never-hand-edited, discoveries-default-active]
---
ChangeArchival is the workflow for finalizing a completed Change, triggered by `/opsx:archive`.

Steps: (1) Check artifact completion; (2) Check task completion; (3) Domain review — load index, surface discovery candidates, capture confirmed ones, flag superseded discoveries, regenerate index; (4) Delta spec sync — compare each delta spec to its main spec, prompt to sync if differences exist; (5) Move change directory to `openspec/changes/archive/YYYY-MM-DD-<name>/`.

The domain review step is skipped silently when `openspec/domain/` does not exist. Index regeneration is always the last step of the domain review, ensuring `_index.yaml` is current before the next change begins.
