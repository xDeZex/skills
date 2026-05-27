---
name: commit
description: Make a git commit with the issue number in brackets in the message, and close the issue if this commit completes all its work. Use this skill whenever the user asks to commit, says "make a commit", "commit this", "commit my changes", "/commit", or wants to save their work to git. Always use this skill for commits — it ensures issues are properly referenced and closed.
---

# Commit Skill

Make a git commit that references the relevant GitHub issue, and closes it when this commit completes all the work.

## Commit message format

```
[#N] type: short description

Optional body with more detail.

Closes #N
```

- The `[#N]` prefix goes at the very start of the subject line.
- `type` follows conventional commits: `feat`, `fix`, `ci`, `refactor`, `test`, `docs`, `chore`.
- The `Closes #N` trailer is only added when this commit finishes all the work in the issue. If it's partial work, omit it — just the `[#N]` prefix suffices.
- If multiple issues are addressed, list them: `[#5, #7]` and add a `Closes` line per completed issue.
- If there's no related issue, omit the `[#N]` prefix entirely and commit normally.

**Example — completing an issue:**
```
[#5] feat: add FeedSourceRepository with DataStore persistence

Implements CRUD operations for feed source URLs/names backed by
Jetpack DataStore.

Closes #5
```

**Example — partial work:**
```
[#8] feat: add ArticleCard composable
```

## Step-by-step

### 1. Understand what changed

Run these in parallel:
- `git status` — see staged/unstaged files
- `git diff HEAD` — see full diff of changes
- `git log --oneline -5` — see recent commit style to match

### 2. Find the issue number

Try in order until you have it:

1. **User told you** — they mentioned "#5" or "issue 5" in their request.
2. **Branch name** — extract from branch names like `feature/issue-5-feed-repo` or `feat/5-something`. Run `git branch --show-current`.
3. **Recent commit** — if prior commits on this branch already reference an issue, use the same one.
4. **Ask the user** — if none of the above works, ask: "Which issue does this work relate to? (or say 'none' if there isn't one)"

### 3. Decide whether to close the issue

Look at the issue to understand its scope: `gh issue view <N>`. Compare the issue's checklist/description against what the diff actually implements.

Close the issue (`Closes #N`) when the diff covers everything the issue describes. If it's one piece of a larger issue, just reference it with `[#N]`.

When in doubt, don't close — it's easy to close manually, hard to reopen.

### 4. Stage and commit

Stage only relevant files — prefer naming them explicitly over `git add -A` to avoid accidentally including secrets or generated files.

Then commit:
```bash
git commit -m "$(cat <<'EOF'
[#N] type: description

Closes #N
EOF
)"
```

Omit the blank line + `Closes #N` if not closing the issue.

### 5. Confirm

Run `git log --oneline -1` and show the user the resulting commit line.
