# Runbook: Stand up Suricata as a network IDS on pfSense

**When to use:** add network intrusion detection on the pfSense LAN chokepoint, feeding
(eventually) the Wazuh SIEM.
**Prereqs:** a working pfSense VM. **Snapshot it first** (this lab used `pre-suricata`).

## Steps

1. **Install the package.** System → Package Manager → Available Packages → install
   **Suricata** (this lab: `7.0.8_5`).
   - A cosmetic banner — *"package is configured but not (fully) installed or
     deprecated"* — may appear. It's a version-string mismatch, not a real failure;
     the Services → Suricata UI still loads. Only reinstall if the UI is genuinely broken.
2. **Global Settings** (Services → Suricata → Global Settings):
   - Enable **ETOpen Emerging Threats**, **Feodo Tracker Botnet C2 IP**, and
     **ABUSE.ch SSL Blacklist**. (Snort/ETPro need paid/registered keys — leave off.)
   - Update interval **1 Day** (start 00:02).
   - **Live Rule Swap OFF** — avoids a RAM spike on a 16 GB host.
   - **GeoLite2 OFF** — needs a MaxMind key; not required here.
3. **Updates tab** → **Update** → confirm all three sources show fresh MD5s / result
   "success".
4. **Create the interface** (Interfaces → Add): pick **LAN (vtnet1)**, **IDS mode**:
   - **Block Offenders OFF** (IDS-only; also avoids the netmap/inline-IPS crash that
     hits virtualized NICs).
   - Run Mode **AutoFP**.
   - **Detect-Engine Profile Medium** (RAM-conscious).
   - **EVE JSON Log ON**.
5. **Select rule categories** (the interface → Categories) — these map to planned
   Phase E attacks: `emerging-scan`, `emerging-exploit`, `emerging-exploit_kit`,
   `emerging-shellcode`, `emerging-attack_response`, `emerging-sql`,
   `emerging-web_server`, `emerging-web_specific_apps`, `emerging-malware`,
   `emerging-user_agents`, plus the **Feodo Tracker** and **ABUSE.ch SSL Blacklist** rows.
   Do **not** enable `emerging-dns` — too noisy on a Pi-hole network.
6. **Start** the interface (Interfaces → ▶) and confirm it comes up **green**.
7. **Smoke test** — from the Proxmox host:

       pct exec 100 -- curl -s http://testmynids.org/uid/index.html

   Then confirm **"GPL ATTACK_RESPONSE id check returned root"** (SID **2100498**)
   under Services → Suricata → **Alerts**. Action should be an alert, not a block
   (proves IDS mode).

## Gotchas
- The "configured but not fully installed" install banner is cosmetic — don't reinstall
  chasing it.
- Keep **Block Offenders OFF** to avoid the netmap/inline-IPS crash on virtualized NICs.
- **Detect profile Medium** and **Live Rule Swap OFF** respect a 16 GB RAM budget.
- Do **not** enable `emerging-dns` — Pi-hole generates constant DNS noise.

## SIEM forwarding (status: IN PROGRESS — does not work yet)
The goal is Suricata alerts in the Wazuh dashboard. A syslog path was chosen to avoid
putting an unsupported Wazuh agent on the FreeBSD firewall. What was tried:
- pfSense: Suricata interface **EVE Output → Syslog** (facility LOCAL1).
- pfSense: Status → System Logs → Settings → Remote Logging → `10.10.40.10:514`,
  contents "Everything".
- Wazuh (VM 201): added a `<remote><connection>syslog</connection><port>514</port>
  <protocol>udp</protocol><allowed-ips>10.10.40.1</allowed-ips></remote>` block to
  `/var/ossec/etc/ossec.conf`; manager restarted; `ss -lunp` confirms `wazuh-remoted`
  on UDP `0.0.0.0:514`. pfSense → `ping 10.10.40.10` = 0.0% loss.

**It still doesn't work.** `tcpdump -n -i any udp port 514` on Wazuh shows **0 packets**
while the curl is fired. Root cause: pfSense's Suricata EVE-syslog uses the FreeBSD
**LOCAL1** facility, but pfSense remote-logging only forwards its own managed log streams
and does **not** forward LOCAL1 — so EVE is written locally on pfSense and never shipped.
Valid config + successful ping, yet nothing arrives.

**Planned fix (next session):** switch EVE output back to **FILE** and ingest pfSense's
`eve.json` into Wazuh via a file-based method; re-run the smoke test to confirm the alert
reaches the dashboard end-to-end.

## Result (2026-06-24)
Suricata live on the LAN chokepoint, first detection confirmed (SID 2100498); SIEM
forwarding pending.
