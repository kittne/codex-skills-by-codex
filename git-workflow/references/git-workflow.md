# Git Workflow Reference

This reference provides longer-form guidance behind `git-workflow`.

## Branching
- Create a branch per unit of work.
- Prefer descriptive, kebab-case names:
- `feature/...`, `fix/...`, `chore/...`, `hotfix/...`.

## Conventional Commits
Format:
```text
<type>(<scope>): <summary>
```

Guidelines:
- Use imperative summaries ("add", "fix", "remove").
- Keep commits small and cohesive.
- Use `!` and `BREAKING CHANGE:` only for breaking changes.

Common types:
- `feat`, `fix`, `docs`, `refactor`, `perf`, `test`, `chore`, `ci`, `build`

## Syncing With Base
- Prefer the repo's convention (merge commits vs linear history).
- Only rebase branches you own; avoid rewriting history on shared branches.

Typical rebase:
```bash
git fetch origin

git rebase origin/main
```

Typical merge:
```bash
git fetch origin

git merge origin/main
```

## Changelog and Versioning
- Follow the repository's existing release/versioning workflow.
- Update changelog/version files only if the repo already uses them or the user requests it.

## PR Preparation Checklist
- Branch is up to date with the base branch.
- Commit history is clean and readable.
- Tests/lint pass or failures are explained.
- Docs/changelog updated if required.
- No secrets or unintended large files included.

## Recovery
Stash work:
```bash
git stash push -m "WIP"
```

Amend last commit:
```bash
git commit --amend
```

Undo last commit but keep changes:
```bash
git reset --soft HEAD~1
```

Revert on shared branches:
```bash
git revert SHA
```

Abort operations:
```bash
git rebase --abort

git merge --abort
```

## Sources
- Conventional Commits: https://www.conventionalcommits.org/
- Atlassian Git tutorials (rebase/merge, reset/revert): https://www.atlassian.com/git/tutorials
- Codex skills: https://developers.openai.com/codex/skills/create-skill/
