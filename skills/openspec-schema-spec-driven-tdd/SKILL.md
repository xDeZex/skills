---
name: openspec-schema-spec-driven-tdd
description: Install the spec-driven-tdd OpenSpec schema into the current project. Use when setting up a new project to use the TDD-focused OpenSpec workflow (RED/GREEN per task, COMMIT and REVIEW per group and full branch, no refactor step).
license: MIT
metadata:
  author: openspec
  version: "1.0"
---

Install the `spec-driven-tdd` OpenSpec schema into the current project.

**What this schema is**: A TDD-focused variant of the standard `spec-driven` workflow. The apply phase runs RED → GREEN per code task (skipped for non-code work), COMMITs once per group as a checkpoint, then REVIEWs per group and again across the full branch (no refactor step).

**Prerequisite**: `openspec init` must have been run in this project already.

**Steps**

1. Check whether `openspec/schemas/spec-driven-tdd/schema.yaml` already exists in the current project.
   - If it does, confirm with the user before overwriting.

2. Copy the schema file from your skill directory, replacing the existing one:
   ```
   openspec/schemas/spec-driven-tdd/schema.yaml  ←  references/schema.yaml
   ```
   Use `cp` (or `Copy-Item` on Windows). Do not inline the content.

3. Confirm:
   ```
   ✓ Schema installed at openspec/schemas/spec-driven-tdd/schema.yaml
   ```
