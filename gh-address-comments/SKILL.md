---
name: gh-address-comments
description: Help address review/issue comments on the open GitHub PR for the current branch using gh CLI; verify gh auth first and prompt the user to authenticate if not logged in.
---

# PR Comment Handler

Use this skill to locate the PR for the current branch and resolve review comments with `gh`.

## Goals
- Identify the correct PR for the current branch.
- Summarize all review threads and inline comments.
- Let the user choose which comments to address.
- Apply fixes and confirm comment resolution status.

## Preconditions
- `gh` is installed and authenticated.
- The repo has a PR for the current branch, or the user provides the PR number/URL.

## Workflow
1. Confirm repo and branch context.
2. Verify `gh` authentication.
3. Locate the PR for the current branch.
4. Fetch review threads and inline comments.
5. Present a numbered summary to the user and ask what to address.
6. Apply fixes for selected items.
7. Re-check threads to ensure comments are resolved.

## Authentication Checks
- Run `gh auth status`.
- If unauthenticated, ask the user to run `gh auth login` and confirm when done.

## Find the PR
Prefer `gh pr view --json` on the current branch. If it fails:
- Ask the user for a PR number or URL.
- Or list recent PRs with `gh pr list` and confirm the right one.

## If No PR Exists
- Ask whether to open a PR or wait.
- If the user wants a PR, follow the repo's normal branch/PR workflow.

## Fetch Comments
Use the bundled script:
```bash
python3 <path-to-skill>/scripts/fetch_comments.py
```
The script should print review threads and comment metadata. If it fails:
- Fall back to `gh pr view --json reviewThreads,comments`.
- Confirm the PR number/URL is correct.

## User Clarification
- Number each thread and comment.
- Provide a one-line summary and location for each item.
- Ask the user which numbers to address.

## Apply Fixes
- Implement the smallest safe change for each chosen comment.
- Keep fixes scoped; avoid unrelated refactors.
- If a comment is invalid or already addressed, explain why and ask whether to close it.

## Comment Resolution
- Prefer to update code first, then mark threads resolved.
- If a discussion is needed instead of code, draft a concise reply and ask the user to approve before posting.

## Commit Hygiene
- Use clear, focused commits if the repo expects them.
- Avoid batching unrelated fixes into one commit unless requested.

## Testing
- Run the most relevant tests or checks for the touched area when available.
- If tests are skipped, state why and what would be ideal to run.

## Re-check Status
- Re-run the comment fetch to ensure addressed items are resolved.
- Summarize remaining open threads.

## Output Expectations
- Clear mapping from comment number to change made.
- File and line references for each fix.
- A short note on any comment left unresolved and why.

## Edge Cases
- Multiple open PRs on the same branch: confirm with the user.
- Draft PRs: proceed only if the user approves.
- Missing repo remote: ask the user to confirm the GitHub remote or repo slug.
- Auth rate limits: pause and ask the user to retry after login.

## Suggested Commands
```bash
git status -sb
gh auth status
gh pr view --json number,title,url,reviewThreads,comments
```

## Definition of Done
- Selected review comments are addressed with code changes.
- Remaining open comments are explained.
- The PR still builds/tests if checks exist.
- The user knows which threads are resolved vs pending.
- The change list maps back to each requested comment.
