# Review lens

Check every finding against a RED test: could a failing test have caught this? A logic error with no test is a missing test as much as a bug — the fix is a RED test that exposes it, then the code change. A test that passes before the implementation exists, or breaks on an internal rename, is not a real test — flag it as its own finding.

For non-code changes (docs, config, schemas), apply the same rigor without assuming tests exist: does a passage contradict another part of the same or a linked file? Does a worked example match the rule it illustrates? Is a stated fact (a count, an order, a cross-reference) still accurate? Fix these by editing the text, not by writing a test.

OpenSpec files (`openspec/changes/**`, `openspec/specs/**`) are the Spec axis's territory end to end — skip them here entirely, even when they appear in the changed-file list.

## Correctness

- Logic errors → suggest the RED test that would catch it, then the fix
- Missing behavior coverage → suggest the test that should exist
- Edge cases the current tests don't reach
- Undeclared impact — the diff touches something not accounted for elsewhere — is itself a finding

## Implementation scope

Code that exists but that no current test requires is speculative. Flag it — either a test should be written to justify it (RED), or it should be removed.

## Test quality

- Does the test verify behavior through public interfaces only? A test that reaches into private methods or internal state is testing implementation, not behavior — it will break on refactors for no good reason
- Is the test asserting the right thing, or just that the code ran without crashing?
- Would this test catch a regression if the behavior changed?

## Style and structure

Flag clearly and suggest the fix directly. Apply the Fix now / Needs filing bar from Output format below. Check the diff against the smell baseline in `references/smells.md` as part of this section.

## Architecture

- Clean separation of concerns — is each module responsible for exactly one thing?
- Public interfaces use domain types, not raw primitives
- Apply the Fix now / Needs filing bar from Output format below.

## Output format

When quoting code in findings, redact any value that looks like a secret (tokens, keys, passwords, connection strings) — replace the value with `[REDACTED]`. Never include raw secret values in output.

Report findings only — do not fix or file anything yourself. Every finding falls into exactly one of these two buckets — decide it and label it, don't leave the disposition implied in prose.

**Fix now** — the default for everything related to the diff under review, including named smells like Duplicated Code, and including fixes that cross file or module boundaries or change a public signature or test contract. Scale is not a reason to file — a fix that touches many files or is structurally significant is still Fix now as long as it's in scope. For each: what is wrong, why it matters (in testing terms where possible), what to do. Where the fix is "write a test first": give a one-line sketch of the test.

**Needs filing** — rare. Reserved only for a large, genuinely out-of-scope problem: something you noticed incidentally that has nothing to do with the current diff, big enough that fixing it now would mean doing unrelated work no one asked for. If the problem is in or touches the changed code, it is Fix now, not Needs filing. For each: what is wrong, why it's unrelated to this diff, why it's too large to fix in passing.

If there is nothing to raise, say so.
