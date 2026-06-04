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
| B | Segmentation | pfSense VM + 4 VLANs; migrate 6 LXCs into SERVICES zone; default-deny firewall rules | **In Progress** — VLANs + DHCP done; firewall rules + LXC migration remaining |
| C | SIEM | Wazuh VM in MONITORING zone; agents on key services | Not started |
| D | IDS | Suricata on pfSense (ETOpen rules); EVE JSON → Wazuh | Not started |
| E | Attack & Detect | Kali + vulnerable target in VULNERABLE zone; 5 attacks; detection writeups | Not started |
| F | Polish | Network diagram, README, demo video, resume bullets | Not started |
| F+ | Stretch | Ansible IaC, honeypot, custom Suricata rules, pfSense as real router, media HDD | Backlog |

## Current State Summary

- **Hardware:** Lenovo ThinkCentre M93p SFF, 16 GB RAM, 512 GB SATA SSD.
- **Hypervisor:** Proxmox VE 9.1.1, single node `bgranillo`, reachable at `https://192.168.0.13:8006`.
- **Services:** 6 LXC containers on a flat `192.168.0.0/24` network.
- **Storage:** LVM thin pool `pve/data` at ~7% used (~324 GB effective free) after disk reclaim.
- **Open risk:** backup status unverified — must confirm before Phase B.

See [network design](network-design.md) for the target VLAN architecture and [IP address plan](ip-address-plan.md)
for the complete address table.
