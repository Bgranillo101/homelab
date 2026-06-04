# Homelab SOC — Service Links & Access Reference

> **How to use:** Tunnel must be running before accessing any `localhost` link.
> Start it with: `ssh -L 9443:10.10.10.1:443 -N root@192.168.0.13`
> If port busy: `pkill -f "ssh -L 9443"` then re-run.

---

## Infrastructure

| Service | URL | Auth | Status |
|---|---|---|---|
| Proxmox Web UI | https://192.168.0.13:8006 | root / password manager | ✅ Live |
| pfSense Web UI | https://localhost:9443 | admin / password manager | ✅ Live (tunnel required) |

---

## pfSense Pages

| Page | URL | Notes |
|---|---|---|
| Dashboard | https://localhost:9443/index.php | Overview, interface status |
| Interface Assignments | https://localhost:9443/interfaces_assign.php | WAN / LAN / OPT assignments |
| VLANs | https://localhost:9443/interfaces_vlan.php | Tags 20, 30, 40 on vtnet1 |
| Interface: TRUSTEDMGMT | https://localhost:9443/interfaces.php?if=opt1 | 10.10.20.1/24 |
| Interface: VULNERABLE | https://localhost:9443/interfaces.php?if=opt2 | 10.10.30.1/24 |
| Interface: MONITORING | https://localhost:9443/interfaces.php?if=opt3 | 10.10.40.1/24 |
| DHCP Server: LAN | https://localhost:9443/services_dhcp.php?if=lan | 10.10.10.100–199 |
| DHCP Server: TRUSTEDMGMT | https://localhost:9443/services_dhcp.php?if=opt1 | 10.10.20.100–199 |
| DHCP Server: VULNERABLE | https://localhost:9443/services_dhcp.php?if=opt2 | 10.10.30.100–199 |
| DHCP Server: MONITORING | https://localhost:9443/services_dhcp.php?if=opt3 | 10.10.40.100–199 |
| Firewall Rules | https://localhost:9443/firewall_rules.php | ⏳ Next session |
| Suricata | https://localhost:9443/suricata/suricata_interfaces.php | ⏳ Phase D |

---

## Lab Services (LXC — currently on flat network, pre-migration)

> These will move to `10.10.10.x` (VLAN 10) after LXC migration in Phase B.

| Service | CTID | Target IP (post-migration) |
|---|---|---|
| Pi-hole | 100 | 10.10.10.100 (static) |
| Nextcloud | 101 | TBD |
| Jellyfin | 102 | TBD |
| Uptime-Kuma | 103 | TBD |
| Portainer | 104 | TBD |
| Homepage | 105 | TBD |

---

## Coming Soon

| Service | Expected URL | Phase |
|---|---|---|
| Wazuh Dashboard | https://10.10.40.x | Phase C |
| Suricata (pfSense pkg) | https://localhost:9443/suricata/ | Phase D |
| Kali Linux | 10.10.30.x (SSH/VNC) | Phase E |
| Vulnerable Target VM | 10.10.30.x | Phase E |

---

## GitHub

| Resource | URL |
|---|---|
| Repo | https://github.com/bgranillo05/homelab-soc |

---

*Last updated: 2026-06-03 — VLANs configured, DHCP live on all zones.*
