---
name: vscode
description: >
  Apply Visual Studio Code best practices for team-wide productivity, reproducibility, and security.
  Use when defining workspace standards, settings/profiles, extension governance, language tooling,
  debugging/tasks automation, remote development workflows, and troubleshooting playbooks.
---

# VS Code

## Workflow
1. Confirm organization constraints for updates, tooling, and security policy.
2. Define workspace contract under version control.
3. Standardize settings scopes, profiles, and sync expectations.
4. Curate extension governance and performance diagnostics.
5. Configure language tooling, debugging, tasks, and automation.
6. Align local workflows with VCS and CI/CD expectations.
7. Establish remote development strategy where needed.
8. Track operational metrics and maintain troubleshooting runbooks.

## Preflight (Ask / Check First)
- Stable vs Insiders policy and update cadence.
- OS and hardware mix across developer fleet.
- Repo size/profile (monorepo, polyrepo, generated assets, watcher pressure).
- Team language stack and required extension set.
- Security/privacy requirements for extensions and telemetry.
- Remote development needs (SSH, containers, WSL, Codespaces).

## Operating Principles
- Treat editor configuration as code, not individual preference only.
- Keep workspace settings minimal and correctness-focused.
- Use user profiles for personal ergonomics.
- Prefer reproducible tasks/launch configs over ad-hoc commands.
- Keep extension surface area intentionally small.
- Use diagnostics to solve performance issues with evidence.

## Installation and Update Strategy
- Choose default channel (Stable for most teams).
- Use Insiders as canary for early validation.
- Define enterprise update policy if centrally managed.
- Keep rollback/reset guidance for corrupted setups.
- Document supported version baseline for extensions/tooling.

## Workspace and Project Organization
- Commit `.vscode/` contract files where team value is clear.
- Standardize `settings.json`, `tasks.json`, `launch.json`, and `extensions.json`.
- Use multi-root workspaces only when they reduce friction.
- Align file/search/watcher exclusions for large repos.
- Keep generated folders excluded to reduce noise and CPU overhead.

### Workspace Contract Checklist
- `.vscode/settings.json`
- `.vscode/tasks.json`
- `.vscode/launch.json`
- `.vscode/extensions.json`
- Optional `.code-workspace` for multi-root workflows

## Settings Scope, Sync, and Profiles
- Keep team correctness settings in workspace scope.
- Keep personal theming/keymaps/fonts in user profile scope.
- Use profiles for role-based workflows (backend/frontend/data).
- Treat Settings Sync as personal continuity, not team policy.
- Validate remote-scope differences when working in SSH/container contexts.

## Extension Governance and Performance
- Approve extensions by necessity, reputation, and maintenance health.
- Document required vs optional extensions by project role.
- Use extension bisect and startup profiling for regressions.
- Remove stale or overlapping extensions periodically.
- Review extension permissions in security-sensitive environments.

### Extension Review Questions
- Does this extension duplicate existing capability?
- Is it actively maintained and trusted?
- Does it need broad workspace/network access?
- Can this be solved by native VS Code feature instead?

## Language-Specific Workflows
- Standardize formatter/linter/test integration per language.
- Ensure command-line tools and VS Code diagnostics match CI behavior.
- Keep language server versions aligned with repo tooling.
- Document per-language debug configurations and tasks.
- Avoid hidden local-only setup that bypasses team defaults.

## Debugging, Testing, and Tasks
- Treat `launch.json` and `tasks.json` as reproducible runbooks.
- Keep debug tasks composable and named by intent.
- Wire test tasks for quick local feedback.
- Use problem matchers and terminal integration consistently.
- Keep one-command "happy path" for common contributor workflows.

### Typical Local Commands
```bash
code --version
code --status
```

```bash
rg -n "\.vscode|launch.json|tasks.json|extensions.json" .
```

## Version Control and Collaboration
- Keep Git integration conventions explicit (branch naming, commit hooks).
- Align pre-commit and format-on-save behavior with CI checks.
- Use Live Share or pair tooling intentionally and securely.
- Keep issue/PR templates easy to access from editor workflows.

## Remote Development Strategy
- Choose SSH, dev containers, WSL, or Codespaces by use case.
- Prefer dev containers for reproducible toolchains across environments.
- Keep remote extensions scoped and documented.
- Validate filesystem and watcher settings in remote contexts.
- Ensure secrets handling differs safely between local and remote hosts.

## Security and Privacy
- Enforce Workspace Trust guidance for untrusted repos.
- Minimize extension privileges and review telemetry posture.
- Keep secret scanning and terminal history practices documented.
- Avoid checking sensitive settings into workspace files.
- Define escalation path for suspected extension compromise.

## Accessibility and Inclusion
- Validate themes/contrast for readability standards.
- Support keyboard-first workflows and shortcut discoverability.
- Keep font/zoom and screen-reader options documented.
- Ensure onboarding materials include accessibility-friendly defaults.

## Metrics and Troubleshooting
- Track startup time, extension impact, and crash frequency.
- Measure onboarding time to first successful build/test/debug.
- Keep runbooks for watcher limits, extension conflicts, and terminal issues.
- Use process explorer and logs before broad reinstallation.
- Capture common fixes in versioned docs.

## Common Failure Modes
- Bloated extension sets causing startup slowdown.
- Workspace settings overriding personal settings unexpectedly.
- Non-reproducible debug tasks that differ from CI.
- File watcher limits on large repositories.
- Remote setup drift between developers.
- Security risks from unchecked extension trust.

## Definition of Done
- Workspace contract is complete and version-controlled.
- Extension set is curated, justified, and documented.
- Debug/test/tasks workflows are reproducible across machines.
- Security and trust controls are explicit.
- Troubleshooting runbooks exist for common failures.

## References
- `references/vscode.md`

## Reference Index
- `rg -n "installation|updates|Stable|Insiders" references/vscode.md`
- `rg -n "workspace|\.vscode|multi-root|settings" references/vscode.md`
- `rg -n "profiles|sync|scope|remote settings" references/vscode.md`
- `rg -n "extensions|performance|bisect|process explorer" references/vscode.md`
- `rg -n "debugging|tasks|automation|launch" references/vscode.md`
- `rg -n "Git|collaboration|remote development|dev container" references/vscode.md`
- `rg -n "security|privacy|workspace trust|telemetry|troubleshooting" references/vscode.md`

## Quick Questions (When Stuck)
- Is this problem caused by settings scope, extension behavior, or environment drift?
- Can the workflow be encoded in tasks/launch instead of tribal knowledge?
- Is this extension essential enough to justify its risk and overhead?
- Does local editor behavior match CI and team standards?
- What one runbook entry would prevent this issue next time?
