# Runbook: Deploy a Wazuh agent on an LXC container

**When to use:** enroll a Proxmox LXC service (SERVICES / VLAN 10) into the Wazuh
manager (MONITORING / VLAN 40, `10.10.40.10`) so it reports events to the SIEM.

**Tested on:** CT 100 (Pi-hole) and CT 101 (Nextcloud), Debian 12 LXC templates,
Wazuh agent 4.11.2. Both enrolled and showed **active** in the dashboard.

## Prerequisites
- Wazuh manager reachable at `10.10.40.10` on ports **1514** (events) and **1515** (enrollment).
- A pfSense rule allowing the agent traffic (see *Firewall* below). Without it the agent
  installs fine but **never connects** — default-deny silently drops SERVICES -> MONITORING.

## Firewall (do this once, not per agent)
On pfSense, **Firewall -> Rules -> LAN**, add a rule placed **above** the
`Block SERVICES -> MONITORING` rule (first match wins, top-down):

- Action: **Pass** | Protocol: **TCP**
- Source: `10.10.10.0/24` (SERVICES subnet — covers every service agent)
- Destination: `10.10.40.10` (single host: the manager)
- Destination port range: `1514` to `1515`
- Description: `Allow Wazuh agents (SERVICES) -> manager 1514-1515`

Apply. pfSense is stateful, so the manager's replies return automatically — no second rule.

## Steps (per container)
Run from the **Proxmox host**, replacing `<CTID>` and `<name>`:

1. Enter the container (avoids the LXC web-console freeze after network changes):
```
   pct enter <CTID>
```
2. **Pre-install `lsb-release`** — minimal LXC templates omit it and the agent depends on
   it. Installing it first prevents a half-failed unpack:
```
   apt-get update && apt-get install -y lsb-release
```
3. Download + install the agent in **one command** so the manager IP is baked in:
```
   wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.11.2-1_amd64.deb \
     && WAZUH_MANAGER='10.10.40.10' WAZUH_AGENT_NAME='<name>' dpkg -i ./wazuh-agent_4.11.2-1_amd64.deb
```
4. **Verify the manager IP was written** before starting:
```
   grep -A1 '<server>' /var/ossec/etc/ossec.conf
```
   Expect `<address>10.10.40.10</address>`. If it shows `MANAGER_IP`, fix it (Gotcha 2).
5. Enable + start, then check:
```
   systemctl daemon-reload && systemctl enable wazuh-agent && systemctl start wazuh-agent
   systemctl status wazuh-agent --no-pager
```
   Expect `active (running)`.
6. Confirm in the dashboard -> **Agents**: the container appears with status **active**.
   `exit` to leave the container.

## Gotchas (both hit during the real deploy)

### 1. `wazuh-agent depends on lsb-release; however: Package lsb-release is not installed`
Minimal Debian LXC templates ship without `lsb-release`, so `dpkg -i` unpacks the agent but
can't configure it (leaves it half-installed). Fix:
```
apt-get update && apt-get install -y lsb-release
apt-get install -f -y      # completes the half-finished agent config
```
Pre-installing `lsb-release` (step 2) avoids this entirely.

### 2. `ERROR: (4112): Invalid server address found: 'MANAGER_IP'`
If the agent was unpacked while broken (Gotcha 1) and only configured **later** with
`apt-get install -f`, the `WAZUH_MANAGER` environment variable is no longer present at
configure time — so the config keeps the literal placeholder `MANAGER_IP`. The agent then
refuses to start (`ERROR: (1215): No client configured. Exiting.`). Fix in place:
```
sed -i 's|<address>MANAGER_IP</address>|<address>10.10.40.10</address>|' /var/ossec/etc/ossec.conf
grep -A1 '<server>' /var/ossec/etc/ossec.conf   # confirm the IP
systemctl start wazuh-agent
```
Installing cleanly in one pass (step 3, after `lsb-release` is present) avoids this.

### 3. Locale warnings (`Cannot set LC_ALL to default locale ...`)
Cosmetic noise on minimal LXC templates; unrelated to Wazuh. Safe to ignore, or set a
locale with `apt-get install -y locales && dpkg-reconfigure locales`.

## Verifying connectivity from the agent side
Inside the container:
```
tail -n 20 /var/ossec/logs/ossec.log
```
Look for `Connected to the server` / agent-online lines, plus `syscheck` (FIM) and
`rootcheck` scans starting — those confirm the agent is enrolled and actively reporting.

## Result (2026-06-23)
Agents 001 (pihole, 10.10.10.100) and 002 (nextcloud, 10.10.10.101) enrolled and **active**.
File Integrity Monitoring and rootcheck running on both. Phase C SIEM operational.
