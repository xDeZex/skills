---
name: code-review
description: Spawn a focused code review subagent. Use after every RED/GREEN cycle (pass the files changed in that cycle), after a commit, or whenever the user asks for a review. Always use this skill for reviews — it ensures the review lens is test-first and findings are fix-oriented.
---

Spawn a code review subagent with the diff and let it gather its own context.

## Determine scope

- **Per-behavior cycle** (during openspec apply): the files changed in this RED/GREEN cycle only — not the full diff
- **Final review** (end of all OpenSpec tasks, user-triggered, or any ad-hoc request): the full branch diff against main

## Steps

1. **Get changed file names — nothing else**
   - Per-cycle: `git show --name-only HEAD` — file names only, no diff content
   - Final review: `git diff --name-only origin/main` — unless the user pointed at something more specific (a file path, commit ref, or range)

2. **For per-cycle reviews, gather task context before spawning:**
   - Read `openspec/changes/<name>/tasks.md` and identify the current in-progress task (task number + full description)
   - Read the delta spec for the matching capability (`openspec/changes/<name>/specs/<capability>/spec.md`) and extract the WHEN/THEN scenarios in scope for that task
   - This context is passed inline to the subagent — do not leave it for the subagent to gather

3. **Spawn a subagent** and pass it:
   - The changed file names
   - The scope: per-cycle or final review
   - Whether an active OpenSpec change exists (run `openspec list --json` only if you don't already know)
   - The active change name, if any
   - Per-cycle only: the current task number, description, and the WHEN/THEN scenarios in scope

   The subagent handles diff retrieval and any additional reading it needs.

   **Per-cycle subagent instructions:**
   > You are doing a per-cycle code review. Read `references/reviewer-light.md` from your skill directory for the review lens.
   >
   > Run `git show --unified=30 HEAD -- <files>` on the changed files listed below to get the full diff with context. Use that diff as your primary source — do not rely on reading file state to reconstruct what changed.
   >
   > The current task and its scenarios are provided below — review against these, do not re-read tasks.md or spec files.
   >
   > Skip any `openspec/changes/**` entries in the file list — those are already provided as structured context.
   >
   > Undeclared impact (code touches something not listed in the proposal) is itself a finding.
   >
   > Model: haiku

   **Final review subagent instructions:**
   > You are doing a final code review. Read `references/reviewer.md` from your skill directory for the review lens.
   >
   > Below is a list of changed files. Read each changed file in full before reviewing.
   >
   > If an active OpenSpec change is named, gather context yourself:
   > - Read `openspec/changes/<name>/proposal.md` and extract capability names under **New Capabilities** and **Modified Capabilities**
   > - For each capability, read the global spec (`openspec/specs/<capability>/spec.md`) and the delta spec (`openspec/changes/<name>/specs/<capability>/spec.md`)
   > - Read `openspec/changes/<name>/tasks.md` to identify which behaviors were in scope
   > - Skip any `openspec/changes/**` entries in the listing — those files are already provided as structured context
   >
   > Undeclared impact (code touches something not listed in the proposal) is itself a finding.
   >
   > Model: sonnet

4. **Print the full report verbatim.** Output the subagent's findings as a text message before doing anything else. Do not skip ahead to fixing.

5. **Act on findings** — only after printing. The main Claude acts on findings per the project workflow in CLAUDE.md.
