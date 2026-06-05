# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

## [Phase B — Segmentation] — 2026-06-05 — COMPLETE

### Added
- Default-deny firewall rules across all four pfSense interfaces:
  - VULNERABLE: block → 10.10.0.0/16 (all lab zones), allow → internet
  - LAN (SERVICES): block → TRUSTEDMGMT, block → MONITORING, allow → internet
  - TRUSTEDMGMT: allow → anywhere (admin zone)
  - MONITORING: allow inbound to Wazuh, allow → internet

### Changed
- Migrated all 6 LXC containers from flat home network (192.168.0.x / vmbr0) to
  SERVICES zone (10.10.10.x / vmbr1):
  - 100 pihole → 10.10.10.100
  - 101 nextcloud → 10.10.10.101
  - 102 jellyfin → 10.10.10.102
  - 103 uptime-kuma → 10.10.10.103
  - 104 portainer → 10.10.10.104
  - 105 homepage → 10.10.10.105
- Internet access verified on all containers (ping 1.1.1.1, 0% packet loss).
- DNS via Pi-hole verified (nslookup google.com 10.10.10.100).

### Fixed
- Removed erroneous VLAN tag=10 from LXC net0 configs — pfSense LAN interface
  is untagged; containers must connect untagged to vmbr1.

## [Phase B — Segmentation] — 2026-06-03 — PARTIAL

### Added
- Proxmox snapshot `post-wizard-clean` on pfSense VM (ID 200).
- VLAN sub-interfaces 20/30/40 on vtnet1 (pfSense LAN).
- pfSense interfaces: TRUSTEDMGMT (10.10.20.1/24), VULNERABLE (10.10.30.1/24),
  MONITORING (10.10.40.1/24).
- DHCP server enabled on all three new interfaces, range .100–.199.
- docs/LINKS.md — service URL quick reference.

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
