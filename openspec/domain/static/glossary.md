### Delta spec
A spec file inside `openspec/changes/<name>/specs/` that differs from or extends the corresponding main spec at `openspec/specs/<capability>/spec.md`.

### Frontmatter
YAML metadata block delimited by `---` at the top of a markdown file; parsed by skills to build the domain index and validate file structure.

### Capability
A named domain area used as the directory name under `openspec/specs/` (e.g., `domain-index`, `domain-skill`).

### Domain index
The auto-generated `openspec/domain/_index.yaml`; provides a token-efficient compact summary of all domain knowledge loaded by skills on entry.
