# AGENTS.md

This file provides guidance to AI when working with code in this repository.

## Project

<!-- TODO: describe what this project does and why it exists -->

See `CONTEXT.md` for the full domain vocabulary.

## General rules

Never `git push` unless explicitly told to.

When starting brand-new work — a new OpenSpec change or any fresh implementation — create a new branch with a fitting name (`git checkout -b <name>`), then run `git fetch origin && git rebase origin/main` to ensure it is up to date with main. For continuing work on an existing branch, just rebase.

### Commit and review workflow

Each OpenSpec step produces one commit. After each step commit, run the `code-review` skill, fix any findings in a single `fix: review finding — <short description>` commit, then move on. Fix commits at this stage do not trigger another review.

Any other `code-review` run — end of OpenSpec, user-triggered, or ad-hoc — uses the full branch diff (`git diff origin/main`) and loops: review → fix (`fix: review finding`) → review again, until no inline-fixable findings remain. Filed-only findings do not keep the loop running.

Each PR must have exactly one commit and must target `main`. Before creating a PR, confirm with the user that they want to squash, then squash all commits into one: `git reset --soft origin/main`, recommit with the final message. If the branch contains multiple unrelated commits, first fetch and check whether any earlier PRs have already merged into `main` (`git fetch origin && git log origin/main..HEAD`) — the branch may look clean once main is up to date. Only if genuinely unrelated commits remain should you create a separate PR for each by cherry-picking onto a fresh branch from `main`.

When asked to "automerge": fetch origin, check `git log origin/main..HEAD` and open PRs (`gh pr list`) to understand the current state. If the commit implements an OpenSpec change, archive it first (`/opsx:archive`) and amend the commit to include the archived change. Then create a PR for the latest commit and enable automerge (`gh pr merge --auto --rebase`).

## Commands

```bash
# TODO: add commands to run, test, lint, and type-check the project
```

<!-- TODO: add any one-time setup steps (e.g. pre-commit install) -->

## Architecture

<!-- TODO: describe the high-level structure — key files, packages, entry points, and where tests live -->
