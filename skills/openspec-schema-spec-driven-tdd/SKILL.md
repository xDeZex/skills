---
name: openspec-schema-spec-driven-tdd
description: Install the spec-driven-tdd OpenSpec schema into the current project. Use when setting up a new project to use the TDD-focused OpenSpec workflow.
license: MIT
metadata:
  author: openspec
  version: "1.0"
---

**What this schema is**: A TDD-focused variant of the standard `spec-driven` workflow. The apply phase runs RED → GREEN per code task (skipped for non-code work), COMMITs once per group as a checkpoint, then REVIEWs per group and again across the full branch (no refactor step).

**Prerequisite**: `openspec init` must have been run in this project already.

**Steps**

1. Check whether `openspec/schemas/spec-driven-tdd/` already exists in the current project.
   - If it does, confirm with the user before overwriting.

2. Copy the schema file and templates from your skill directory, replacing any existing ones:
   ```
   openspec/schemas/spec-driven-tdd/schema.yaml           ←  references/schema.yaml
   openspec/schemas/spec-driven-tdd/templates/design.md   ←  references/templates/design.md
   openspec/schemas/spec-driven-tdd/templates/proposal.md ←  references/templates/proposal.md
   openspec/schemas/spec-driven-tdd/templates/spec.md     ←  references/templates/spec.md
   openspec/schemas/spec-driven-tdd/templates/tasks.md    ←  references/templates/tasks.md
   ```
   Use `cp` (or `Copy-Item` on Windows). Do not inline the content.

3. Point the project at the new schema: in `openspec/config.yaml`, set the `schema:` line to `schema: spec-driven-tdd`. Edit that line only — preserve any existing `context:`/`rules:` content.

4. Confirm:
   ```
   ✓ Schema installed at openspec/schemas/spec-driven-tdd/
   ✓ openspec/config.yaml now points at spec-driven-tdd
   ```
