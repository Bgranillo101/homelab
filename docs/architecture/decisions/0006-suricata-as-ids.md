# ADR-0006: Suricata as network IDS

- **Status:** Accepted
- **Date:** 2026-06-02

## Context
Need network intrusion detection on inter-zone traffic at the pfSense chokepoint, with
output that feeds the Wazuh SIEM.

## Decision
Suricata running on pfSense (via the pfSense package), using the ETOpen (Emerging Threats
Open) ruleset, with EVE JSON output forwarded to Wazuh.

## Alternatives considered
- **Snort** — older tooling on pfSense; smaller community and slower rule updates than
  Suricata's ETOpen feed; multi-threaded performance also weaker.
- **Zeek** — analysis and protocol logging rather than signature-based alerting; better
  as a supplement than a primary IDS for this use case.

## Consequences
Signature-based detection at the network chokepoint, feeding the SIEM via EVE JSON.
ETOpen is a free, well-maintained ruleset covering common attack patterns. Tradeoff:
rule tuning is required to reduce false positives, especially for home/lab traffic
patterns (e.g., Pi-hole DNS queries may trigger some rules initially).

### Implemented (Phase D, 2026-06-24)
Suricata `7.0.8_5` deployed on the pfSense LAN (vtnet1) interface in IDS mode with the
ETOpen ruleset (plus Feodo Tracker and ABUSE.ch SSL Blacklist). First detection
validated: a `testmynids.org` smoke test tripped GPL ATTACK_RESPONSE id check returned
root (SID 2100498) in the Suricata Alerts tab.

### Open consequence — EVE → Wazuh forwarding
Forwarding EVE output to Wazuh via syslog did **not** work: pfSense's Suricata EVE-syslog
output uses the FreeBSD LOCAL1 facility, but pfSense remote-logging only forwards its own
managed log streams and does not forward LOCAL1 — so EVE messages never leave the
firewall. The planned path is to switch EVE output back to FILE and ingest pfSense's
`eve.json` into Wazuh via a file-based method.
