# CLAUDE.md — homelab-soc (Project Sentinel)

> Persistent instructions Claude Code loads every session. Keep this **durable and short**
> (< ~180 lines). Procedures live in `@docs/ROADMAP.md`; deep context in
> `@HOMELAB_PROJECT_CONTEXT.md`; decisions in `@docs/architecture/decisions/`.
> **Last reviewed:** 2026-06-13

## Project
Re-architecting a live 6-service Proxmox homelab into a **portable, zero-inbound security +
DevOps platform**: segmented SOC lab (Wazuh SIEM, Suricata IDS, attack/detect) + a self-hosted
dev platform (Forgejo, CI/CD, Supabase) + outbound-only remote access (Tailscale, Cloudflare
Tunnel). It is a résumé project — optimize for **legible, defensible, secure**, not just working.

## Who you're working with
**Brandon Granillo** — junior, Computer Systems Engineering, ASU. Learning much of this as he
goes, with heavy Claude Code use. So: **explain the *why*, define jargon on first use, give one
clear step at a time, and never ship something he can't explain.** Every generated artifact gets
a one-line `# why` comment. He works hands-on and confirms via screenshots.

## Reference map (load these for full context)
- @HOMELAB_PROJECT_CONTEXT.md — verified hardware, IPs, versions, gotchas
- @docs/ROADMAP.md — step-by-step build plan (M0–M4 + Phase 2)
- @homelab-platform-PRD.md — scope, goals, success metrics, résumé framing
- @docs/architecture/decisions/ — ADRs (the "why" behind decisions)
- @.claude/rules/ — deeper per-domain rules (security.md, iac.md) if present

## Environment (durable facts)
- Proxmox 9.1.1, node `bgranillo`, `192.168.0.13:8006`. Home uplink Cox CGM4331COX (`192.168.0.1`) blocks most inter-device ports except SSH/22.
- pfSense VM **200**, LAN `10.10.10.1`, default-deny across 5 VLANs (10 SERVICES / 20 TRUSTED-MGMT / 30 VULNERABLE / 40 MONITORING / 50 PUBLIC-DMZ).
- 6 service LXCs on `10.10.10.100–105`. Wazuh VM **201** @ `10.10.40.10`. Vuln target @ `10.10.30.51`.
- **No inbound ports are ever available** (blocked ISP now, shared Wi-Fi later). All remote access is outbound-only: Tailscale (private), Cloudflare Tunnel (public).
- Reach pfSense: `ssh -L 9443:10.10.10.1:443 -N root@192.168.0.13`. The noVNC console mangles typed URLs — use its clipboard-paste panel or SSH.

## Repo layout
`docs/architecture/` (overview, network-design, ADRs) · `docs/runbooks/` · `docs/journal/` (dated
log) · `configs/` (sanitized only) · `detections/` (attack writeups) · `diagrams/` · `terraform/`
· `ansible/` · `.gitleaks.toml` · `.gitignore`.

## Conventions (always do)
- **Snapshot first.** Before any risky change, take a Proxmox snapshot; name it meaningfully.
- **Validate after.** Every change includes how to confirm it worked and how to roll it back.
- **Document as you go.** Append to `docs/journal/<date>.md`; update `CHANGELOG.md` and `docs/LINKS.md`.
- **One ADR per significant decision.** Immutable, numbered; supersede rather than delete.
- **IaC by default.** Prefer Terraform (`bpg/proxmox`, Cloudflare provider) + Ansible over clicking; commit it.
- **Explain before applying.** State what a change does and why before running it. No black boxes.
- **Specific & verifiable instructions.** Exact commands/IPs/ports, not vague guidance.

---

## 🔒 SECURITY SPINE (highest priority — overrides convenience)

This project's whole point includes "no leaks or exposures." These rules are non-negotiable.

### Secrets — never in the open
- **NEVER write a secret into any committed file, this CLAUDE.md, a doc, a journal entry, a commit message, a screenshot, or chat output.** Not passwords, API keys, tokens, JWT secrets, private keys, `age`/SSH keys, DB credentials, or recovery codes.
- Secrets live **only** in: a password manager (human-held), **SOPS/age** (encrypted in Git), or **Vault/OpenBao** (runtime). Reference secrets by *name/path*, never by value.
- Any machine-local or sensitive pointer goes in **`CLAUDE.local.md`**, which **must be gitignored** (add it to `.gitignore`). Never commit it.
- Generated configs reference secret *placeholders* (`${DB_PASSWORD}`), never literals.
- If a secret is ever exposed (committed, pasted, screenshotted): **treat it as compromised** — rotate it immediately, then scrub it (incl. Git history). Don't just delete the latest copy.
- `.gitleaks.toml` already denies the hardware serials and defaults; **`gitleaks protect --staged` must pass before every commit.** If it fails, stop and fix — never `--no-verify`.

### Exposure — least privilege, zero inbound
- Default-deny between VLANs stays the baseline. New allow rules are **single host + single port**, justified in the rule description and an ADR.
- Public exposure is **only** via Cloudflare Tunnel from VLAN 50 (DMZ), fronted by Cloudflare Access where possible. **Never** suggest opening an inbound port or DMZ'ing the host.
- Internet-facing workloads live in VLAN 50, isolated from SERVICES — never co-locate public apps with personal data.
- ntfy, Vault, code-server, dashboards: **private over Tailscale only**, never tunneled public.

### Data — don't risk what can't be replaced
- **HARD GATE:** Nextcloud does not become primary/irreplaceable storage until a ZFS mirror **and** a tested 3-2-1 backup **and** a *verified restore* exist. Refuse to migrate primary data before that.
- Back up (snapshot/vzdump/PBS) before destructive or migration steps.

### Human-in-the-loop — stop and get Brandon's explicit OK before:
- Deleting data, volumes, snapshots, or VMs/containers (anything irreversible).
- Changing firewall rules, exposure, or anything that could open an attack surface.
- Publishing/posting anything public (a tunnel hostname, a repo going public, a deploy users hit).
- Handling real secret **values** — Brandon enters/rotates them himself; you only reference names and write the scanning/encryption plumbing.
- Modifying access controls, sharing permissions, or auth config.
- Editing Git **history** or force-pushing.
State the action, its blast radius, and the rollback, then **wait for confirmation.** When unsure, ask — never assume authorization.

### When troubleshooting
- Saying less is safer. If asked to print logs/config that may contain secrets, **redact** first.
- Never echo a secret back "to confirm it." Confirm by *name* and by whether a check passed.
- Reproduce the issue, isolate, fix the smallest thing, verify, document — don't shotgun changes on production-adjacent services.

---

## Anti-patterns (never do)
- `git commit --no-verify` / bypassing gitleaks · committing `configs/*-raw.xml` or backups (gitignored) · pasting unredacted screenshots with creds/serials · "temporary" plaintext secret files · port-forwarding or UPnP as a remote-access answer · applying AI-generated IaC without reading it · making this CLAUDE.md a long procedure manual (put procedures in ROADMAP/runbooks).

## Definition of done (per change)
Works **and** verified **and** rolled-back-path known **and** documented (journal + CHANGELOG + LINKS) **and** gitleaks clean **and** Brandon can explain it.
