# Ubuntu Server Best Practices

This document summarizes practical best practices for administering Ubuntu Server, with emphasis on the **latest current Ubuntu release (25.10, “Questing Quokka”)** and how to run it safely in **production**. As of **February 8, 2026**, Ubuntu **25.10** is a current release (released **October 2025**) with standard security maintenance until **July 2026**; Ubuntu’s release cadence is **every six months**, with “interim” releases supported for **9 months**. citeturn1view0turn2view0

For most real-world server estates, the recommended production baseline remains the current **LTS** (Ubuntu 24.04 LTS, “Noble Numbat”), because LTS releases have **5 years** of standard security maintenance, and can be extended via Ubuntu Pro (ESM) to **10 years**, and via the “Legacy add-on” to **15 years**. citeturn2view0turn8view0

## Release strategy and lifecycle posture

Choosing the right release is a risk-management decision: dependency stability, security maintenance windows, and your organization’s change tolerance matter more than “newest.” Ubuntu explicitly distinguishes **interim** releases (fast-moving, short support window) and **LTS** releases (stability and longer maintenance). citeturn2view0turn16view3

**Recommended default:**
- For **production**, run an **LTS** unless you have a strong reason to accept the operational churn of interim releases (e.g., you need a very new kernel/toolchain/hardware support). citeturn2view0turn16view3  
- For **development / pre-production / feature testing**, interim releases (like 25.10) can be appropriate because they deliver newer components earlier, with a predictable short lifecycle. citeturn2view0turn1view0

**Security maintenance is also tied to repository component coverage.** Ubuntu’s security maintenance differs across **Main/Restricted** vs **Universe/Multiverse**; the Ubuntu security documentation summarizes how interim vs LTS vs Ubuntu Pro (ESM) changes coverage, including the “15 years” path with ESM + Legacy. citeturn8view0turn2view0

**Cloud image upgrade posture is different by design.** Ubuntu’s cloud image guidance recommends launching a new image for the new release and migrating workloads/data, warning that in-place upgrades of cloud images are not recommended (because cloud image customization is typically applied during image creation). citeturn23search4

## Baseline installation and system configuration

A strong Ubuntu Server posture starts before first boot: verify what you install, minimize unnecessary surface area, and make configuration reproducible.

### Installation media integrity and provenance

When you install from ISO (bare metal, VM templates, offline environments), verify integrity and authenticity using Ubuntu’s published checksum and GPG signature guidance. Ubuntu’s security documentation describes GPG signing for installation media and the verification workflow (GPG + `sha256sum`). citeturn9search1turn9search5

### Automate builds for repeatability

For fleets and reproducible servers, prefer **automated installation** and **declarative initialization**:

- **Subiquity autoinstall** supports an autoinstall config that can include/merge **cloud-init user-data**. citeturn10search0  
- Cloud-init is designed to apply your initial configuration at boot so instances come up already configured, and it’s explicitly positioned as reusable for consistent results across machines. citeturn10search1  
- Cloud-init may deliver autoinstall configuration to the installer, but does **not** process autoinstall directives itself; it runs during install and first boot, then becomes inert on subsequent boots. citeturn10search7

### Networking: standardize on Netplan

Ubuntu Server uses **Netplan** for network configuration, defined via **YAML**, and rendered to either **systemd-networkd** or **NetworkManager**. citeturn19view0turn19view2

Operational best practice is to treat Netplan config as code:
- Keep `/etc/netplan/*.yaml` under change control (reviewable diffs, rollback plan).
- Validate changes and apply them deliberately (especially on remote systems) using Netplan’s tooling such as `netplan status` / `netplan apply`. citeturn19view1turn19view0  
- For temporary DNS adjustments, the networking guide references using `resolvectl` for transient settings and describes persistent configuration separately. citeturn19view1

### Time synchronization: know what changed in 25.10

As of **Ubuntu 25.10**, Ubuntu uses **chrony** for time synchronization and it is installed by default; timedatectl/timesyncd can still be used optionally. citeturn10search2turn10search6turn10search9

Time sync best practice is to use reliable sources and redundancy. Chrony’s upstream FAQ recommends multiple time servers (three or four as a typical minimum) to detect false time and improve robustness. citeturn10search15

### Storage protections: encrypt data at rest when theft or offline access is in-scope

Ubuntu’s security documentation describes **Full Disk Encryption (FDE)** as protecting data at rest on disk/partitions, especially relevant for devices at risk of theft/loss. citeturn10search3

For boot-chain integrity, Ubuntu documents **UEFI Secure Boot** as preventing untrusted code from executing during boot, with kernel signature enforcement noted for modern Ubuntu releases. citeturn11search0

### Configuration at scale: use configuration managers intentionally

Ubuntu Server documentation highlights configuration managers as a way to automate deployment/configuration for scalability and consistency, and gives examples:
- **Ansible**: agentless orchestration via SSH, often favored in small-to-medium estates. citeturn24view0  
- **Puppet**: agent-based client/server, often used in larger and more complex environments requiring detailed reporting. citeturn24view0

## Secure remote access and identity management

Your remote access model is one of the highest leverage security controls on a server. Ubuntu’s own security guidance emphasizes least privilege, firewalling, and SSH as the foundation for safe remote administration. citeturn3view0

### Least privilege and user lifecycle hygiene

Operate day-to-day as a non-root user and reserve privileged elevation for administrative tasks (principle of least privilege). citeturn3view0

When disabling an account, understand the difference between passwords and key-based access: Ubuntu’s user management documentation warns that disabling/locking a user password does **not** prevent SSH login if the user already has authorized keys configured; you must also review/remove SSH key material (e.g., `~/.ssh/authorized_keys`) if your goal is access removal. citeturn17view0

### SSH configuration: prefer drop-in snippets and validate changes

Ubuntu’s OpenSSH server documentation recommends configuration via `/etc/ssh/sshd_config` or modular snippets in `/etc/ssh/sshd_config.d/`, noting that Ubuntu includes an `Include /etc/ssh/sshd_config.d/*.conf` line at the top and that (for most directives) the first set value is used, so snippet-defined settings can override main config. citeturn17view1

Treat SSH changes as “high risk of lockout” changes:
- Validate config with `sshd -t` before restarting the service. citeturn17view1  
- Restart with systemd only after validation. citeturn17view1

OpenSSH’s `sshd_config` reference describes server-side controls like `PermitRootLogin` (including modes such as prohibiting password-based root login). citeturn7search3

A practical “safer-by-default” SSH posture (adapt to your needs) is:
- Key-based authentication as the default.
- Password authentication disabled for remote login unless you explicitly need it.
- Root remote login disabled or restricted (e.g., prohibit password and require key, or disallow entirely).
- Limit who can SSH in (source IP restrictions at the firewall and/or SSH allowlists), and log authentication events.

These patterns are consistent with Ubuntu’s emphasis on SSH for remote access plus least privilege. citeturn3view0turn17view1turn7search3

### Multi-factor authentication for SSH

Ubuntu Server includes documented approaches for SSH 2FA. The TOTP/HOTP guide recommends hardware authenticators supporting U2F/FIDO for best 2FA security; when using TOTP/HOTP, Ubuntu documents a configuration where the first factor is public key auth and the second factor is the OTP, with password auth unavailable. citeturn17view2

## Updates, upgrades, and supply chain hygiene

### How Ubuntu ships security updates

Ubuntu is a fixed-release distribution and provides security updates as **backported patches** during the support window; the intent is stability and backward compatibility rather than introducing new functionality via security updates. citeturn8view0

Ubuntu documents how security updates are delivered via archive “pockets” and how Server users receive update notifications via MOTD (`update-notifier-common` on modern releases). citeturn8view0

Ubuntu also publishes:
- Vulnerability tracking via the **Ubuntu CVE Tracker**.
- Security update announcements via **Ubuntu Security Notices (USNs)**. citeturn8view0  
- Machine-readable vulnerability/update data feeds (OVAL, OSV, VEX) for integration with scanners and automated management tooling. citeturn8view0

### Daily patching: unattended-upgrades and its operational tradeoffs

Ubuntu Server applies security updates automatically via `unattended-upgrades`, which the server documentation states is installed by default; it can also reboot the system if configured, apply non-security updates if allowed, and block specific packages/origins. citeturn4view0turn3view0

Ubuntu documents the key configuration layout:
- `/etc/apt/apt.conf.d/50unattended-upgrades` for behavior options (reboot scheduling, package blacklists, allowed origins).
- `/etc/apt/apt.conf.d/20auto-upgrades` for enabling and frequency.
- Detailed logs in `/var/log/unattended-upgrades/`. citeturn4view0

Ubuntu also documents that the work is triggered by systemd timers (`apt-daily.timer`, `apt-daily-upgrade.timer`) and that missed timer events can run on startup, which can block other package operations; it provides guidance on overriding timer behavior when that startup behavior is counterproductive for your environment. citeturn4view0

For production operations, the key best practice is to decide which of these models you are using and make it explicit:
- “Update in place daily” (unattended security updates, controlled reboots).
- “Redeploy with updated images” (immutable/ephemeral systems), which Ubuntu explicitly discusses as a scenario where automatic updates may be disabled, provided your deployment pipeline is where patching happens. citeturn4view0turn23search4

When customizing unattended-upgrades, Ubuntu security documentation discourages editing the original configuration directly and recommends drop-in files with higher lexicographic order so your changes survive upgrades cleanly. citeturn8view0

### Phased updates: treat “kept back” as a safety mechanism, not an error

Ubuntu documents “phased updates” in APT: updates roll out to subsets of machines first, expanding as stability is proven. Ubuntu explains this reduces the chance that a broken update impacts everyone at once, and advises leaving phased updates enabled unless you intentionally want to be in the earliest cohort (for testing). citeturn16view1

In production estates, a common best practice is:
- Leave phased updates on for most machines.
- If you need earlier access for validation, target a canary ring or staging environment explicitly designed for early uptake. Ubuntu’s own explanation frames early uptake as effectively volunteering to detect breakage, which is best done intentionally, not accidentally. citeturn16view1

### Pre-release validation: test proposed updates before they land

Ubuntu documents an enterprise-friendly pattern for advance testing: proposed updates are typically available publicly for at least a week before release, and Ubuntu encourages automated testing against proposed updates so regressions can be detected before broad rollout. citeturn16view2

It also warns that “upgrade everything in proposed” is generally unsafe and recommends recreating test environments after use to avoid carrying forward inconsistent pockets. citeturn16view2

### Release upgrades: plan carefully and avoid surprise major jumps

Ubuntu Server documentation recommends `do-release-upgrade` for Server/cloud images because it handles cross-release config changes; it also documents that you cannot skip LTS releases (upgrades must be sequential). citeturn16view3

The documented pre-upgrade checklist emphasizes:
- Review release notes.
- Fully update your current system (including handling phased packages that could block release upgrade).
- Reboot if required (`/run/reboot-required`).
- Ensure sufficient disk space and planned maintenance time.
- Expect third-party repositories/PPAs to be disabled during upgrade and plan re-enablement afterwards.
- Take backups before proceeding. citeturn16view3

### Ubuntu Pro, ESM, and Livepatch: shorten exposure windows and extend support

Ubuntu’s lifecycle documentation explains that Ubuntu Pro adds security and support services, including up to **15 years** of security coverage (via ESM + Legacy add-on), plus tools like Livepatch and compliance/hardening offerings. citeturn2view0

The Pro Client documentation explains:
- `esm-infra` extends security updates for “main” packages beyond the standard window.
- `esm-apps` provides longer security coverage for “universe” packages. citeturn12view1turn8view0

Livepatch is documented as applying fixes for high/critical kernel vulnerabilities while the system runs, so you can schedule reboots rather than reboot immediately after kernel updates; Ubuntu Pro Client docs show how it is enabled and how to check status. citeturn12view2turn3view0turn2view0

### Supply chain hygiene: use trust chains, don’t bypass them

Ubuntu’s security documentation stresses that archive authenticity is protected via cryptographic signatures generated independently of distribution infrastructure, and that APT automatically verifies integrity and authenticity of downloaded packages using those signatures. citeturn9search0turn9search17

A practical best practice is to avoid bypassing APT’s safeguards:
- Prefer installing software via APT (official repositories) rather than downloading `.deb` files directly, because APT’s chain-of-trust checks are part of the standard safety model. citeturn9search17turn9search0  
- Minimize use of third-party repositories; Ubuntu’s server documentation warns they can be harder to diagnose and can lead to serious problems (including service unavailability), so treat them as exceptions that require prior research and tight controls. citeturn3view0turn16view0  

For controlled rollback/reproducibility, Ubuntu provides the **Ubuntu snapshot service**, enabling access to the Ubuntu archive as it existed at a specific time and supporting staged rollouts and known-good upgrade states. citeturn24view1

## Host and network hardening controls

Ubuntu “hardening” is most effective when it is layered: reduce exposed services, restrict access, confine processes, and keep the platform patched.

### Minimize exposed services and open ports

Ubuntu security documentation defines an “open port” as a port bound to a service actively listening, and documents Ubuntu’s long-standing “No Open Ports” policy: by default, a new installation should have no listening network services, with rare exceptions. citeturn30view0

Ubuntu provides concrete guidance for identifying open ports using `ss`, including filtering loopback-only listeners. citeturn30view0turn31view0

When open ports are required (which is normal for servers), Ubuntu’s security documentation recommends:
- Disable network services no longer required and prevent them from starting automatically.
- Avoid binding to wildcard/public addresses unless necessary; bind to specific interfaces/addresses or loopback where appropriate.
- Use firewalls to control reachability of open ports.
- Keep software up to date to reduce vulnerability exposure. citeturn31view0

Ubuntu explicitly notes that `systemctl disable` may not be sufficient if a service is started as a dependency of other enabled services—so “service reduction” needs dependency awareness. citeturn31view0turn15search0

### Firewalling: UFW, nftables, and practical access control

Ubuntu Server documentation presents `ufw` as the default firewall configuration tool and notes it is initially disabled; it shows how to enable/disable it and manage allow/deny rules and source-limited rules. citeturn17view3

The `ufw(8)` manual documents that (on installation) UFW is disabled and sets default policies including **deny incoming**, **deny forward**, and **allow outgoing** (with stateful tracking behavior). citeturn27search1

Ubuntu’s security documentation also explains that `ufw` can act as a frontend for both `iptables` and `nftables`, and separately documents `nftables` as the modern successor to legacy `iptables` tooling in the netfilter ecosystem. citeturn9search3turn9search7

Operational best practice is to align firewalling to your threat model:
- “Internet-facing service”: deny by default, allow only necessary ports, consider source restrictions for administration ports.
- “Internal-only service”: bind to private interfaces and only allow internal subnets that need access, consistent with Ubuntu’s guidance to avoid wildcard/public binds when not required. citeturn31view0turn17view3turn27search1

### Mandatory access control: AppArmor as a default safety net

Ubuntu Server documentation describes AppArmor as a Linux Security Module providing mandatory access control (MAC) on top of traditional discretionary access control, and notes that on Ubuntu it is installed and loaded by default (checkable via `aa-status`). It also documents “complain” vs “enforce” modes and the tooling to manage profiles. citeturn18view0

Best practice is to:
- Keep AppArmor enabled and run key services under enforced profiles where feasible.
- Use complain mode to develop profiles safely, then transition to enforce once validated. citeturn18view0turn3view0

### Kernel/network safeguards: SYN cookies

Ubuntu documents SYN cookies as a protection against TCP SYN flood attacks, implemented in the kernel and controlled via the `net.ipv4.tcp_syncookies` sysctl parameter (applying to both IPv4 and IPv6). citeturn14search1turn31view0

### Packaging security: snaps, confinement, and “trusted publishers”

Ubuntu’s lifecycle documentation explains that Ubuntu supports both Debian packages (“debs”) and snaps, and that snaps can run confined with limited access or in classic mode with broader access; it recommends choosing snaps from trusted publishers, especially when using classic confinement. citeturn2view0

Snapcraft documentation explains that strict confinement uses kernel security features (including AppArmor, seccomp, namespaces) to isolate applications, while Canonical’s snap security documentation notes that classic confinement doesn’t benefit from snapd’s sandboxing and therefore has fewer security boundaries. citeturn11search12turn11search16

### Service-level hardening: use systemd sandboxing where it fits

Systemd provides an ecosystem of unit-level sandboxing settings (documented in `systemd.exec`) that can reduce service privileges and filesystem exposure. citeturn15search2turn21search15

For auditing what you already have, `systemd-analyze security` is documented (including in Ubuntu manpages) as analyzing the sandboxing/security settings for specified service units, or all long-running services if no unit is specified. citeturn21search1

A pragmatic best practice is to:
- Use `systemd-analyze security` as a discovery tool (not as a guarantee of overall app security).
- Apply systemd sandboxing selectively where it doesn’t break application behavior, and capture these changes as drop-in unit overrides so they are maintainable across updates. citeturn21search1turn15search2turn15search0

### Compliance automation and benchmarks

For regulated environments, Ubuntu provides compliance automation via the Ubuntu Security Guide (USG), including audit/remediation workflows aligned to entity["organization","Center for Internet Security","cis benchmark publisher"] Benchmarks and DISA-STIG profiles, and documents how to apply CIS benchmark rules using `usg fix <PROFILE>`. citeturn9search2turn9search23turn9search15

## Operational excellence: logging, backups, monitoring, and fleet management

Security and reliability degrade quickly when operational basics are missing: you need logs you can trust, backups you can restore, and visibility into system behavior.

### Logging: configure retention intentionally and preserve signals

Systemd-journald stores logs either persistently on disk (under `/var/log/journal`) or in volatile storage (under `/run/log/journal`), losing volatile logs on reboot; the journald documentation explains that default persistence depends on whether `/var/log/journal` exists, and can be controlled via `Storage=` in `journald.conf`. citeturn29search12turn29search3

The `journald.conf` reference documents size/retention controls including `SystemMaxUse=` / `RuntimeMaxUse=` and related “keep free” settings, which you should set according to incident response needs and disk capacity. citeturn29search1turn29search3

For operational usage, `journalctl` is the standard interface for reading journald logs, and is documented as printing log entries stored by systemd-journald. citeturn29search18

Best practice outcomes to aim for:
- Persistent logs on systems where post-reboot forensics matter.
- Explicit disk caps so logs don’t crowd out application storage.
- A plan for centralization (forwarding/aggregation) if the machine is not the ultimate source of truth. citeturn29search12turn29search1

Also treat update logs as first-class operational signals: Ubuntu documents unattended-upgrades logs in `/var/log/unattended-upgrades/`, which should be monitored for failures in any environment relying on automatic patching. citeturn4view0turn8view0

### Backups: plan, practice restores, and keep copies off-site

Ubuntu Server documentation emphasizes that backup planning should define what is backed up, frequency, storage location, and restore procedures, and recommends off-site storage for important backup media. citeturn20view0

It also documents two broad approaches:
- Backup utilities (e.g., Bacula, rsnapshot) and their typical fit, including incremental backups and remote snapshots over SSH.
- Script-based approaches for flexibility. citeturn20view0

A practical best practice is to include configuration in your resiliency plan: Ubuntu’s backups guide calls out version control approaches (e.g., storing `/etc` in a VCS via “etckeeper”), which reduces recovery time and configuration drift risk. citeturn20view0

### Observability and monitoring: align tools with your future Ubuntu baseline

Ubuntu Server’s observability documentation notes that the “classic” LMA stack still works but is being succeeded by the Canonical Observability Stack (COS), and that the observability section will be deprecated for Ubuntu 26.04 LTS and onward—so production teams should avoid building a long-lived strategy solely around deprecated guidance. citeturn24view2turn22search5

Even when you keep tooling lightweight, best practice is to monitor:
- Patch status and reboot requirements (especially for kernel/userland security updates).
- Disk pressure and log growth.
- Critical daemons (SSH, networking, storage) and service availability. citeturn4view0turn29search1turn31view0

### Fleet management: don’t confuse unattended-upgrades with governance

Ubuntu explicitly states `unattended-upgrades` is helpful but not intended to replace fleet management tooling for large deployments. citeturn4view0

For large estates, Ubuntu Pro Client documentation describes Landscape as a management/admin tool capable of managing large numbers of Ubuntu machines and automating patching/hardening/compliance. citeturn12view1

### Practical checklists

**Baseline hardening checklist (applies to any server role)** — prioritize correctness over completeness:
- Verify installation artifacts (ISO/image integrity) before first boot. citeturn9search1turn9search5  
- Ensure time sync is active and stable (chrony by default on 25.10). citeturn10search2turn10search15  
- Minimize running services and open ports; identify listeners with `ss`, disable unneeded services, avoid wildcard/public binds unless required. citeturn30view0turn31view0  
- Configure SSH using snippets, validate with `sshd -t`, and enforce least-privilege access patterns. citeturn17view1turn3view0turn7search3  
- Keep AppArmor enabled; prefer enforce mode once profiles are validated. citeturn18view0turn3view0  
- Decide your patching model (unattended security updates vs redeploy), and monitor the relevant logs/outcomes. citeturn4view0turn23search4turn8view0  

**Production readiness checklist (what usually separates “works” from “operable”)**
- Document your release cadence (LTS baseline + planned upgrade window) and ensure repository coverage matches your dependency footprint (Main vs Universe, Pro/ESM decisions). citeturn2view0turn8view0  
- Establish staged validation: canary/staging, and (if needed) testing against proposed updates before wide rollout. citeturn16view2turn16view1  
- Ensure logs are persistent where needed, sized appropriately, and (ideally) centrally aggregated. citeturn29search12turn29search1  
- Implement backups with a restore procedure you have tested; store critical backups off-site. citeturn20view0  
- Treat third-party repositories as controlled exceptions with explicit risk review. citeturn16view0turn3view0  
- If you need extended security windows or compliance automation, formalize Ubuntu Pro/ESM/USG usage and operationalize the tooling (`pro status`, Livepatch posture, CIS/STIG auditing). citeturn2view0turn12view1turn12view2turn9search2turn9search23