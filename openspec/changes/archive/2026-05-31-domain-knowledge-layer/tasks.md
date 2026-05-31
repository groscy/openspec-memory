## 1. Domain Directory and Index

- [x] 1.1 Define `_index.yaml` schema (top-level keys: `concepts`, `processes`, `glossary`, `rules`, `discoveries`)
- [x] 1.2 Write index generator: reads frontmatter from all files under `openspec/domain/static/` and `openspec/domain/discovered/`, writes `_index.yaml`
- [x] 1.3 Write index validator: warns on missing required frontmatter fields, skips malformed files
- [x] 1.4 Verify index generator handles empty domain directory without error

## 2. Static Knowledge Files

- [x] 2.1 Define required frontmatter schema for concept and process files (`name`, `type`, `aliases`, `related`)
- [x] 2.2 Define optional frontmatter fields (`states`, `rules`, `actors`)
- [x] 2.3 Define `glossary.md` format (`### Term` headings, one-line definitions)
- [x] 2.4 Define `rules.md` format (`### Rule: <id>` headings, one-line constraint)
- [x] 2.5 Write example concept file (`concepts/example.md`) to serve as template for the `add` command

## 3. Discovered Knowledge Files

- [x] 3.1 Define required frontmatter schema for discovery files (`name`, `type: discovery`, `discovered_in`, `date`, `relates_to`, `status`)
- [x] 3.2 Add `superseded_by` as optional field and validate it when `status: superseded`
- [x] 3.3 Verify index generator includes discovery status labels (`[SUPERSEDED]`, `[INVALIDATED]`) in index entries

## 4. /opsx:domain Skill

- [x] 4.1 Create skill file `opsx:domain` with subcommand routing (`init`, `add`, `sync`)
- [x] 4.2 Implement `init`: conversational interview flow (ask for domain description, extract concepts/processes/rules/actors)
- [x] 4.3 Implement `init`: optional codebase read (scan models/schemas/routes if present, enrich draft concepts)
- [x] 4.4 Implement `init`: show draft summary, require user confirmation before writing files
- [x] 4.5 Implement `init`: create directory structure and write all files, then generate index
- [x] 4.6 Implement `add`: accept `concept|process|rule|actor <name>`, check for duplicates, create file, regenerate index
- [x] 4.7 Implement `sync`: regenerate `_index.yaml` from all current frontmatter, report changes (added/updated/removed count)

## 5. Archive Integration

- [x] 5.1 Extend `/opsx:archive` to read `_index.yaml` if domain exists
- [x] 5.2 Implement discovery candidate surfacing: compare change implementation against domain KB, propose specific candidates
- [x] 5.3 Implement discovery capture flow: user confirms candidate → write discovery file
- [x] 5.4 Implement contradiction detection: flag active discoveries that conflict with the completed change, offer to mark `superseded`
- [x] 5.5 Implement index regeneration as the final step before marking a change archived

## 6. Existing Skill Domain Awareness

- [x] 6.1 Extend `/opsx:explore`: load `_index.yaml` on entry, offer to read concept files when concept names appear in conversation
- [x] 6.2 Extend `/opsx:propose`: load `_index.yaml`, scan change description for concept names, load matching concept frontmatter, flag rule conflicts before finalizing proposal
- [x] 6.3 Extend `/opsx:apply`: load `_index.yaml`, load full concept files (frontmatter + body) for concepts in current task scope, use canonical terminology in output
- [x] 6.4 Verify all skills handle missing `openspec/domain/` gracefully (no error, no domain context loaded)

## 7. Validation and Edge Cases

- [x] 7.1 Test index with zero entries in each category (empty arrays, no errors)
- [x] 7.2 Test `domain add` with a name that already exists (duplicate warning)
- [x] 7.3 Test `domain init` on a project already in flight (codebase read path)
- [x] 7.4 Test archive discovery flow with no candidates (clean change, nothing to capture)
- [x] 7.5 Test skill behavior when `_index.yaml` exists but is malformed (graceful degradation)
