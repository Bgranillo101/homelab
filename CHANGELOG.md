# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

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
