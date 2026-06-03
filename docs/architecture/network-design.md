# Network Design

## Current state (pre-segmentation)

Everything is on a single flat `192.168.0.0/24` network — home router does DHCP.
One Proxmox bridge (`vmbr0`) bound to the onboard NIC. No VLANs, no firewall rules,
no IDS. All containers can reach each other and the internet without restriction.

**Risk:** any compromised service can reach all others (lateral movement), and there
is no visibility into east-west traffic.

## Target architecture

### Bridges

| Bridge | Purpose |
|---|---|
| `vmbr0` | Uplink to home router / WAN. pfSense WAN interface. |
| `vmbr1` | VLAN-aware lab-side bridge. pfSense LAN; all zone trunk ports. |

### VLAN zones

| VLAN | Name | Subnet | Gateway | Contents |
|---|---|---|---|---|
| 10 | SERVICES | 10.10.10.0/24 | 10.10.10.1 | The 6 existing LXCs (Pi-hole, Nextcloud, Jellyfin, Uptime-Kuma, Portainer, Homepage) |
| 20 | TRUSTED-MGMT | 10.10.20.0/24 | 10.10.20.1 | Admin jump host; access into SERVICES and MONITORING |
| 30 | VULNERABLE | 10.10.30.0/24 | 10.10.30.1 | Kali attacker VM + deliberately-weak targets |
| 40 | MONITORING | 10.10.40.0/24 | 10.10.40.1 | Wazuh SIEM |

pfSense is the default gateway for all zones. It runs Suricata on its LAN interface
to inspect all inter-zone traffic.

### Firewall posture

**Default-deny** between all zones. Explicit allow rules only for:
- TRUSTED-MGMT → SERVICES (management/admin traffic)
- TRUSTED-MGMT → MONITORING (Wazuh UI access)
- SERVICES → MONITORING (Wazuh agent → manager traffic, port 1514/1515)
- VULNERABLE zone is fully isolated — outbound internet blocked; no inbound from other zones

### Security tooling placement

| Tool | Location | Role |
|---|---|---|
| pfSense | VM, vmbr0 + vmbr1 | Routing, firewall, Suricata IDS |
| Suricata | Inside pfSense | Inline IDS on all inter-zone flows |
| Wazuh | VM, VLAN 40 | SIEM: Suricata EVE JSON + host agents |
| Kali | VM, VLAN 30 | Attacker |
| Target VM | VM, VLAN 30 | Intentionally vulnerable target |

## Migration plan (Phase B)

1. Create `vmbr1` as VLAN-aware bridge in Proxmox.
2. Spin up pfSense VM; assign WAN (`vmbr0`) and LAN (`vmbr1`).
3. Create VLAN 10, 20, 30, 40 sub-interfaces on pfSense.
4. Move each LXC network interface to `vmbr1` with VLAN tag 10.
5. Apply default-deny firewall policy; add TRUSTED-MGMT allow rules.
6. Verify service reachability from TRUSTED-MGMT before proceeding.
