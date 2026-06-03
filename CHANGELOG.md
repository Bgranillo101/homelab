# Changelog

All notable changes to this project are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]
### Added
- Repository scaffolding, documentation structure, and ADRs.

### Changed
- Reclaimed ~54 GB of allocated storage by shrinking all six LXC root disks
  (Pi-hole/Uptime-Kuma/Portainer/Homepage 8→4 GB, Jellyfin 20→12 GB,
  Nextcloud 50→20 GB). See ADR-0003 and the disk-resize runbook.

### Open
- Verify pre-change container backups exist before starting Phase B.

## [2026-06-02] — Project documented
### Added
- Initial audit of the running homelab (Proxmox 9.1.1, six LXC services).
