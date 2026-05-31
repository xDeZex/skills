---
name: code-review
description: Spawn a focused code review subagent. Use after every RED/GREEN cycle (pass the files changed in that cycle), after a commit, or whenever the user asks for a review. Always use this skill for reviews — it ensures the review lens is test-first and findings are fix-oriented.
---

Spawn a code review subagent with the diff and let it gather its own context.

## Determine scope

- **Per-behavior cycle** (during openspec apply): the files changed in this RED/GREEN cycle only — not the full diff
- **Final review** (end of all OpenSpec tasks, user-triggered, or any ad-hoc request): the full branch diff against main

## Steps

1. **Get changed files and line ranges — nothing else**
   - Per-cycle: `git show --unified=0 HEAD -- <changed files> | grep -E "^(---|\\+\\+\\+|@@)"` (identify files from `git show --name-only HEAD`)
   - Final review: `git diff --unified=0 origin/main | grep -E "^(---|\\+\\+\\+|@@)"` — unless the user pointed at something more specific (a file path, commit ref, or range)

   The result is a compact listing of which files changed and which line ranges, with no diff content. Example:
   ```
   --- a/src/foo.py
   +++ b/src/foo.py
   @@ -45,3 +45,5 @@
   @@ -78,1 +80,1 @@
   ```

2. **Spawn a subagent** and pass it:
   - The changed-lines listing (file names + `@@` line ranges, no diff content)
   - The scope: per-cycle or final review
   - Whether an active OpenSpec change exists (run `openspec list --json` only if you don't already know)
   - The active change name, if any

   The subagent handles all further reading — reviewer reference, OpenSpec files, changed files. Do not pre-read those yourself.

   **Per-cycle subagent instructions:**
   > You are doing a per-cycle code review. Read `references/reviewer-light.md` from your skill directory for the review lens.
   >
   > Below is a listing of which files changed and which line ranges (no diff content). For each hunk, read ±30 lines of context around the changed range directly from the file — do not ask for a diff.
   >
   > If an active OpenSpec change is named, gather context yourself:
   > - Read `openspec/changes/<name>/proposal.md` and extract capability names under **New Capabilities** and **Modified Capabilities**
   > - For each capability, read the global spec (`openspec/specs/<capability>/spec.md`) and the delta spec (`openspec/changes/<name>/specs/<capability>/spec.md`)
   > - Read `openspec/changes/<name>/tasks.md` to identify the behavior in scope
   > - Skip any `openspec/changes/**` entries in the listing — those files are already provided as structured context
   >
   > Undeclared impact (code touches something not listed in the proposal) is itself a finding.
   >
   > Model: haiku

   **Final review subagent instructions:**
   > You are doing a final code review. Read `references/reviewer.md` from your skill directory for the review lens.
   >
   > Below is a listing of which files changed and which line ranges (no diff content). Read each changed file in full before reviewing — use the line ranges only to know which files to open.
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

3. **Print the full report verbatim.** Output the subagent's findings as a text message before doing anything else. Do not skip ahead to fixing.

4. **Act on findings** — only after printing. The main Claude acts on findings per the project workflow in CLAUDE.md.
