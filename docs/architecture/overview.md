# Architecture Overview & Project Roadmap

A live 6-service Proxmox homelab is being re-architected into a segmented security lab.
The narrative: this is not a greenfield tutorial build — a real, running system is being
hardened in place, making the story "re-architected a live system" rather than "followed a guide."

## Goal

Demonstrate end-to-end security operations skills on owned hardware:
- Network segmentation with a default-deny firewall
- Network intrusion detection (Suricata, ETOpen rules)
- Centralized SIEM (Wazuh) with host agents
- Simulated attacks with documented detection outcomes

## Phase Plan

| Phase | Name | Outcome | Status |
|---|---|---|---|
| A | Pre-flight | Backups verified, docs scaffolded, storage reclaimed | **Complete** |
| B | Segmentation | pfSense VM + 4 VLANs; migrate 6 LXCs into SERVICES zone; default-deny firewall rules | **Complete** |
| C | SIEM | Wazuh VM in MONITORING zone; agents on key services | **Complete** |
| D | IDS | Suricata on pfSense (ETOpen rules); EVE JSON → Wazuh | **In progress** (Suricata detecting; EVE→Wazuh forwarding pending) |
| E | Attack & Detect | Kali + vulnerable target in VULNERABLE zone; 5 attacks; detection writeups | Not started |
| F | Polish | Network diagram, README, demo video, resume bullets | Not started |
| F+ | Stretch | Ansible IaC, honeypot, custom Suricata rules, pfSense as real router, media HDD | Backlog |

## Current State Summary

- **Hardware:** Lenovo ThinkCentre M93p SFF, 16 GB RAM, 512 GB SATA SSD.
- **Hypervisor:** Proxmox VE 9.1.1, single node `bgranillo`, reachable at `https://192.168.0.13:8006`.
- **Network:** segmented into 4 VLAN zones behind a pfSense VM (`fw.lab.local`) with a
  default-deny posture between zones. All 6 LXC services run in the SERVICES zone on
  VLAN 10 (`10.10.10.100–105`).
- **SIEM:** Wazuh all-in-one live on VM 201 in the MONITORING zone (VLAN 40, `10.10.40.10`),
  with agents reporting from Pi-hole (CT 100) and Nextcloud (CT 101).
- **IDS:** Suricata running on the pfSense LAN chokepoint (vtnet1) in IDS mode with the
  ETOpen ruleset; first detection validated (GPL ATTACK_RESPONSE, SID 2100498). Forwarding
  EVE output into Wazuh is still in progress — see Phase D status above.
- **Storage:** LVM thin pool `pve/data` at ~7% used (~324 GB effective free) after disk reclaim.

See [network design](network-design.md) for the target VLAN architecture and [IP address plan](ip-address-plan.md)
for the complete address table.
