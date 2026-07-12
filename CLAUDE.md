# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal collection of Claude Code skills, distributed via [skills.sh](https://skills.sh). Downstream projects install skills from here with `npx skills add xDeZex/skills`. There is no build, lint, or test tooling — this repo is markdown (skill definitions) plus a couple of reference/template files consumed by those skill definitions.

## Structure

Each skill lives in `skills/<name>/` with a required `SKILL.md` (`name` + `description` frontmatter).

Skills that need more than one file put supporting material in subfolders next to `SKILL.md`:
- `skills/code-review/agents/` — subagent prompt files the skill dispatches to
- `skills/code-review/references/` — lens/reviewer detail docs pulled in as needed
- `skills/openspec-schema-spec-driven-tdd/references/` — a `schema.yaml` and `templates/` that get installed into a consuming project's `openspec/schemas/` directory

`README.md` maintains a table of all skills with one-line descriptions — keep it in sync when adding, renaming, or removing a skill.

## Commit and PR conventions

- Conventional Commits, no scopes (see `skills/commit/SKILL.md` for the exact convention this repo follows for its own commits, including issue-trailer rules).
- Never `git push` unless explicitly told to.
