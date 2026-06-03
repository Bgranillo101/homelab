# Runbook: Shrink an LXC root disk on a Proxmox thin pool

**When to use:** reclaim *allocated* thin-pool space from an over-provisioned LXC.
**Risk:** shrinking is destructive if done out of order. Always back up first.

## Order (never deviate)
1. Back up the container (`vzdump` / UI → Backup Now).
2. Stop the container: `pct stop <CTID>`
3. Filesystem check: `e2fsck -fy /dev/pve/vm-<CTID>-disk-0`
4. Shrink filesystem (target minus 1 G for safety):
   `resize2fs /dev/pve/vm-<CTID>-disk-0 <TARGET-1>G`
5. Shrink the LVM volume: `lvreduce -L <TARGET>G /dev/pve/vm-<CTID>-disk-0`
6. Sync Proxmox config: `pct resize <CTID> rootfs <TARGET>G`
7. Start + verify: `pct start <CTID> && pct exec <CTID> -- df -h /`

## Gotcha: MMP lock ("Multi-Mount Protection")
If `e2fsck` refuses with `MMP: e2fsck being run while checking MMP block`, clear it
(only when you're sure the FS isn't mounted on any node):

    tune2fs -f -E clear_mmp /dev/pve/vm-<CTID>-disk-0

Then re-run step 3 onward.

## Recovery
If the container won't start after a failed resize, restore the backup from
Datacenter → storage → Backups → Restore. ~2 minutes back to the prior state.

## Result (2026-06-02)
All six containers resized successfully; ~54 GB allocated reclaimed.
