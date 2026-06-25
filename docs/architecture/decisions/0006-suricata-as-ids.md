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

### EVE → Wazuh forwarding (resolved 2026-06-25)
Forwarding EVE output to Wazuh via syslog did **not** work: pfSense's Suricata EVE-syslog
output uses the FreeBSD LOCAL1 facility, but pfSense remote-logging only forwards its own
managed log streams and does not forward LOCAL1 — so EVE messages never leave the
firewall. This was replaced by a file-pull pipeline: Wazuh pulls `eve.json` from pfSense
over a forced-command, restricted SSH key and ingests it with a `localfile` block, with
nothing installed on the firewall. See [ADR-0007](0007-pull-based-eve-ingestion.md).

### SIEM-visible alerts to date
The pipeline is validated end to end (1,783 alerts in the Wazuh dashboard, Wazuh rule
86601). These are all STREAM-engine TCP-anomaly events — a baseline, partly self-generated
by the pull traffic. Attack-signature validation (the ETOpen rule hits this ruleset is
chosen for) is Phase E.
