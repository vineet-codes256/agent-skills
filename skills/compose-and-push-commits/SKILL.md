---
name: compose-and-push-commits
description: Organise staged and unstaged changes into meaningful, atomic commits, then push them to the remote. Use when you have a messy working tree and want clean git history published upstream. Trigger on "compose and push commits", "compose commits", "organise commits", "split commits", "clean up commits", "commit and push", or "commit my changes".
---

# Compose and Push Commits

You are a senior engineer who writes clean, atomic, reviewable git history. Your job is to look at everything in the working tree, turn it into a set of well-scoped commits — one logical change per commit, no more, no less — and push the result to the remote.

**Core principle:** A good commit history tells the story of the change. Each commit should be independently reviewable, independently revertable, and have a subject line that answers "what did this change and why."

---

## The Mandatory Loop

```
1. SURVEY   — understand the full diff (staged + unstaged + untracked)
2. GROUP    — cluster changes into logical units
3. PLAN     — propose commit sequence + push target to the user; wait for approval
4. COMMIT   — stage and commit each group in order
5. VERIFY   — confirm clean tree, show final log
6. PUSH     — push the new commits to the remote
```

Never skip step 3. The user must approve the plan before any commit is created or pushed.

---

## Step 1 — Survey

Run all of these in parallel. Report every finding.

```bash
# Overall working-tree state
git status --short

# Full diff of tracked changes (staged + unstaged)
git diff HEAD

# Staged-only diff
git diff --cached

# List untracked files
git ls-files --others --exclude-standard

# Recent commits for style reference
git log --oneline -10

# Current branch, upstream, and remote
git branch --show-current
git remote -v
git rev-parse --abbrev-ref --symbolic-full-name @{upstream} 2>/dev/null || echo "no upstream"
```

Look for:
- Files that were modified together for the same reason (cohesive → same commit)
- Files that were modified for different reasons (separate commits)
- New files that belong with an existing modified file
- Deleted files that are part of a rename or refactor
- Test files — do they belong with the implementation they test?
- Config/tooling changes that are independent of feature work
- Migration files — always commit with the code that requires them

---

## Step 2 — Group

Apply these grouping rules in order of priority:

**Always separate:**
- Schema migrations from application code (unless the migration is a one-liner triggered by the feature)
- Dependency changes (`package.json`, lock files, `requirements.txt`, etc.) from feature code
- Config/tooling (linters, build config, CI) from application code
- Test-only changes from implementation changes (unless TDD — test + impl belong together)
- Bug fixes from feature additions
- Refactors from behaviour changes

**Always keep together:**
- A component and its co-located test (`Foo.tsx` + `Foo.test.tsx`)
- A migration and the seed data it creates (if inseparable)
- An API endpoint, its types, and the client code that calls it
- Database access policies/constraints and the table they protect (within the same migration file)

**When in doubt:** ask yourself "if I revert this commit alone, does the app still compile and run?" If not, the commit is too small.

---

## Step 3 — Plan

Present the proposed commit sequence as a numbered list. Each entry must include:

```
N. <subject line>
   Files: <list of files>
   Reason: <one sentence — why these belong together>
```

Subject line rules (enforce these):
- Imperative mood, present tense: "Add", "Fix", "Remove", "Update", "Refactor" — not "Added", "Fixed"
- Under 72 characters
- No trailing period
- Format: `<type>: <what>` where type is one of: feat, fix, refactor, test, chore, style, docs, perf, migration
- If the change touches a named surface or module, include it as scope: `feat(auth): add password reset flow`

State the push target explicitly: "After committing, I will push to `<remote>/<branch>`."

Then ask: **"Does this grouping and push target look right? Any changes before I start committing?"**

Wait for explicit approval. If the user says "go" or "looks good" or equivalent, proceed. If they request changes, revise the plan and ask again.

---

## Step 4 — Commit

For each group in the approved plan, in sequence:

1. `git add <specific files only>` — never `git add -A` or `git add .`
2. Verify with `git diff --cached --stat` that only the intended files are staged
3. Commit with a HEREDOC:

```bash
git commit -m "$(cat <<'EOF'
type(scope): subject line

Optional body paragraph if the why is non-obvious. One blank line after
subject. Wrap at 72 chars. Do not describe the what — the diff shows that.
Explain constraints, workarounds, or intent that the code cannot express.
EOF
)"
```

4. If a pre-commit hook fails: fix the issue, re-stage the same files, create a NEW commit (never --amend)
5. Report the commit hash after each successful commit

After all commits: run `git status` to confirm a clean tree.

---

## Step 5 — Verify

```bash
# Show the commits just created
git log --oneline -$(N+2)

# Confirm working tree is clean
git status
```

Report the log to the user. If anything is unintentionally uncommitted, ask whether to create a follow-up commit or leave it for the next session.

---

## Step 6 — Push

Only after the working tree is clean and every planned commit exists:

1. If the branch has an upstream: `git push`
2. If the branch has no upstream: `git push -u origin <branch>`
3. Confirm the push succeeded and report the remote ref:

```bash
git log --oneline @{upstream} -3
git status -sb
```

Push rules (never break these):
- **Never force-push.** No `--force`, no `--force-with-lease`. If the push is rejected as non-fast-forward, stop and tell the user — suggest `git pull --rebase` but let them decide.
- Push only the current branch. Never `git push --all`.
- If there is no remote configured, stop and ask the user where to push.
- If a pre-push hook fails, fix the issue (new commit, never --amend), then push again.

---

## Edge Cases

**Partial-file changes (hunks):** If two logically separate changes live in the same file, use `git add -p <file>` to stage hunks interactively. Describe which hunks belong to which commit before staging.

**Untracked files with no obvious home:** Propose a `chore: add <description>` commit and ask the user to confirm.

**Already-staged files:** If files are already staged, include them in the survey output and ask whether the existing staging intention should be preserved or reorganised.

**Merge conflicts:** Do not attempt to resolve merge conflicts. Stop and tell the user.

**Empty diffs:** If `git diff HEAD` is empty (everything already committed), say so. If there are unpushed commits, offer to skip straight to step 6 and push them. Otherwise exit the skill immediately.

**Detached HEAD:** Do not commit or push from a detached HEAD. Stop and tell the user.

**Protected/default branch:** If the current branch is the repo's default branch and recent history suggests a PR-based workflow, point this out in the plan and ask whether to push directly or create a feature branch first.

---

## What "Done" Means

- `git status` shows nothing to commit, working tree clean
- `git log --oneline` shows the new commits with clear, consistent subject lines
- No commit contains more than one logical change
- No logical change is split across multiple commits (unless intentional and noted)
- Pre-commit and pre-push hooks passed for every commit
- The branch is pushed and `git status -sb` shows it level with its upstream
