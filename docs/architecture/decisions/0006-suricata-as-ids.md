# ADR-0006: Suricata as network IDS

- **Status:** Proposed
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
