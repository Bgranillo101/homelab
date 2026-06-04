# Homelab SOC

Re-architecting a live 6-service Proxmox homelab into a segmented security lab —
firewall, network IDS, and SIEM — validated with simulated attacks.

> Status: **Phase B (segmentation) — in progress (~65%).** VLANs + DHCP done; firewall rules next. See [roadmap](docs/architecture/overview.md).

## Why this exists
Most homelabs run every service on one flat network. This project hardens a real,
running homelab in place: network segmentation, a default-deny firewall, intrusion
detection, centralized logging, and an attack/detect validation loop — documented
end to end.

## Architecture (target)
![Network diagram](diagrams/network-diagram.png)

| Layer | Tool |
|---|---|
| Hypervisor | Proxmox VE 9.1 |
| Services | LXC: Pi-hole, Nextcloud, Jellyfin, Uptime-Kuma, Portainer, Homepage |
| Firewall / Router | pfSense (VM) |
| Network IDS | Suricata (on pfSense, ETOpen ruleset) |
| SIEM | Wazuh (VM) |
| Attacker / Targets | Kali Linux + intentionally-vulnerable VMs |

Network is segmented into four VLAN zones (SERVICES, TRUSTED-MGMT, VULNERABLE, MONITORING)
with a default-deny posture between them. See [network design](docs/architecture/network-design.md).

## Roadmap
| Phase | Outcome | Status |
|---|---|---|
| A — Pre-flight | Backups, docs, storage headroom | **Complete** |
| B — Segmentation | pfSense + VLANs, migrate services | **In Progress** |
| C — SIEM | Wazuh + agents | Planned |
| D — IDS | Suricata → Wazuh | Planned |
| E — Attack & Detect | 5 attacks, detection writeups | Planned |
| F — Polish | Diagram, demo, writeup | Planned |

## Repo map
- `docs/architecture/` — roadmap, network design, IP plan, ADRs
- `docs/runbooks/` — step-by-step operational procedures
- `docs/journal/` — dated build log
- `configs/` — sanitized configuration exports
- `detections/` — attack simulations and how each was detected
- `diagrams/` — architecture diagrams (source + exports)

## Disclaimer
All offensive testing is performed exclusively against systems owned by the author,
inside an isolated lab network. Configuration files in this repo are sanitized.
