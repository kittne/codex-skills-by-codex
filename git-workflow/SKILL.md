---
name: git-workflow
description: >
  Use when the user requests Git help or when preparing commits, branches, or PRs. Enforce a clean,
  repo-agnostic workflow: branching, Conventional Commits, changelog hygiene (when the repo uses one),
  safe rebasing/merging, and recovery/rollback practices.
---

# Git Workflow

## Workflow
1. Identify the repo default branch and current state (`git status`, `git branch`, `git remote -v`).
2. Create a scoped branch for the work (avoid committing directly to default branch).
3. Make changes in small, logical units; stage deliberately.
4. Commit using Conventional Commits (or repo convention) with clear, imperative summaries.
5. Update changelog/version files only if the repo already uses them or the user requests it.
6. Sync with the base branch (merge or rebase per repo convention).
7. Run tests/lint and open a PR with a clear summary, risks, and testing notes.

## Branching
- Do not commit directly to the default branch unless the repo explicitly allows it.
Branch name patterns:
- `feature/<topic>` for new work.
- `fix/<topic>` for bug fixes.
- `chore/<topic>` for maintenance.
- `hotfix/<topic>` for urgent production fixes.

Common commands:
```bash
# Detect the remote default branch (often "main" or "master")
DEFAULT_BRANCH="$(git remote show origin | sed -n '/HEAD branch/s/.*: //p')"

git fetch origin

git switch -c feature/my-topic "origin/$DEFAULT_BRANCH"
```

## Commits (Conventional Commits)
Use:
```text
<type>(<scope>): <summary>
```

Guidelines:
- Keep commits cohesive (one intent per commit).
- Keep the summary imperative and short.
- Use `!` and a `BREAKING CHANGE:` footer only for breaking changes.

Common `type` values:
- `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`

Examples:
```text
feat(auth): add Google OAuth callback endpoint
fix(api): return 404 on missing user
```

## Staging Discipline
- Prefer `git add -p` for clean commits when changes are interleaved.
- Avoid committing generated artifacts unless the repo expects them.

## Changelog and Versioning
- If a `CHANGELOG.md` (or similar) exists, follow its established format.
- If the repo uses a release tool (changesets, semantic-release, cargo/npm versioning, etc.), use it.
- Do not introduce new versioning rules unless the user asks.

## Syncing With the Base Branch
Choose merge vs rebase based on the repo's convention:
- If the repo prefers merge commits, merge the base branch.
- If the repo prefers linear history, rebase your branch.

Rebase (typical):
```bash
git fetch origin

git rebase origin/main
```

Merge (typical):
```bash
git fetch origin

git merge origin/main
```

If conflicts happen:
```bash
git status
# edit files to resolve conflicts

git add PATH

git rebase --continue  # or: git merge --continue
```

## Pull Requests
Checklist before opening:
- Branch is up to date with the base branch.
- Commit history is clean (no WIP/fixup commits unless the repo wants them).
- Tests/lint pass (or failures are explained).
- Docs/changelog updated if required by the repo.
- No secrets, credentials, or large unintended artifacts included.
- PR description includes:
  - What changed and why.
  - How it was tested.
  - Risks/rollout notes (migrations, flags, compatibility).

## Recovery and Rollback
Stash WIP:
```bash
git stash push -m "WIP"
```

Amend last commit (only if safe):
```bash
git commit --amend
```

Undo last commit but keep changes:
```bash
git reset --soft HEAD~1
```

Revert a bad commit on a shared branch:
```bash
git revert SHA
```

Find "lost" commits after a bad reset/rebase:
```bash
git reflog
```

Abort an in-progress operation:
```bash
git rebase --abort

git merge --abort
```

## References
- `references/git-workflow.md`

## Extended Guidance
Use this when resolving conflicts or preparing releases.

## Conflict Resolution Defaults
- Prefer `git status` and `git diff` before and after every conflict fix.
- Keep conflict resolutions focused; avoid mixing refactors.
- Re-run tests after conflict resolution when feasible.
