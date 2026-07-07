---
name: code-review
description: Spawn a focused review subagent over a diff. Use after a commit or a batch of related commits, at the end of a branch, or whenever the user asks for a review — for source code or any other changed files (docs, config, specs). Always use this skill for reviews — findings come back fix-oriented, not just observations.
---

Spawn a review subagent with the diff and let it gather its own context.

## Determine scope

- **Incremental** — a commit, a range of commits, or a set of files the caller already identified as "this batch of work" (e.g. one TDD cycle, one task group). Review just that range, not the full diff.
- **Full** — end of the work, user-triggered, or any ad-hoc request. The full branch diff against main.

## Steps

1. **Get changed file names — nothing else**
   - Incremental: use the commit range or file list already identified for this batch (e.g. `git diff --name-only <parent>..HEAD`); if none was given, default to the last commit (`git show --name-only HEAD`)
   - Full: `git diff --name-only origin/main` — unless the user pointed at something more specific (a file path, commit ref, or range)

2. **For incremental reviews, gather task context before spawning, if an OpenSpec change is active:**
   - Read `openspec/changes/<name>/tasks.md` and identify the current in-progress task (task number + full description)
   - Read the delta spec for the matching capability (`openspec/changes/<name>/specs/<capability>/spec.md`) and extract the WHEN/THEN scenarios in scope for that task
   - This context is passed inline to the subagent — do not leave it for the subagent to gather

3. **Spawn a subagent** and pass it:
   - The changed file names
   - The scope: incremental or full
   - Whether an active OpenSpec change exists (run `openspec list --json` only if you don't already know)
   - The active change name, if any
   - Incremental only, if applicable: the current task number, description, and the WHEN/THEN scenarios in scope

   The subagent handles diff retrieval and any additional reading it needs.

   **Incremental subagent instructions:**
   > You are doing an incremental review. Read `references/reviewer-light.md` from your skill directory for the review lens.
   >
   > Run `git diff --unified=30 <range> -- <files>` for the range identified below (or `git show --unified=30 HEAD -- <files>` if none was given) to get the full diff with context. Use that diff as your primary source — do not rely on reading file state to reconstruct what changed.
   >
   > If task context is provided below, review against it and do not re-read tasks.md or spec files.
   >
   > Skip any `openspec/changes/**` entries in the file list — those are already provided as structured context.
   >
   > Undeclared impact (the diff touches something not accounted for elsewhere) is itself a finding.
   >
   > Model: sonnet

   **Full review subagent instructions:**
   > You are doing a full review. Read `references/reviewer.md` from your skill directory for the review lens.
   >
   > Below is a list of changed files. Read each changed file in full before reviewing.
   >
   > If an active OpenSpec change is named, gather context yourself:
   > - Read `openspec/changes/<name>/proposal.md` and extract capability names under **New Capabilities** and **Modified Capabilities**
   > - For each capability, read the global spec (`openspec/specs/<capability>/spec.md`) and the delta spec (`openspec/changes/<name>/specs/<capability>/spec.md`)
   > - Read `openspec/changes/<name>/tasks.md` to identify which behaviors were in scope
   > - Skip any `openspec/changes/**` entries in the listing — those files are already provided as structured context
   >
   > Undeclared impact (the diff touches something not accounted for elsewhere) is itself a finding.
   >
   > Model: sonnet

4. **Print the full report verbatim.** Output the subagent's findings as a text message before doing anything else. Do not skip ahead to fixing.

5. **Act on findings** — only after printing. The main Claude acts on findings per the project workflow in CLAUDE.md.
