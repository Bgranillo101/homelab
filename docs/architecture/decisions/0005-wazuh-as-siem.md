# ADR-0005: Wazuh as SIEM

- **Status:** Accepted
- **Date:** 2026-06-02

## Context
Need centralized security visibility: log aggregation, alerting, and dashboards across
the segmented network — with a Suricata decoder and host agent support.

## Decision
Wazuh, deployed as a VM in VLAN 40 (MONITORING zone).

## Alternatives considered
- **ELK stack (Elasticsearch + Logstash + Kibana)** — more assembly required; heavier
  resource footprint; no bundled Suricata decoder or host agent framework.
- **Graylog** — solid log management but weaker on host-agent alerting and rule-based
  detection out of the box.

## Consequences
Bundled Suricata EVE JSON decoder, host agents for the LXC services, and pre-built
security dashboards. Single-VM deployment (~4 GB RAM). Wazuh's opinionated agent/manager
model means some tuning is needed to suppress noise, but the out-of-the-box detection
coverage is better for a first SIEM than building from scratch.

Deployed 2026-06-23 (Phase C): VM 201 in VLAN 40 (10.10.40.10); agents active on Pi-hole and Nextcloud.
