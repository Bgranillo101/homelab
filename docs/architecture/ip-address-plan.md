# IP address plan

## Infrastructure
| Host | Address |
|---|---|
| Home router / gateway | 192.168.0.1 |
| Proxmox host (bgranillo) | 192.168.0.13 |
| pfSense WAN | 192.168.0.164 |

## VLAN zones (live)
| VLAN | Name | Subnet | Gateway |
|---|---|---|---|
| 10 | SERVICES | 10.10.10.0/24 | 10.10.10.1 |
| 20 | TRUSTED-MGMT | 10.10.20.0/24 | 10.10.20.1 |
| 30 | VULNERABLE | 10.10.30.0/24 | 10.10.30.1 |
| 40 | MONITORING | 10.10.40.0/24 | 10.10.40.1 |

pfSense LAN/management: 10.10.10.1 on the VLAN-aware bridge `vmbr1`; WAN via `vmbr0`.

## Host assignments (live)
| Host | VLAN | Address |
|---|---|---|
| pihole (CT 100) | 10 SERVICES | 10.10.10.100 |
| nextcloud (CT 101) | 10 SERVICES | 10.10.10.101 |
| jellyfin (CT 102) | 10 SERVICES | 10.10.10.102 |
| uptime-kuma (CT 103) | 10 SERVICES | 10.10.10.103 |
| portainer (CT 104) | 10 SERVICES | 10.10.10.104 |
| homepage (CT 105) | 10 SERVICES | 10.10.10.105 |
| wazuh (VM 201) | 40 MONITORING | 10.10.40.10 |
