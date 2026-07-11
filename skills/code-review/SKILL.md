---
name: code-review
description: Review a diff via focused subagents — Quality (correctness, tests, style, smells) always, plus Spec (does it match the OpenSpec scenarios in scope) for full reviews. Use after a commit or a batch of related commits, at the end of a branch, or whenever the user asks for a review — for source code or any other changed files (docs, config, specs). Findings come back fix-oriented, not just observations, so prefer this over reviewing the diff yourself.
---

Spawn a review subagent with the diff and let it gather its own context.

## Determine scope

- **Incremental** — a commit, a range of commits, or a set of files the caller already identified as "this batch of work" (e.g. one TDD cycle, one task group). Review just that range, not the full diff.
- **Full** — end of the work, user-triggered, or any ad-hoc request. The full branch diff against main.

## Steps

1. **Get changed file names — nothing else**
   - Incremental: use the commit range or file list already identified for this batch (e.g. `git diff --name-only <parent>..HEAD`); if none was given, default to the last commit (`git show --name-only HEAD`)
   - Full: `git diff --name-only origin/main` — unless the user pointed at something more specific (a file path, commit ref, or range)

2. **Gather task/spec context before spawning, if an OpenSpec change is active:**
   - Run `openspec list --json` only if you don't already know whether a change is active.
   - Incremental: read `openspec/changes/<name>/tasks.md` and identify the current in-progress task (task number + full description). Read the delta spec for the matching capability and extract the WHEN/THEN scenarios in scope for that task only — the batch being reviewed is just that task.
   - Full: read `openspec/changes/<name>/proposal.md` and extract every capability listed under **New Capabilities** and **Modified Capabilities**. For each, read the delta spec and collect every WHEN/THEN scenario for the change as a whole — not just the current in-progress task — since a full branch diff can include completed tasks and multiple capabilities. For **Modified Capabilities**, also read the base spec (`openspec/specs/<capability>/spec.md`) and include its unchanged scenarios — the diff must not regress a contract the delta doesn't touch. Read `tasks.md` to confirm which behaviors were actually in scope.
   - This context is passed inline to the subagent(s) — do not leave it for a subagent to gather.
   - If no active change, or no in-scope scenarios are found, there is no spec to review against — proceed without spec context.

3. **Spawn the subagent(s).** Each is a named agent defined in `agents/` (model + effort fixed in its frontmatter — do not override with a `Model:` line). The subagent(s) handle diff retrieval and any additional reading they need. These agent definitions must also exist at `.claude/agents/` (project or user level) wherever this skill is installed, or `subagent_type` will fail to resolve — copy them alongside the skill itself.

   **Incremental — one subagent, Quality + Spec combined.** Spawn `subagent_type: code-reviewer-incremental` (`agents/code-reviewer-incremental.md`, effort: medium):
   > Range: <the range identified in step 1, or the last commit if none was given>. Files: <the file list from step 1>.
   >
   > Task context (if step 2 found one): <task number + full description, and the in-scope WHEN/THEN scenarios>.

   **Full — two parallel subagents, one per axis.** Send a single message with both `Agent` tool calls.

   Quality subagent — `subagent_type: code-reviewer-quality` (`agents/code-reviewer-quality.md`, effort: high):
   > Changed files: <the file list from step 1>. Diff command: `git diff origin/main -- <files>` (or the range/ref the user pointed at).

   Spec subagent — `subagent_type: code-reviewer-spec` (`agents/code-reviewer-spec.md`, effort: medium) — only spawn if step 2 found an active change with in-scope scenarios:
   > Diff command and commit list, the active OpenSpec change name, and the WHEN/THEN scenarios in scope from step 2.

4. **Print the full report verbatim.** Output the findings as a text message before doing anything else, in order — do not skip ahead to fixing. Then act on the findings per the project's documented workflow (e.g. `AGENTS.md`, `CLAUDE.md`), if one exists.
   - Incremental: print the single subagent's report as-is.
   - Full: print the Quality subagent's report under a `## Quality` heading, then the Spec subagent's report under a `## Spec` heading — or, if the Spec subagent was skipped, a `## Spec` heading with a one-line "no spec available for this review" note. Do not merge, rerank, or compare findings across the two headings — each axis stands on its own.
