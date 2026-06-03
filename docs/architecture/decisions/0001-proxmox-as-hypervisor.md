# ADR-0001: Proxmox as hypervisor

- **Status:** Accepted
- **Date:** 2026-06-02

## Context
Needed a base OS for the homelab that supports both lightweight containers and full VMs
on a single consumer-grade SFF machine.

## Decision
Proxmox VE — installed bare-metal on the SSD.

## Alternatives considered
- **Bare-metal Ubuntu + Docker** — less isolation between workloads, no native VM
  management GUI, harder to snapshot/migrate individual services.
- **ESXi (VMware)** — licensing friction for home use; free tier dropped in 2024.

## Consequences
Native VM + LXC support in a single platform; snapshots, backup tooling, and a web UI
are included. Tradeoff: ~1 GB of hypervisor overhead and a Proxmox-specific skill set
(though transferable to enterprise KVM/QEMU workflows).
