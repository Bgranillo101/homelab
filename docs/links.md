# Service Links & Quick Reference

## Access pfSense (tunnel required)

```bash
ssh -L 9443:10.10.10.1:443 -N root@192.168.0.13
```

Then browse to `https://localhost:9443`

## Management UIs

| Service | URL | Tunnel? | Status |
|---|---|---|---|
| Proxmox Web UI | https://192.168.0.13:8006 | No | ✅ Live |
| pfSense Dashboard | https://localhost:9443 | Yes | ✅ Live |
| pfSense → VLANs | https://localhost:9443/interfaces_vlan.php | Yes | ✅ Configured |
| pfSense → Firewall Rules | https://localhost:9443/firewall_rules.php | Yes | ✅ Rules live |
| pfSense → DHCP Server | https://localhost:9443/services_dhcp.php | Yes | ✅ Live |
| Wazuh Dashboard | https://10.10.40.10 (planned) | Yes | ⏳ Phase C |
| Suricata (pfSense pkg) | https://localhost:9443/suricata/ | Yes | ⏳ Phase D |

## Lab Services (now on VLAN 10 — 10.10.10.x)

| Service | IP | Port | Notes |
|---|---|---|---|
| Pi-hole | 10.10.10.100 | 80 | DNS + ad blocking |
| Nextcloud | 10.10.10.101 | 8081 | File storage |
| Jellyfin | 10.10.10.102 | 8096 | Media server |
| Uptime-Kuma | 10.10.10.103 | 3001 | Uptime monitoring |
| Portainer | 10.10.10.104 | 9000 | Docker management |
| Homepage | 10.10.10.105 | 80 | Dashboard |

> All services now behind pfSense. Access via SSH tunnel through Proxmox:
> `ssh -L <local_port>:<service_ip>:<service_port> -N root@192.168.0.13`

## GitHub

| Resource | URL |
|---|---|
| Repo | https://github.com/bgranillo05/homelab-soc |

*Last updated: 2026-06-05 — Phase B complete, all LXCs on VLAN 10.*
