# ADR-0004: Network segmentation with pfSense + VLANs

- **Status:** Proposed
- **Date:** 2026-06-02

## Context
The current flat `192.168.0.0/24` network offers no lateral-movement barrier between services
and no inspection point for inter-zone traffic. An IDS needs a chokepoint to sit on.

## Decision
Deploy pfSense as a VM with a VLAN-aware bridge (`vmbr1`) on the lab side. Four VLAN zones
(SERVICES/10, TRUSTED-MGMT/20, VULNERABLE/30, MONITORING/40) with a default-deny firewall posture.

## Alternatives considered
- **OPNsense** — functionally equivalent; pfSense chosen for its documentation density and
  community resources, especially for Suricata integration.
- **VyOS** — steeper learning curve; better fit for pure routing/BGP, not a SOC lab where
  firewall GUI aids documentation and demos.
- **Proxmox SDN only** — provides VLAN isolation but no native IDS hook; would still need
  a separate firewall VM.

## Consequences
Real network segmentation with an auditable default-deny policy. pfSense becomes a VM with
WAN on `vmbr0` and LAN on the VLAN-aware `vmbr1`. All inter-zone traffic passes through
pfSense (and Suricata) — a clean IDS chokepoint. Tradeoff: one additional VM (~512 MB RAM,
~16 GB disk) and pfSense-specific configuration knowledge.
