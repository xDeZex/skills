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

3. **Spawn the subagent(s).** The subagent(s) handle diff retrieval and any additional reading they need.

   **Incremental — one subagent, Quality + Spec combined:**
   > You are doing an incremental review. Read `references/reviewer-incremental.md` from your skill directory for the review lens.
   >
   > Run `git diff --unified=30 <range> -- <files>` for the range identified below (or `git show --unified=30 HEAD -- <files>` if none was given) to get the full diff with context. Use that diff as your primary source — do not rely on reading file state to reconstruct what changed.
   >
   > If task context is provided below, review against it and do not re-read tasks.md or spec files.
   >
   > `openspec/changes/**` files are already summarized in the context provided above — don't re-read them to rebuild that context. If any such file is itself part of the changed-file list, still review it for quality (internal consistency, accurate cross-references, examples that match their own rule) like any other changed file.
   >
   > Undeclared impact (the diff touches something not accounted for elsewhere) is itself a finding.
   >
   > Model: sonnet

   **Full — two parallel subagents, one per axis.** Send a single message with both `Agent` tool calls.

   Quality subagent:
   > You are doing a full review, Quality axis. Read `references/reviewer-full.md` from your skill directory for the review lens.
   >
   > Below is a list of changed files and the command to get the full diff (`git diff origin/main -- <files>`, or the range/ref the user pointed at). Get the diff first — it shows where to focus — then read each changed file in full before reviewing.
   >
   > `openspec/changes/**` files are already summarized in the context provided above — don't re-read them to rebuild that context. If any such file is itself part of the changed-file list, still review it for quality (internal consistency, accurate cross-references, examples that match their own rule) like any other changed file; spec compliance itself is handled by a separate review.
   >
   > Undeclared impact (the diff touches something not accounted for elsewhere) is itself a finding.
   >
   > Model: sonnet

   Spec subagent — only spawn if step 2 found an active change with in-scope scenarios:
   > You are doing a full review, Spec axis. Read `references/reviewer-spec.md` from your skill directory for the review lens.
   >
   > Below is the diff command and commit list, the active OpenSpec change name, and the WHEN/THEN scenarios in scope from step 2. Do not re-read tasks.md or the proposal for scope — use what's provided.
   >
   > Model: sonnet

   If step 2 found no spec to review against, skip the Spec subagent — do not spawn it.

4. **Print the full report verbatim.** Output the findings as a text message before doing anything else, in order — do not skip ahead to fixing. Then act on the findings per the project's documented workflow (e.g. `AGENTS.md`, `CLAUDE.md`), if one exists.
   - Incremental: print the single subagent's report as-is.
   - Full: print the Quality subagent's report under a `## Quality` heading, then the Spec subagent's report under a `## Spec` heading — or, if the Spec subagent was skipped, a `## Spec` heading with a one-line "no spec available for this review" note. Do not merge, rerank, or compare findings across the two headings — each axis stands on its own.
