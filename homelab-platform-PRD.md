# PRD — Portable Self-Hosted Security & Dev Platform ("Project Sentinel")

| | |
|---|---|
| **Author(s)** | Owner (builder) · drafted by Sr. PM + FAANG Hiring-Manager review lens |
| **Status** | Approved for build — v1.0 |
| **Last updated** | 2026-06-13 |
| **Target milestone** | Résumé-ready core in **~3 months** (bursty effort) |
| **Repo** | `homelab-soc` (public, GitHub) — extend in place |

> **How to read this doc.** Two lenses are baked in. The **PM lens** defines scope, sequencing,
> metrics, and risk. The **Hiring-Manager (HM) lens** — flagged as **🎯 HM** — judges whether each
> choice produces a defensible, S-tier résumé signal. Where they conflict, the doc says so.

---

## 1. TL;DR

Evolve a live, segmented Proxmox SOC lab into a **portable, zero-inbound, self-hosted security +
DevOps platform** that runs behind hostile shared apartment Wi-Fi, is reachable from anywhere via
outbound tunnels, and is reproducible from Git. The build serves three career targets at once —
**Security/SOC**, **DevOps/SRE/Platform**, **Network Engineering** — and is sequenced so a complete,
demoable, *defensible* result exists within ~3 months on existing hardware (~$70 spend), with a
clearly-deferred Phase 2 for bulk storage and orchestration.

**The one-line pitch (résumé headline):**
> *"Designed and operated a reproducible, zero-trust home infrastructure — segmented SOC lab
> (SIEM + IDS + attack/detect), a self-hosted DevOps platform with a secure software supply chain,
> and a portable network that runs behind adversarial shared Wi-Fi with no inbound ports."*

---

## 2. Context & current state

A real, running 6-service Proxmox homelab is being hardened in place — not a greenfield tutorial.
This "re-architected a live system" framing is a deliberate strength.

**Done (verified):**
- **Phase A:** backups, ~54 GB reclaimed, GitHub repo scaffolded (ADRs, runbooks, journal, CHANGELOG, gitleaks).
- **Phase B:** pfSense VM, 4 VLANs (SERVICES/TRUSTED-MGMT/VULNERABLE/MONITORING), default-deny firewall, all 6 LXCs migrated to VLAN 10.
- **Platform:** Lenovo M93p SFF (i7-4790, 16 GB, 512 GB SSD), Proxmox VE 9.1.1.

**Constraints (from scoping):**
- **Budget:** ~$100–300 over ~6 months → stay on the SFF; defer the tower.
- **Time:** bursty → milestone-gated, atomic, resumable stages.
- **Deadline:** applying in ~3 months → front-load the near-done, high-signal work.
- **Skill:** learning as-you-go with heavy Claude Code assistance → runbooks assume no fluency; every stage has a *learning gate*.
- **Network:** **strictly shared public Wi-Fi**, no private line, ever → travel router + outbound tunnels are mandatory; **no port-forwarding is possible**.

---

## 3. Goals & non-goals

### Goals
1. **Three résumé signals from one project** — one defensible bullet each for Security, DevOps/Platform, and Networking.
2. **A self-hosted DevOps platform** (builder's #1 priority): Git + CI/CD + container registry + remote dev + app deployment for React Native/Expo backends and hackathons.
3. **Complete the SOC arc** (#2): Wazuh SIEM, Suricata IDS, 5 documented attack→detect exercises.
4. **Portable, zero-inbound access** (#3): private admin via Tailscale; selective public exposure via Cloudflare Tunnel — both survive the apartment move.
5. **Reproducible from Git** (IaC) and **defensible in an interview** (learning gates).
6. **Operate it like production**: measured SLO, low MTTR, phone alerting, tiered backups.

### Non-goals (explicitly out of scope for v1)
- **Bulk media storage / Nextcloud-as-primary** — ranked last; deferred to Phase 2 (needs drives + tested backups first).
- **High availability / clustering** — single node; honest SLO instead of fake 99.99%.
- **A second machine ("Vault" tower)** — Phase 2, budget-gated.
- **External app users at scale / paid SaaS** — friends/family + transient hackathon judges only.
- **Anything requiring inbound ports** — architecturally impossible behind shared Wi-Fi; designed around.

---

## 4. Users & personas

| Persona | Who | Needs |
|---|---|---|
| **Builder/Operator** | You | Build with Claude Code, learn by doing, defend every choice, work remotely from any device |
| **Consumers** | A few friends/family | Simple, secure access to media/files (Phase 2) |
| **Transient public** | Hackathon judges, app testers | Hit a public URL/API reliably for short windows |
| **🎯 The real audience** | FAANG recruiters & interviewers | Encounter it via GitHub + writeup; probe it in the loop |

> **🎯 HM note:** The fourth persona is the one that pays. Every deliverable is also an artifact this
> persona will read or interrogate. Optimize for *legible* and *defensible*, not just *working*.

---

## 5. Success metrics

### Product / résumé KPIs
- **≥ 3 distinct, quantified résumé bullets** shipped (one per target track). *(see §13)*
- **Demoable in < 5 min**: live Tailscale walkthrough + one public Cloudflare URL + the GitHub repo.
- **100% defensibility**: every architecture decision has an ADR + you pass its learning gate.
- **Reproducibility**: core environment rebuildable from Git via Terraform + Ansible (target MTTR < 1 hr).

### Technical KPIs
- Default-deny verified between all zones (documented test).
- **5/5** simulated attacks executed and written up, *including detection gaps*.
- CI pipeline enforces **≥ 4 security gates** (secrets, SAST, deps, container/IaC) with SBOM + signed artifacts.
- Uptime **measured** (Uptime-Kuma) against a stated SLO (~95–99% at home base); MTTR tracked.
- All critical alerts reach your **phone** (ntfy) within seconds.

> **🎯 HM note:** Notice these are *measured*, not aspirational. "I set an SLO and tracked MTTR"
> is a senior signal; "it's always up" on one SFF is a tell that you've never operated anything.

---

## 6. 🎯 What makes this S-tier (hiring-manager lens)

A typical student homelab = "I installed Plex and a Pi-hole." Yours clears the bar because it stacks
**five** differentiators most candidates never touch:

1. **Re-architected a *live* system** — risk management, backups-before-changes, real migration sequencing (you already did this in Phase A/B).
2. **Adversarial network design** — built for *hostile shared Wi-Fi* with **zero inbound ports**; your own firewall isolating you from strangers. Most homelabs assume a friendly router. This is a genuine network-security thesis.
3. **Secure software supply chain** — the intersection of your #1 (dev) and #2 (security): a CI/CD pipeline that *scans itself*. Rare even among working engineers.
4. **Infrastructure-as-Code + AI-assisted, but understood** — reproducible from Git, built fast with Claude Code, and *defensible* line-by-line. Using AI tooling well is a 2026 positive signal — framed correctly.
5. **Operated like production** — measured SLO, MTTR, tiered 3-2-1 backups, phone on-call alerting, observability.

**The narrative that ties it together (use this in interviews):**
> "I treated my home lab as a product with an SLO and a threat model, not a toy. It runs behind a
> network I don't trust, deploys my own apps through a pipeline that enforces security gates, and I
> can rebuild the whole thing from Git."

---

## 7. Requirements

### 7.1 Functional

**Dev platform (P0 — builder's #1)**
- Self-hosted Git (Forgejo/Gitea) with private repos, issues, **built-in container registry**.
- CI/CD (Forgejo/Gitea Actions, GitHub-Actions-compatible) for build/test/deploy.
- **Remote dev environment** (code-server) — code in-browser from any device.
- Self-hosted **app backends**: Postgres + **Supabase** (Firebase alternative; reusable for Expo apps + hackathons).
- **Public exposure** of selected apps/APIs via Cloudflare Tunnel (+ Access for auth).
- Deployment target: **Docker Compose/Portainer now → k3s in Phase 2** (orchestration signal).

**Security / SOC (P0 — #2)**
- Wazuh SIEM (single-node) with agents on key hosts.
- Suricata IDS on pfSense (ETOpen) → EVE JSON → Wazuh.
- 5 attack→detect exercises (Nmap, Hydra, Metasploit, SQLi, C2 callback) documented with gaps.
- **Secure CI/CD pipeline** (the crown jewel): gitleaks → Trivy (deps+image+IaC) → Semgrep (SAST) → checkov (IaC) → Syft (SBOM) → cosign (signing).
- **Secrets management**: gitleaks (prevent) + SOPS/age (encrypt-in-Git) + Vault/OpenBao (runtime).

**Networking / access (P0 — #3)**
- 4 existing VLANs **+ a new PUBLIC/DMZ zone** for internet-exposed (tunneled) workloads, isolated from SERVICES.
- Travel router (WISP/client mode) → pfSense WAN (dedicated NIC port).
- **Tailscale** subnet-router (private admin/dev from anywhere).
- **Cloudflare Tunnel** (public, outbound-only) + **Cloudflare Access** (zero-trust auth).

**Operations (P1)**
- Phone alerting via self-hosted **ntfy** (Wazuh, Uptime-Kuma, PBS, CI all route here).
- Backups: **PBS** for VMs/CTs; **restic → Backblaze B2** (encrypted) for irreplaceable data.
- IaC: **Terraform** (Proxmox + Cloudflare providers) + **Ansible** — whole core reproducible from Git.

### 7.2 Non-functional
- **Availability:** best-effort always-on at home base; stated SLO ~95–99%; MTTR < 1 hr via IaC rebuild. Must-not-fail demos → ephemeral cloud, not the lab.
- **Security posture:** default-deny between zones; zero inbound ports; least-privilege; no plaintext secrets in Git.
- **Portability:** entire stack survives a physical move + reconnect behind a different shared Wi-Fi with no re-architecture.
- **Capacity:** must fit **32 GB** with disciplined on-demand VMs (see §9.3). Bulk storage is Phase 2.
- **Maintainability:** every generated artifact carries a one-line "why"; jargon defined; runbooks assume no fluency.

---

## 8. System architecture (target)

```
                Internet — apartment SHARED PUBLIC Wi-Fi (untrusted)
                                   │
                          ┌────────┴─────────┐
                          │  Travel router   │  WISP/client mode →
                          │  (your firewall  │  private wired uplink you own;
                          │   #0, isolates   │  handles captive portal
                          │   you from bldg) │
                          └────────┬─────────┘
                                   │ (wired → pfSense WAN, dedicated NIC port)
                          ┌────────┴─────────┐
                          │  pfSense + Suricata IDS  │  default-deny, 4+1 VLANs
                          └─┬───────┬────────┬───────┬────────┬──┘
                          V10     V20      V30      V40      V50 (NEW)
                        SERVICES TRUSTED  VULNERABLE MONITOR  PUBLIC/DMZ
                           │      -MGMT      │        │         │
                  ┌────────┴───┐         ┌───┴───┐  ┌─┴────┐  ┌─┴──────────┐
                  │ 6 LXCs +   │         │ Kali +│  │Wazuh │  │ cloudflared │
                  │ dev platf. │         │ target│  │ SIEM │  │ + public    │
                  │ Forgejo/CI │         │(on-   │  └──────┘  │ app/API     │
                  │ Supabase   │         │ demand│            │ surface     │
                  │ code-server│         │ only) │            └─────────────┘
                  │ Vault,ntfy │         └───────┘
                  └────────────┘

  OUTBOUND-ONLY ACCESS (no inbound ports, ever):
    • Tailscale subnet-router  → private admin/dev from any device, anywhere
    • Cloudflare Tunnel+Access → selected PUBLIC services (hackathon demos, app APIs)

  All on ONE Proxmox node (M93p SFF, 32 GB). "Vault" tower + bulk storage = Phase 2.
```

---

## 9. Scope & capacity

### 9.1 In scope (v1 — the 3-month core)
Dev platform (Forgejo + CI + registry + code-server + Supabase + Docker) · secure CI/CD pipeline ·
SOC completion (Wazuh, Suricata, 5 attack/detect) · Tailscale + Cloudflare Tunnel · new PUBLIC VLAN ·
SOPS/age + Vault · ntfy alerting · PBS + restic offsite for *configs/repos* · IaC (Terraform+Ansible) ·
domain purchase · RAM→32 GB + dual NIC · polished writeup.

### 9.2 Phase 2 (deferred — post-résumé, budget/time-gated)
"Vault" tower + ZFS mirror · **Nextcloud-as-primary + Jellyfin bulk media** · **k3s** migration ·
Grafana/Prometheus · **Authentik SSO** · the physical apartment move · offsite backup of bulk data.

### 9.3 Capacity plan (32 GB RAM budget — *show this to interviewers*)
| Workload | Always-on | ~RAM |
|---|---|---|
| Proxmox host | ✓ | 1.5 GB |
| 6 existing LXCs (Pi-hole, Nextcloud, Jellyfin, Uptime-Kuma, Portainer, Homepage) | ✓ | 3 GB |
| pfSense | ✓ | 1 GB |
| Wazuh (tuned single-node) | ✓ | 4 GB |
| Forgejo + Postgres + CI runner | ✓ | 2 GB |
| code-server | ✓ | 1 GB |
| Vault/OpenBao + ntfy + Tailscale + cloudflared | ✓ | 1.5 GB |
| **Always-on subtotal** | | **~14 GB** |
| Supabase (full stack) | on-demand | 2.5 GB |
| Kali + vulnerable target | exercises only | 2–4 GB |
| Headroom (Phase-2 Grafana, bursts) | | ~10 GB |

> **🎯 HM note:** This table *is* the interview answer to "how did you manage constrained resources?"
> Capacity planning + on-demand scheduling on a fixed budget is exactly the muscle SRE/Platform roles test.

---

## 10. Roadmap (milestone-gated, resumable)

Each milestone is atomic, ends in a working+documented state, and closes with a **learning gate** —
things you must be able to explain in your own words before advancing. Front-loaded so the near-done,
highest-signal work lands first and de-risks the deadline.

### M0 — Foundation *(first heavy burst; ~$70 + $12 domain)*
- RAM → 32 GB; install dual-port NIC; buy domain (Cloudflare Registrar).
- Scaffold IaC repo (Terraform + Ansible skeleton); stand up Tailscale subnet-router + self-hosted ntfy.
- **Gate:** explain why 32 GB matters here, what a subnet-router does, why outbound tunnels beat port-forwarding.

### M1 — Finish the SOC arc *(your fastest complete win — ~80% done)*
- Wazuh (Phase C) → Suricata→Wazuh (Phase D) → 5 attack/detect writeups (Phase E).
- **Gate:** explain default-deny, one Suricata signature end-to-end, and one detection *gap* you found.
- **🎯 Why first:** banking a near-done, complete security story early de-risks the 3-month deadline and is your strongest single narrative for the primary target. (Swap with M2 only if you'd rather lead with dev.)

### M2 — Dev platform core *(your #1 — what you'll live in)*
- Forgejo + Actions CI + container registry; code-server; Supabase + Postgres; Docker/Portainer deploy.
- Cloudflare Tunnel + Access → first public URL (a real hackathon-ready backend).
- **Gate:** explain your CI pipeline stages and how a request reaches a tunneled service from the public internet.

### M3 — Secure supply chain + secrets + IaC consolidation *(the crown jewel)*
- CI gates: gitleaks → Trivy → Semgrep → checkov → Syft (SBOM) → cosign (sign).
- SOPS/age for secrets-in-Git; Vault/OpenBao for runtime; codify the whole core in Terraform/Ansible.
- **Gate:** explain SAST vs SCA vs secret scanning, and why secrets live encrypted in Git but values never do.

### M4 — Polish → **RÉSUMÉ-READY GATE** *(end of 3 months)*
- Updated network diagram, README refresh, one blog-style writeup, < 5-min demo, ADRs for the big calls.
- Draft the 3 résumé bullets (§13) + rehearse the interview talking points (§14).
- **Gate:** deliver the 5-minute live demo without notes.

### Phase 2 (when budget/time allow) — see §9.2.

---

## 11. Bill of materials & budget

| Item | Milestone | Priority | ~Cost |
|---|---|---|---|
| 4× 8 GB DDR3-1600 **UDIMM** → 32 GB | M0 | **NOW** | $25–40 |
| Low-profile dual-port Intel NIC (i350-T2) | M0 | **NOW** | $15–25 |
| Domain (Cloudflare Registrar, `yourname.dev`) | M0 | **NOW** | $12/yr |
| Backblaze B2 (configs/repos only, encrypted) | M3 | SOON | ~$1–3/mo |
| **v1 core total** | | | **~$55–80 + ~$2/mo** |
| WISP travel router (GL.iNet) | Phase 2 | LATER | $25–100 |
| Tower "Vault" + ZFS drives + SSD + UPS | Phase 2 | LATER | $400–700 |
| *(EVGA 650 PSU from the thrift run — reuse if the Vault is an ATX build)* | Phase 2 | — | (already cheap) |

> **🎯 HM note:** Delivering an S-tier signal for **~$70** is itself a story — resourcefulness and
> prioritization under constraint. Lead with that, not apology for "only" an SFF.

---

## 12. Risks & mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Over-scoping one 32 GB SFF | High | Med | On-demand VMs (Kali/Supabase), capacity table (§9.3), Phase-2 tower |
| Bursty time vs 3-mo deadline | High | High | Front-load near-done M1; atomic, resumable stages; IaC to resume cleanly |
| AI-built but not understood | Med | **High** | Learning gate per milestone; one-line "why" per artifact; ADRs |
| Data loss (Nextcloud-as-primary too early) | Med | **High** | **Hard gate:** no primary data until ZFS mirror + tested 3-2-1 + verified restore (Phase 2) |
| Shared Wi-Fi: captive portal / MAC lock / flakiness | Med | Med | Travel router handles portal; ephemeral cloud for must-not-fail demos |
| Single node = no HA | Certain | Low | Accept; honest SLO + low MTTR story instead |
| Secret leak via generated config | Med | High | gitleaks + SOPS/age + Vault; values never in Git |
| Cost creep | Low | Med | Tiered backups; buy only the NOW tier; Phase-2 gated |

---

## 13. 🎯 Résumé bullets (draft — quantify as you go)

- **Security/SOC —** *Architected and operated a segmented SOC lab (pfSense + 5 VLAN zones, default-deny) with Wazuh SIEM and Suricata IDS; executed and documented 5 simulated attacks end-to-end (Nmap, Hydra, Metasploit, SQLi, C2 callback), including detection gaps and tuning.*
- **DevOps/Platform —** *Built a self-hosted DevOps platform (Forgejo + CI/CD, container registry, remote dev) with a secure software supply chain — secret scanning, SAST, dependency/container/IaC scanning, SBOM, and signed artifacts — with the entire environment reproducible from Git via Terraform + Ansible.*
- **Networking —** *Designed portable, zero-inbound infrastructure that operates behind untrusted shared Wi-Fi: outbound-only access via Tailscale (private) and Cloudflare Tunnel/Access (public, zero-trust), with no exposed ports and a measured uptime SLO.*

---

## 14. 🎯 Interview defense — questions you must be able to answer

Treat these as the exit exam. If a milestone's gate questions feel shaky, you're not done.
- Why default-deny, and how did you *prove* it works?
- Walk me through one Suricata alert from packet to Wazuh dashboard to phone.
- Why outbound tunnels instead of port-forwarding — what threat does that remove?
- SAST vs SCA vs secret scanning — what does each catch that the others miss?
- Why are encrypted secrets in Git acceptable but plaintext isn't? What's your key story?
- How do you rebuild this from scratch, and what's your MTTR?
- What's your uptime SLO, how do you measure it, and why isn't it 99.99%?
- Where did Claude Code help, and show me a change you rejected or rewrote.
- You're on 32 GB — how did you decide what runs always-on vs on-demand?

---

## 15. Open decisions to revisit
- M1 vs M2 order — default is SOC-first (deadline math); flip if you'd rather lead with dev.
- k3s timing — Phase 2 by default; pull earlier only if a specific deploy need appears.
- Authentik SSO — Phase 2; pull earlier if friends/family access arrives before then.
- Exact attack set for M1 — keep aligned with your TryHackMe rooms for efficiency.

---

## Appendix A — Tech stack at a glance

| Layer | Tool | Phase |
|---|---|---|
| Hypervisor | Proxmox VE 9.1 | live |
| Firewall/router/IDS | pfSense + Suricata (ETOpen) | live / M1 |
| SIEM | Wazuh (single-node) | M1 |
| Git + CI + registry | Forgejo/Gitea + Actions | M2 |
| Remote dev | code-server | M2 |
| App backend | Supabase + Postgres | M2 |
| Deploy | Docker/Portainer → k3s | M2 / P2 |
| Public ingress | Cloudflare Tunnel + Access | M2 |
| Private access | Tailscale (subnet router) | M0 |
| Secrets | gitleaks + SOPS/age + Vault/OpenBao | M0–M3 |
| Supply-chain security | Trivy, Semgrep, checkov, Syft, cosign | M3 |
| Alerting | ntfy (self-hosted) | M0 |
| Backups | PBS + restic → Backblaze B2 | M3 |
| IaC | Terraform + Ansible | M0–M3 |
| Observability | Uptime-Kuma → +Grafana/Prometheus | live / P2 |
| Identity/SSO | Authentik | P2 |
| Storage | 512 GB SSD → +ZFS mirror tower | live / P2 |

## Appendix B — Glossary (first-use terms)
**WISP/client mode** — a router joining an existing Wi-Fi as a client and re-sharing it as your own private network · **Outbound tunnel** — a connection your server dials *out*, so nothing needs an open inbound port · **SAST** — static analysis of your source for bugs/vulns · **SCA** — scanning your *dependencies* for known CVEs · **SBOM** — a signed inventory of everything in a build · **MTTR** — mean time to recovery · **SLO** — the reliability target you commit to and measure · **IaC** — infrastructure defined as code, rebuildable from Git.
