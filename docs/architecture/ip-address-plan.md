# IP address plan

## Current (pre-segmentation)
| Host | Address |
|---|---|
| Home router / gateway | 192.168.0.1 |
| Proxmox host (bgranillo) | 192.168.0.13 |
| LXC services | DHCP on 192.168.0.0/24 (to be migrated) |

## Target (post-segmentation)
| VLAN | Name | Subnet | Gateway |
|---|---|---|---|
| 10 | SERVICES | 10.10.10.0/24 | 10.10.10.1 |
| 20 | TRUSTED-MGMT | 10.10.20.0/24 | 10.10.20.1 |
| 30 | VULNERABLE | 10.10.30.0/24 | 10.10.30.1 |
| 40 | MONITORING | 10.10.40.0/24 | 10.10.40.1 |

pfSense LAN/management: 10.10.10.1 on the VLAN-aware bridge `vmbr1`; WAN via `vmbr0`.
