# ADR-0002: LXC for services, VMs for security infrastructure

- **Status:** Accepted
- **Date:** 2026-06-02

## Context
Choosing the right virtualization type per workload on constrained hardware (16 GB RAM).

## Decision
LXC containers for application services; full VMs for security infrastructure
(pfSense, Wazuh, Kali, target machines).

## Alternatives considered
- **All VMs** — heavier RAM/CPU overhead; the six service containers would each need a
  full guest kernel (~200–400 MB overhead per VM vs. near-zero for LXC).
- **All LXC** — cannot run pfSense (requires its own kernel and network stack); Kali and
  Wazuh also benefit from kernel isolation and custom networking unavailable in LXC.

## Consequences
Low RAM overhead for the six existing services (~3 GB total observed). pfSense, Wazuh,
and Kali run as VMs for kernel isolation and custom networking. The clear boundary
("services = LXC, security = VM") also makes the architecture easier to explain.
