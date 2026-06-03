# ADR-0003: Reclaim storage via in-place LXC thin-pool resize

- **Status:** Accepted
- **Date:** 2026-06-02

## Context
The security phase requires ~126 GB for new VMs (pfSense ~50 GB, Wazuh ~50 GB, Kali ~32 GB).
The LVM thin pool metadata showed low free space on paper despite the pool being only ~7% used —
all six LXC root disks were over-provisioned relative to actual usage.

## Decision
Shrink existing LXC disks in-place (filesystem → LVM volume → Proxmox config) rather than
purchasing additional storage now.

## Alternatives considered
- **Add a 2 TB HDD** — deferred to the media phase (Jellyfin); solves storage permanently
  but adds cost and complexity now.
- **Do nothing** — on-paper headroom looked insufficient, risking VM provisioning failures
  even though actual used space was low.

## Consequences
Freed ~54 GB of *allocated* thin-pool space; thin pool now ~7% used with ~324 GB effective
free. The in-place shrink carries risk (data loss if done out of order); mitigated by taking
`vzdump` backups before each resize and following the strict filesystem→volume→config sequence
documented in the [runbook](../../runbooks/lxc-disk-resize.md).

Hit and resolved an MMP (Multi-Mount Protection) lock on CT 101 (Nextcloud) — cleared with
`tune2fs -E clear_mmp` and re-ran `e2fsck`. All six containers healthy post-resize.
