# Implementation Roadmap — Project Sentinel

> Companion to `homelab-platform-PRD.md`. This is the **step-by-step build guide**: every
> milestone broken into small, verifiable steps written for "learning as I go."
> Authored by Sr. PM + Principal Engineer. Owner: **Brandon Granillo** (junior, Computer
> Systems Engineering, ASU). Working style: hands-on, one step at a time, snapshots before
> risky changes, screenshots to confirm.
>
> **Last reviewed:** 2026-06-25

---

## How to use this doc

Each step follows the same shape so nothing is ambiguous:

- **Goal / Why** — what this accomplishes and why it matters (no step is a black box).
- **Do** — exact commands or clicks.
- **Verify** — how you *know* it worked.
- **Rollback** — how to undo it if it breaks.

Every milestone ends with a **🎓 Learning Gate** — 2–3 things you must be able to explain in
your own words before moving on. If a gate feels shaky, you're not done. (This is what makes
the project *defensible* in an interview instead of "Claude built it.")

**Five rules that apply to every step:**
1. **Snapshot before any risky change** (Proxmox → VM → Snapshots). Name it meaningfully.
2. **Never type secrets into a chat, screenshot, or committed file.** See the Security Spine in `CLAUDE.md`.
3. **noVNC console mangles typed URLs** — use the console's clipboard-paste panel or SSH in instead.
4. **Document as you go** — append to `docs/journal/<date>.md`, update `CHANGELOG.md` and `docs/LINKS.md`.
5. **`gitleaks protect --staged` must pass before every commit.**

---

## You are here (live state, 2026-06-25)

| Thing | Value |
|---|---|
| Host | M93p SFF, Proxmox 9.1.1, node `bgranillo`, `192.168.0.13:8006` |
| Home uplink | Cox CGM4331COX, `192.168.0.1` — blocks most inter-device ports except SSH/22 → tunnels + Tailscale |
| pfSense | VM **200**, LAN `10.10.10.1`, 4 VLANs, default-deny live |
| Services | 6 LXCs on `10.10.10.100–105` (pihole/nextcloud/jellyfin/uptime-kuma/portainer/homepage) |
| **Wazuh** | VM **201** @ `10.10.40.10` — **live (Phase C complete)**: manager/indexer/dashboard active; agents 001 pihole + 002 nextcloud active |
| **Suricata** | on pfSense LAN (vtnet1), IDS mode — **live (Phase D complete)**: EVE JSON shipped to Wazuh via scoped-key file-pull (1,783 alerts indexed, rule 86601). STREAM baseline; attack sigs are Phase E |
| Vuln target | planned @ `10.10.30.51` (VLAN 30) |
| pfSense access | `ssh -L 9443:10.10.10.1:443 -N root@192.168.0.13` → `https://localhost:9443` |
| Tailscale | account active; **tailscaled doesn't persist across CT reboots** (fix in M0) |

> **Critical reality check:** because Cox/CGM4331COX blocks inter-device ports today and the
> apartment will be shared Wi-Fi, **no inbound port-forwarding is ever available.** Every remote
> path in this roadmap is outbound-only (Tailscale, Cloudflare Tunnel). This is by design.

---

# M0 — Foundation & Security Remediation

*First heavy burst. Unblocks everything. ~$70 hardware + ~$12 domain.*

### Step 0.0 — Establish a credential baseline (do this FIRST)
- **Why:** Before building anything new, make sure every service uses a unique, strong secret and nothing sensitive lives in the repo. Credential hygiene is the foundation everything else sits on.
- **Do:**
  1. Inventory every service/account that holds a credential (Nextcloud, its database, Jellyfin, Uptime-Kuma, Portainer, pfSense admin, Proxmox root, Tailscale, any DB roles).
  2. Ensure each has a **unique, strong** secret stored in a password manager (Bitwarden/1Password); rotate any that are weak, shared, or reused. **Never reuse one password across two services.**
  3. Confirm no secret is committed anywhere: `gitleaks detect --source . --redact` across the repo **and history**.
  4. Going forward, secrets live only in your password manager and (later) SOPS/Vault — never in a doc, chat, screenshot, or commit.
- **Verify:** `gitleaks detect` reports no findings; you can log into each service with its stored secret; any rotated credential is updated everywhere it's used.
- **Rollback:** none needed — you're tightening, not changing topology. Keep prior creds in the password manager's history only until each rotation is confirmed.

### Step 0.1 — RAM to 32 GB
- **Why:** 16 GB can't hold Wazuh + the dev platform concurrently. This is the single biggest unblock.
- **Do:** `vzdump` all CTs (UI → each CT → Backup → Backup Now) → power down → install 4× 8 GB DDR3-1600 **UDIMM** (desktop, not SO-DIMM) in pairs → boot.
- **Verify:** Proxmox → node `bgranillo` → Summary shows ~32 GB; `free -h` in host shell confirms.
- **Rollback:** reseat/restore original sticks; CTs are backed up.

### Step 0.2 — Dual-port NIC (optional now, recommended)
- **Why:** Gives pfSense a real physical WAN+LAN later and frees you from VLAN-on-a-stick. Fits the low-profile PCIe x16 slot.
- **Do:** Power down → install Intel i350-T2 (or PRO/1000) → boot → it appears as new `enpXsY` interfaces in Proxmox → Network.
- **Verify:** `ip link` lists the new interfaces.
- **Rollback:** remove card; nothing else changes.

### Step 0.3 — Buy the domain
- **Why:** public URLs, clean TLS, Cloudflare Tunnel all want a real domain.
- **Do:** Register `yourname.dev` (or `.com`) at **Cloudflare Registrar** (at-cost). Let Cloudflare manage DNS.
- **Verify:** domain shows "Active" in the Cloudflare dashboard.

### Step 0.4 — Scaffold the IaC repo
- **Why:** "reproducible from Git" is the headline infra signal; start the skeleton now so everything afterward gets codified.
- **Do:** In `homelab-soc`, add `terraform/` (provider `bpg/proxmox`, Cloudflare provider) and `ansible/` (inventory, roles/) skeletons. Use Claude Code to scaffold; **read every generated file and add a one-line `# why` comment.**
- **Verify:** `terraform validate` and `ansible-lint` run clean on the skeleton.
- **Rollback:** it's a new directory; delete if needed.

### Step 0.5 — Tailscale subnet router + persistence fix
- **Why:** private remote access to *all* internal subnets from any device; fixes the "tailscaled dies after CT reboot" bug.
- **Do:**
  1. Create a small LXC (Debian 12) on VLAN 10, e.g. `tailscale-router` @ `10.10.10.9`.
  2. Install Tailscale; bring it up advertising your subnets:
     `tailscale up --advertise-routes=10.10.10.0/24,10.10.20.0/24,10.10.40.0/24 --accept-dns=false`
  3. In the Tailscale admin console → the node → **approve the advertised routes.**
  4. Make it durable: run Tailscale as the **systemd `tailscaled` service** (enable it), *not* the manual `tailscaled &` command you used before. For LXC, ensure `/dev/net/tun` is available to the container.
- **Verify:** from your phone on cellular (Wi-Fi off), you can reach `10.10.10.100` (Pi-hole admin). Reboot the LXC — it reconnects automatically.
- **Rollback:** `tailscale down`; remove the route approval.

### Step 0.6 — Self-hosted ntfy (phone alerting)
- **Why:** makes the lab feel operational — alerts from Wazuh/Uptime-Kuma/PBS/CI land on your phone.
- **Do:** Deploy `binwiederhier/ntfy` (Docker) in an LXC on VLAN 10. Install the ntfy app on your phone; subscribe to a private topic over Tailscale (do **not** expose ntfy publicly).
- **Verify:** `curl -d "test" http://<ntfy-ip>/sentinel-alerts` pops a phone notification.

> **🎓 Learning Gate M0:** Explain (1) why 32 GB unblocks the workload, (2) what a Tailscale
> *subnet router* does and why it beats port-forwarding on a blocked/shared network, (3) why
> rotating to unique per-service secrets matters even on a "home" lab.

---

# M1 — Finish the SOC Arc (Phases C → D → E)

*Your fastest complete win. **Phases C (SIEM) and D (Suricata IDS) are banked** — Phase E (attack & detect) is next.*

### Step 1.1 — Finish the Wazuh install (VM 201) ✅ Done (2026-06-23)
- **Why:** Phase C centerpiece; central SIEM everything else feeds.
- **Status:** Complete — manager/indexer/dashboard active + enabled on VM 201; `admin`
  password rotated via `wazuh-passwords-tool.sh` (stored in the password manager).
- **Pre:** snapshot VM 201 as `pre-wazuh-install`. Confirm it has ≥4 GB RAM, ≥50 GB disk, internet + DNS (`ping -c2 1.1.1.1`, `ping -c2 google.com`).
- **Do (in the VM 201 console — use clipboard-paste, the console mangles typed URLs):**
  ```bash
  curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
  sudo bash ./wazuh-install.sh -a
  ```
  The assistant installs indexer + manager + dashboard and **prints generated admin credentials at the end** — capture them straight into your password manager (never a plaintext file).
- **Verify:**
  ```bash
  systemctl status wazuh-manager wazuh-indexer wazuh-dashboard   # all active
  ```
  Then reach the dashboard. The path you already proved works:
  - pfSense → Firewall → Rules → **LAN**: Pass `LAN subnets → 10.10.40.10 : 443` (management rule).
  - Proxmox host route: `ip route add 10.10.40.0/24 via 10.10.10.1 dev vmbr1` (persist it in `/etc/network/interfaces` so it survives reboot).
  - Tunnel: `ssh -L 9444:10.10.40.10:443 -N root@192.168.0.13` → `https://localhost:9444`.
- **Rollback:** restore the `pre-wazuh-install` snapshot; or `sudo bash ./wazuh-install.sh -u` to uninstall.

### Step 1.2 — Deploy agents ✅ Done (2026-06-23)
- **Why:** the SIEM is only as useful as what reports to it.
- **Do:** From the Wazuh dashboard → Agents → Deploy. Install the agent on Pi-hole (100) and one more LXC, pointing `WAZUH_MANAGER=10.10.40.10`.
- **Verify:** both agents show **Active** in the dashboard; events appear.
- **Status:** Complete — agents 001 (pihole, CT 100) and 002 (nextcloud, CT 101) active
  (agent 4.11.2). Required a pfSense LAN rule `10.10.10.0/24 → 10.10.40.10` TCP 1514-1515
  above the `Block SERVICES → MONITORING` rule. Full procedure + gotchas in
  [docs/runbooks/wazuh-agent-on-lxc.md](runbooks/wazuh-agent-on-lxc.md).

### Step 1.3 — Suricata IDS → Wazuh (Phase D) ✅ Done (2026-06-25)
- **Why:** network-layer detection at the pfSense chokepoint.
- **Do:** pfSense → System → Package Manager → install **Suricata** → enable on the LAN/inter-VLAN interface → enable **ETOpen** ruleset → enable **EVE JSON** output → forward EVE JSON to Wazuh.
- **Verify:** trigger a test (e.g., `curl http://testmynids.org/uid/index.html` from a lab host) and see the alert in Wazuh. Tune the obvious Pi-hole DNS false positives.
- **Rollback:** disable Suricata service in pfSense; snapshot first.
- **Status:** Complete — Suricata `7.0.8_5` on pfSense LAN (vtnet1) in IDS mode (ETOpen +
  Feodo + ABUSE.ch SSL); first detection SID 2100498. **Syslog forwarding was a dead end**
  (pfSense doesn't forward the Suricata `LOCAL1` facility), so EVE is shipped via a
  **scoped-key file-pull** instead — a forced-command SSH key + `pull-eve.sh` + per-minute
  cron + Wazuh `localfile`, with nothing installed on the firewall. End to end: 1,783
  Suricata alerts in the dashboard (rule 86601). These are STREAM-engine baseline events;
  real attack-signature hits come in Phase E. Full procedure + gotchas in
  [docs/runbooks/suricata-wazuh-integration.md](runbooks/suricata-wazuh-integration.md);
  decision in [ADR-0007](architecture/decisions/0007-pull-based-eve-ingestion.md).

### Step 1.4 — Attack & Detect ×5 (Phase E)
- **Why:** proves the SOC works end to end — the strongest single story for the security track.
- **Pre:** stand up Kali + a deliberately-vulnerable target (e.g., Metasploitable/DVWA) in **VLAN 30** (`10.10.30.x`, target `10.10.30.51`). Confirm VLAN 30 is isolated by default-deny.
- **Do:** run and document each — Nmap recon, Hydra brute force, a Metasploit exploit, a web SQLi, a C2/malware callback. (Your TryHackMe Nmap/Hydra/Metasploit rooms map directly here.)
- **Verify + record:** for each, fill `detections/<attack>.md` from the template: exact command → what Suricata saw (rule + SID + timestamp) → what Wazuh showed (screenshot) → defender response → **what I learned, including detection gaps.**
- **Rollback:** Kali/target are on-demand VMs; power them off when not running exercises (RAM budget).

> **🎓 Learning Gate M1:** Walk one Suricata alert from packet → EVE JSON → Wazuh dashboard →
> phone. Explain default-deny and how you *proved* VLAN 30 can't reach VLAN 10. Name one attack
> that slipped past and why.

---

# M2 — Dev Platform Core (your #1)

*What you'll actually live in. Docker first so hackathon backends go live fast.*

### Step 2.1 — Forgejo (Git + registry) + CI runner
- **Why:** self-hosted GitHub: private repos, issues, built-in **container registry**, and Actions CI.
- **Do:** Deploy **Forgejo** (Docker) + Postgres in an LXC on VLAN 10. Enable the **package/container registry**. Register a **Forgejo Actions runner** (GitHub-Actions-compatible).
- **Verify:** create a repo, `git push` to it, run a hello-world Actions workflow that goes green, push an image to the registry.
- **Rollback:** snapshot the LXC first; Compose `down` to remove.

### Step 2.2 — code-server (remote dev)
- **Why:** code in-browser from any device (ties to portability — work from a tablet/locked-down machine).
- **Do:** Deploy `code-server` (Docker) in an LXC on VLAN 10. Reach it privately over Tailscale.
- **Verify:** open it from your iPad/phone over Tailscale, edit a repo, commit.

### Step 2.3 — Supabase (reusable app backend)
- **Why:** Postgres + auth + storage + realtime — a self-hosted Firebase alternative your Expo apps and hackathons reuse.
- **Do:** Deploy self-hosted Supabase (Docker) in an LXC on VLAN 10. **Generate fresh JWT secret + DB password into your password manager** (do not use any example/default keys).
- **Verify:** hit the Supabase REST endpoint from a test Expo app over Tailscale; a row read/write works.
- **Rollback:** snapshot first; Compose `down`.

### Step 2.4 — Cloudflare Tunnel (public exposure, zero inbound)
- **Why:** the only way to get public URLs behind blocked/shared Wi-Fi; outbound-only, TLS, hides your IP.
- **Do:**
  1. New VLAN **50 (PUBLIC/DMZ)** in pfSense for internet-facing workloads, isolated from SERVICES by default-deny.
  2. Run `cloudflared` (Docker) in the DMZ; create a **named tunnel** in the Cloudflare Zero Trust dashboard; map `api.yourname.dev → http://<supabase>:port`, `demo.yourname.dev → <app>`.
  3. Put **Cloudflare Access** (zero-trust auth) in front of anything not meant to be fully public.
- **Verify:** from cellular, open `https://demo.yourname.dev` — it resolves with valid TLS and no port-forward exists anywhere.
- **Rollback:** delete the tunnel/hostname in the Zero Trust dashboard; stop `cloudflared`.

> **🎓 Learning Gate M2:** Trace a request from the public internet → Cloudflare edge → tunnel →
> your DMZ service, and explain why no inbound port is open. Explain why public apps live in
> VLAN 50, not alongside SERVICES.

---

# M3 — Secure Supply Chain + Secrets + IaC (the crown jewel)

*The intersection of your #1 and #2 — the highest-signal work in the build.*

### Step 3.1 — Secrets management
- **Why:** with heavy IaC + AI-generated configs + public exposure, secret hygiene is non-negotiable.
- **Do (layered):**
  1. **gitleaks** pre-commit — already in place; keep it.
  2. **SOPS + age** — encrypt secrets *in* Git so IaC stays reproducible without leaking values. Store the `age` private key in your password manager, never in the repo.
  3. **Vault** (or **OpenBao**, fully-open fork) in an LXC for runtime/dynamic secrets. Start small.
- **Verify:** a secret committed via SOPS is unreadable in the repo but decrypts locally; gitleaks finds nothing; an app pulls a secret from Vault at runtime.

### Step 3.2 — Secure CI/CD pipeline
- **Why:** a pipeline that scans itself = "secure software supply chain." Rare, standout signal.
- **Do:** add stages to your Forgejo Actions workflow, in order: **gitleaks** (secrets) → **Trivy** (dependencies + container image + IaC misconfig) → **Semgrep** (SAST) → **checkov** (IaC) → **Syft** (SBOM) → **cosign** (sign the image). Fail the build on high-severity findings.
- **Verify:** a PR with a planted vuln/secret is **blocked** by CI; a clean PR produces a signed image + SBOM artifact.
- **Rollback:** gates are additive; loosen severity thresholds if they block legitimate work, but document why.

### Step 3.3 — Codify everything (IaC consolidation)
- **Why:** reproducibility + low MTTR; "rebuild from Git" is the SRE story.
- **Do:** move VM/LXC definitions to Terraform (`bpg/proxmox`), service config to Ansible roles, DNS/tunnels to the Cloudflare provider. Write one ADR per major decision.
- **Verify:** tear down a *non-critical* test LXC and rebuild it entirely from `terraform apply` + an Ansible play. Time it (target MTTR < 1 hr).

> **🎓 Learning Gate M3:** Explain SAST vs SCA vs secret scanning (what each catches). Explain
> why encrypted secrets in Git are fine but plaintext isn't, and where your keys live. State
> your measured rebuild MTTR.

---

# M4 — Polish → Résumé-Ready Gate

*End of the 3-month core.*

- **Step 4.1 — Diagram:** update the network diagram (now: travel-router → pfSense → 5 VLANs, Tailscale + Cloudflare Tunnel, two outbound channels). Export PNG to `diagrams/`.
- **Step 4.2 — README + writeup:** refresh `README.md`; write one blog-style post (a detection chain or the zero-inbound architecture). Substantially your own words.
- **Step 4.3 — Demo:** record a <5-min screen capture (Wazuh quiet → Kali Nmap from VLAN 30 → alert appears → phone buzzes via ntfy → show the public `demo.yourname.dev`). Unlisted YouTube, link from README.
- **Step 4.4 — ADRs:** write/refresh ADRs for the big calls (5th VLAN, Tailscale+Cloudflare, Forgejo-over-GitLab, SOPS+Vault, single-node SLO).
- **Step 4.5 — Résumé:** finalize the three bullets from PRD §13; rehearse PRD §14 interview questions out loud.

> **🎓 Learning Gate M4 (the exit exam):** deliver the 5-minute demo **without notes**, then
> answer three random questions from PRD §14.

---

# Phase 2 — Deferred (post-résumé, budget/time-gated)

Pull these when money/time allow. None block the résumé-ready core.

1. **Travel router (WISP) + apartment move** — GL.iNet in WISP/repeater mode → pfSense WAN; handle captive portal; check subnet overlap (`10.10.x.x` vs the building's range). Tailscale + Cloudflare Tunnel carry over unchanged.
2. **"Vault" tower + ZFS** — used tower (reuse the EVGA 650 PSU if ATX) as a 2nd Proxmox node; ZFS mirror for irreplaceable data, single disk for media.
3. **Nextcloud as primary storage** — **HARD GATE:** only after ZFS mirror + tested 3-2-1 (PBS + `restic`→Backblaze B2) + a *verified restore*. Don't move your only copy before this.
4. **Jellyfin bulk media** — big HDD on the tower; enable i7-4790 QuickSync transcoding.
5. **k3s** — migrate Dockerized apps to k3s for the orchestration signal.
6. **Grafana + Prometheus** — node_exporter/cAdvisor across hosts; dashboards beside Wazuh.
7. **Authentik SSO** — one secure login (2FA) for friends/family + front the public services.

---

## Appendix — quick command reference (verify, don't memorize)

```bash
# Reach pfSense UI
ssh -L 9443:10.10.10.1:443 -N root@192.168.0.13      # → https://localhost:9443
# Reach Wazuh dashboard (after LAN rule + host route)
ssh -L 9444:10.10.40.10:443 -N root@192.168.0.13     # → https://localhost:9444
# Proxmox host route to VLAN 40 (persist in /etc/network/interfaces)
ip route add 10.10.40.0/24 via 10.10.10.1 dev vmbr1
# Wazuh single-node install (run IN VM 201, paste via console clipboard)
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
# Tailscale subnet router (persist via systemd service)
tailscale up --advertise-routes=10.10.10.0/24,10.10.20.0/24,10.10.40.0/24 --accept-dns=false
# Secret scan before every commit
gitleaks protect --staged
```
