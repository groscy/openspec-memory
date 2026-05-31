---
name: Spec
type: concept
aliases: [capability-spec, delta-spec]
related: [Change, Capability]
---
A Spec is a capability specification containing requirements and BDD-style scenarios (WHEN/THEN). Specs live in two locations: main specs at `openspec/specs/<capability>/spec.md` (the authoritative record) and delta specs at `openspec/changes/<name>/specs/<capability>/spec.md` (the change-specific additions or modifications).

Delta specs are compared against main specs during `/opsx:archive`. If differences exist, the user is prompted to sync — which copies or merges the delta spec content into the main spec. After archiving, the main spec reflects the full accumulated state of all past changes.

A spec file contains requirements using `### Requirement: <name>` headings and scenarios using `#### Scenario: <name>` headings with WHEN/THEN/AND structure.
