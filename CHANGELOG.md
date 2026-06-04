# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

## [Phase B — Segmentation] — 2026-06-03 — IN PROGRESS
### Added
- Proxmox snapshot `post-wizard-clean` on pfSense VM (ID 200) as rollback point.
- Three VLAN sub-interfaces on `vtnet1` (LAN side): tags 20, 30, 40.
- Interface assignments: OPT1 → TRUSTEDMGMT, OPT2 → VULNERABLE, OPT3 → MONITORING.
- Static IPv4 addresses on all three new interfaces:
  - TRUSTEDMGMT: `10.10.20.1/24`
  - VULNERABLE: `10.10.30.1/24`
  - MONITORING: `10.10.40.1/24`
- DHCP server enabled on all three new interfaces, range `.100–.199` each.

### Remaining (Phase B)
- Default-deny firewall rules between zones.
- Migrate 6 LXCs into VLAN 10 (SERVICES) with static DHCP mappings.
- Fix CT 103 (uptime-kuma) — move from untagged vmbr1 into VLAN 10 properly.

## [Phase A — Pre-flight] — 2026-06-02 — COMPLETE
### Added
- Repository scaffolding, documentation structure, and ADRs.
- Took vzdump backups of all six LXC containers (CTIDs 100–105) to local
  storage before beginning Phase B. Backups verified present in Datacenter UI.

### Changed
- Reclaimed ~54 GB of allocated storage by shrinking all six LXC root disks
  (Pi-hole/Uptime-Kuma/Portainer/Homepage 8→4 GB, Jellyfin 20→12 GB,
  Nextcloud 50→20 GB). See ADR-0003 and the disk-resize runbook.

## [2026-06-02] — Project documented
### Added
- Initial audit of the running homelab (Proxmox 9.1.1, six LXC services).
