# ADR-0007: Pull-based EVE ingestion (no agent on the firewall)

- **Status:** Accepted
- **Date:** 2026-06-25

## Context
Suricata on pfSense writes alerts to `eve.json`, but those alerts need to reach the Wazuh
SIEM for centralized detection (Phase D). The syslog path tried first is unworkable:
pfSense emits Suricata EVE under the FreeBSD `LOCAL1` facility, and pfSense remote-logging
only relays its own managed streams — it never forwards `LOCAL1`, so EVE never leaves the
firewall (confirmed with tcpdump: zero packets despite valid config and successful ping).
A different transport was needed without compromising the firewall.

## Decision
Pull `eve.json` from pfSense to the Wazuh box over a **forced-command, restricted SSH key**
and ingest it locally with a Wazuh `localfile` block. An incremental shipper
(`pull-eve.sh`, per-minute cron) appends only new lines. **Nothing is installed on the
firewall.**

## Alternatives considered
- **Wazuh agent on pfSense** — the standard pattern, rejected. Installing an unsupported
  package on a FreeBSD production router requires enabling the upstream FreeBSD pkg repo
  (the most common way to corrupt a pfSense box), and a pfSense update can wipe or break
  the agent. The blast radius of a firewall failure (whole-house outage) outweighs the
  convenience.
- **rsync pull** — rejected: not shipped with pfSense, would again require enabling the
  pkg repo. SSH + `cat` is already present and supported.
- **Syslog (LOCAL1)** — failed: pfSense does not forward the `LOCAL1` facility (see
  Context and ADR-0006).

## Consequences
Upgrade-safe and minimal blast radius — the firewall is untouched and log shipping is
decoupled from the appliance. The restricted key can only `cat` the one EVE file (no
shell, no port-forwarding, no agent forwarding), and the private key never leaves the
Wazuh box. The pull uses the allowed MONITORING→pfSense outbound direction; SERVICES→
MONITORING stays blocked by default-deny. Tradeoff: a small custom shipper plus a
per-minute cron, and ~60 s ingestion latency. See the
[Suricata→Wazuh integration runbook](../../runbooks/suricata-wazuh-integration.md).
