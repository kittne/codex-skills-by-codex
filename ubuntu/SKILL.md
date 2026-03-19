---
name: ubuntu
description: Ubuntu Server administration, hardening, and operations best practices. Use when choosing Ubuntu release strategy (LTS vs interim), building reproducible images (autoinstall/cloud-init), securing remote access (SSH, MFA), configuring networking (Netplan), managing updates/upgrades (APT, unattended-upgrades, phased updates, do-release-upgrade), applying host/network hardening (UFW, AppArmor, systemd sandboxing), setting up logging/retention (journald), backups/restore drills, compliance automation (USG/CIS/STIG), and fleet management (Ubuntu Pro/Landscape).
---

# Ubuntu

Turn "we need to run Ubuntu servers safely" into an actionable posture: stable release choices, minimal exposure, predictable patching, and operational discipline.

## Workflow
1. Clarify system boundary and goals (host vs fleet, prod vs dev, RPO/RTO).
2. Choose release posture (LTS by default for production).
3. Decide deployment model (immutable images vs in-place upgrades).
4. Define access and identity posture (users, sudo, SSH, MFA, break-glass).
5. Reduce exposed services and ports.
6. Set patching and reboot policy.
7. Apply host hardening (AppArmor, systemd sandboxing).
8. Configure logging, backups, and monitoring.
9. Plan upgrades with staging/canary steps.

## Quick Intake
- Ubuntu release(s) and environment.
- Deployment platform (bare metal, VM, cloud image).
- Exposure (internet-facing vs internal).
- Identity model and key management.
- Maintenance windows and reboot tolerance.
- Required compliance baselines (CIS/STIG).

## Release Strategy
- Use LTS as the default production baseline.
- Use interim releases only for hardware enablement or explicit needs.
- Keep upgrades sequential; do not skip LTS releases.

## Provisioning and Config
- Use autoinstall + cloud-init for reproducible builds.
- Keep config under version control.
- Prefer immutable images for fleet-scale changes.

## Networking (Netplan)
- Keep `/etc/netplan/*.yaml` under change control.
- Validate before apply to avoid lockouts.
```bash
netplan status
netplan generate
netplan apply
```

## Time Sync
```bash
timedatectl status
chronyc sources -v || true
```

## SSH and Identity
- Use non-root admin users and sudo.
- Prefer SSH keys; disable password auth when possible.
- Restrict SSH by source IP when feasible.
```bash
sshd -t
systemctl reload ssh
```

## Firewalling
- Deny inbound by default; allow only required ports.
- Align firewall rules with bind addresses.
```bash
ufw status verbose
ss -lntup
```

## Reduce Exposed Services
```bash
systemctl --type=service --state=running
systemctl disable --now <service>
```

## AppArmor
- Keep AppArmor enabled; move to enforce after validation.
```bash
aa-status
aa-enforce /etc/apparmor.d/<profile> || true
```

## systemd Hardening
```bash
systemd-analyze security <unit> || true
systemctl edit <unit>
```

## Updates and Patching
Choose one model and make it explicit:
- In-place updates with unattended security upgrades.
- Immutable image rebuilds (disable auto-updates).
- For major maintenance windows, prefer `apt dist-upgrade -o APT::Get::Always-Include::Phased-Updates=true` so phased updates are not silently skipped.
- If using unattended upgrades, explicitly set allowed origins (`-security`, optional `-updates`, and ESM channels when enabled).
```bash
apt-get update
apt-get -s upgrade
test -f /run/reboot-required && echo "reboot required" || true
```

## Release Upgrades
- Fully patch first and reboot if required.
- Expect third-party repos to be disabled during upgrade.
- Use `do-release-upgrade` for LTS-to-LTS upgrades; avoid `-d` for production unless explicitly testing pre-point releases.
```bash
do-release-upgrade -c
lsb_release -a || cat /etc/os-release
```

## Logging (journald)
```bash
journalctl --disk-usage
journalctl -u ssh --since "24 hours ago" || true
```

## Backups and Restore Drills
- Define what is backed up and the restore procedure.
- Test restores regularly.

## Observability
- Monitor patch status, reboot requirements, disk pressure, and critical daemons.
- For fleets, use centralized tooling (Landscape or equivalent).

## Compliance (USG/CIS/STIG)
- Run audits first.
- Apply remediations in staging before production.
- Track exceptions explicitly.

## Common Failure Modes
- Lockout from SSH/Netplan changes without validation.
- Password auth enabled on internet-facing SSH.
- Unattended upgrades colliding with manual APT operations.
- Backups never tested with a restore drill.

## References
- `references/ubuntu-best-practices.md`

## Definition of Done
- Baseline documented with owners and validation commands.
- Access controls and exposed ports are minimal and intentional.
- Patching and upgrade policy is explicit and tested.
- Backups and logging are configured with verification steps.
