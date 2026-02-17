---
name: ubuntu
description: >
  Ubuntu Server administration, hardening, and operations best practices. Use when choosing Ubuntu
  release strategy (LTS vs interim), building reproducible images (autoinstall/cloud-init), securing
  remote access (SSH, MFA), configuring networking (Netplan), managing updates/upgrades (APT,
  unattended-upgrades, phased updates, do-release-upgrade), applying host/network hardening (UFW,
  AppArmor, systemd sandboxing), setting up logging/retention (journald), backups/restore drills,
  compliance automation (USG/CIS/STIG), and fleet management (Ubuntu Pro/Landscape).
---

# Ubuntu

Use this skill to turn "we need to run Ubuntu servers safely" into an actionable posture: a clear
release baseline, minimal exposed surface area, predictable patching/upgrades, and operational
discipline (logs, backups, monitoring).

Default bias:
- Prefer stable, boring, repeatable choices (LTS, declarative config, least privilege).
- Treat lockouts and surprise reboots as top risks; validate before applying.
- Provide verification commands for every recommendation.

## Workflow
1. Clarify system boundary and goals (single host vs fleet; prod vs dev; RPO/RTO; threat model).
2. Choose release posture (LTS by default for production; interim only with explicit reasons).
3. Decide deployment model (pets vs cattle; in-place upgrades vs immutable redeploy).
4. Establish identity and remote access posture (users, sudo, SSH, MFA, break-glass).
5. Reduce exposed surface area (services, ports, bind addresses, firewall rules).
6. Set patching strategy (unattended security updates vs image redeploy) and define reboot policy.
7. Harden host controls (AppArmor, systemd sandboxing, secure boot/FDE where in scope).
8. Configure operational baselines (journald retention, backups, monitoring signals).
9. Plan upgrades (staging/canary, proposed pocket testing when needed, sequential release upgrades).
10. Produce a short, owner-assigned plan with validation steps and failure modes.

## Quick Intake (Ask/Confirm)
- Ubuntu release(s) in use, and whether the environment is production.
- Deployment platform (bare metal, VM, cloud image, Kubernetes node).
- Lifecycle constraints (maintenance windows, RPO/RTO, reboot tolerance).
- Server role(s) and exposure (internet-facing vs internal; required ports).
- Identity model (local users, LDAP/SSO, key management) and break-glass requirements.
- Network config ownership (Netplan files, DHCP/static, DNS, MTU/VLAN, cloud networking).
- Update posture (unattended-upgrades enabled? phased updates allowed? canary ring exists?).
- Security baseline requirements (CIS/STIG, audit logging, FDE, secure boot).

## Release And Upgrade Strategy

Decision rules:
- Use LTS as the default production baseline.
- Use interim releases when you need newer kernel/toolchain/hardware enablement and can absorb churn.
- For cloud images, prefer rebuild + migrate rather than in-place upgrades (treat images as immutable).
- Keep upgrades sequential (do not skip LTS releases).

What to deliver:
- A documented baseline (e.g., "LTS only in prod") and an upgrade calendar.
- A canary ring (or staging) that takes updates earlier than the fleet.

## Build And Configuration At Scale

Principles:
- Treat provisioning as code: autoinstall + cloud-init for first boot configuration.
- Treat ongoing config as code: version-controlled config, reviewed changes, controlled rollout.

Practical defaults:
- Use Subiquity autoinstall for consistent installs where you control the OS image.
- Use cloud-init for instance bootstrapping (cloud/VM templates).
- Use a config manager (Ansible/Puppet/etc.) when configuration drift is a real risk.

## Networking (Netplan)

Rules:
- Keep `/etc/netplan/*.yaml` under change control.
- Validate and apply carefully on remote systems to avoid losing access.
- Prefer a rollback plan for network changes (out-of-band console, or staged apply window).

Common commands:
```bash
netplan status
netplan generate
netplan apply
ip addr
ip route
resolvectl status
```

## Time Sync

Rules:
- Ensure time sync is enabled and stable.
- Use multiple sources for resiliency.

Common commands:
```bash
timedatectl status
chronyc sources -v || true
```

## Remote Access And Identity (SSH)

Defaults:
- Admin as a non-root user; use sudo for elevation.
- Prefer key-based SSH; disable password auth unless explicitly required.
- Disable or tightly restrict root SSH login.
- Restrict SSH reachability with firewall/source IP allowlists where practical.

High-risk change guardrails (avoid lockout):
```bash
sshd -t
systemctl reload ssh
```

Account lifecycle trap:
- Locking a password does not remove existing SSH authorized keys; remove keys explicitly when offboarding.

## Firewalling (UFW / nftables)

Defaults:
- Deny incoming by default; allow only required service ports.
- Prefer source-limited rules for administrative ports.
- Align firewall policy with bind addresses (do not rely on one layer only).

Common commands:
```bash
ufw status verbose
ufw enable
ss -lntup
```

## Reduce Exposed Services And Open Ports

Rules:
- Identify what is listening and why.
- Disable services you do not need and ensure they do not start as dependencies.
- Avoid binding to wildcard/public addresses unless required.

Common commands:
```bash
ss -lntup
systemctl --type=service --state=running
systemctl disable --now <service>
```

## Mandatory Access Control (AppArmor)

Defaults:
- Keep AppArmor enabled.
- Prefer enforce mode for critical daemons once validated.
- Use complain mode to iterate safely on profiles.

Common commands:
```bash
aa-status
aa-enforce /etc/apparmor.d/<profile> || true
aa-complain /etc/apparmor.d/<profile> || true
```

## systemd Service Hardening

Use this when you own the unit files and want defense-in-depth.

Workflow:
1. Discover current posture:
   - `systemd-analyze security`
2. Apply targeted sandboxing via drop-in overrides.
3. Restart and validate service behavior.

Common commands:
```bash
systemd-analyze security <unit> || true
systemctl cat <unit>
systemctl edit <unit>
```

## Updates And Patching

Choose one of these models and make it explicit:
- In-place updates: unattended security updates plus controlled reboots.
- Immutable updates: redeploy from patched images; disable auto-updates if the pipeline is the patching mechanism.

Unattended upgrades guardrails:
- Monitor logs under `/var/log/unattended-upgrades/`.
- Be aware timers can run on boot if missed and can block other APT operations.
- Prefer drop-in configuration rather than editing vendor config directly.

Common commands:
```bash
apt-get update
apt-get -s upgrade
systemctl list-timers | rg -n "apt-daily"
ls -la /var/log/unattended-upgrades/ || true
test -f /run/reboot-required && echo "reboot required" || true
```

Phased updates rule:
- Treat "kept back" as a safety mechanism; use canaries for early uptake rather than disabling phasing globally.

## Release Upgrades (do-release-upgrade)

Rules:
- Fully patch current system first; reboot if required.
- Ensure disk space and a maintenance window.
- Expect third-party repos/PPAs to be disabled during upgrade; plan re-enable after validation.
- Take backups before upgrading.

Common commands:
```bash
do-release-upgrade -c
lsb_release -a || cat /etc/os-release
df -h
```

## Ubuntu Pro / ESM / Livepatch

Use Ubuntu Pro when you need:
- Extended security coverage beyond standard support.
- Broader security coverage for universe packages.
- Livepatch to shorten exposure windows for critical kernel vulnerabilities.

Common commands:
```bash
pro status || true
pro security-status || true
canonical-livepatch status || true
```

## Supply Chain Hygiene (APT And Third-Party Repos)

Rules:
- Prefer official APT repositories and the built-in chain of trust.
- Minimize third-party repos; treat them as controlled exceptions with explicit risk review.
- Use snapshot/known-good baselines when you need deterministic rollouts.

Checks:
```bash
apt-cache policy | head
ls -la /etc/apt/sources.list.d || true
```

## Logging (journald)

Rules:
- Decide whether logs must persist across reboots.
- Cap disk usage so logs do not crowd out application data.
- Centralize logs for incident response when the host is not the source of truth.

Common commands:
```bash
journalctl --disk-usage
journalctl -u ssh --since "24 hours ago" || true
test -d /var/log/journal && echo "persistent journald enabled" || true
```

## Backups And Restore Drills

Rules:
- Define what is backed up, frequency, retention, and off-site storage.
- Practice restores (restore drills are the test of backups).
- Treat `/etc` configuration as part of your recovery plan (version control where feasible).

Minimum outcome:
- A written restore procedure that someone else can run.

## Observability And Fleet Management

Rules:
- Monitor patch status and reboot requirement signals.
- Monitor disk pressure and log growth.
- Monitor critical daemons (SSH, networking, storage) and alert on failures.
- For large fleets, do not confuse unattended-upgrades with governance; use fleet tooling (Landscape or equivalents).

## Compliance Automation (USG / CIS / STIG)

Use this when you have a benchmark target.

Rules:
- Run audits first; understand what will change.
- Apply remediations in staging before production.
- Track exceptions explicitly (risk acceptance with owner and review date).

## Default Output Template (Server Posture Review)

```markdown
## Summary
- Environment:
- Role/exposure:
- Release posture:
- Patching model:

## Findings (Prioritized)
1. ...
2. ...

## Recommendations
1. ...
2. ...

## Verification Commands
- ...

## Risks / Failure Modes
- ...
```

## Common Failure Modes (Call Out Early)
- Locking yourself out by applying Netplan/SSH changes without validation.
- Leaving password auth enabled on internet-exposed SSH without compensating controls.
- Unattended upgrades blocking manual APT operations during startup windows.
- Treating "kept back" packages as an error and disabling phased updates fleet-wide.
- Offboarding users by locking passwords but leaving authorized SSH keys in place.
- Having backups but never testing restores.

## References
- Use `references/ubuntu-best-practices.md` for the full guide and deeper rationale.
- Find topics quickly:
  - `rg -n "Release strategy|lifecycle posture" references/ubuntu-best-practices.md`
  - `rg -n "autoinstall|cloud-init" references/ubuntu-best-practices.md`
  - `rg -n "Netplan|resolvectl" references/ubuntu-best-practices.md`
  - `rg -n "chrony|Time synchronization" references/ubuntu-best-practices.md`
  - `rg -n "SSH configuration|sshd -t|2FA" references/ubuntu-best-practices.md`
  - `rg -n "unattended-upgrades|phased updates|proposed" references/ubuntu-best-practices.md`
  - `rg -n "do-release-upgrade|Release upgrades" references/ubuntu-best-practices.md`
  - `rg -n "UFW|nftables|Firewalling" references/ubuntu-best-practices.md`
  - `rg -n "AppArmor" references/ubuntu-best-practices.md`
  - `rg -n "systemd-analyze security" references/ubuntu-best-practices.md`
  - `rg -n "journald|journalctl" references/ubuntu-best-practices.md`
  - `rg -n "Backups" references/ubuntu-best-practices.md`
  - `rg -n "USG|CIS|STIG" references/ubuntu-best-practices.md`

## Quick Questions (When Stuck)
- What is the minimal change that improves security without risking lockout?
- What is the reboot policy and the maintenance window?
- What is the deployment model (in-place vs redeploy) and how does patching happen?
- What is the smallest set of open ports required for this role?
- What is the rollback path for SSH and networking changes?
- What evidence would change the decision (logs, `ss`, `ufw status`, `aa-status`, APT output)?

