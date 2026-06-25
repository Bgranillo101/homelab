# Runbook: Ship Suricata EVE JSON from pfSense into Wazuh (file-pull)

**When to use:** you have Suricata running on pfSense (see
[suricata-ids-setup.md](suricata-ids-setup.md)) and need its `eve.json` alerts in the Wazuh
SIEM — **without installing anything on the firewall**. Syslog forwarding does not work
(pfSense does not forward Suricata's `LOCAL1` facility), and a Wazuh agent on a FreeBSD
production router is too risky. This runbook pulls the file over a locked-down SSH key.
See [ADR-0007](../architecture/decisions/0007-pull-based-eve-ingestion.md) for the why.

**Direction matters:** the pull runs *from* Wazuh (MONITORING, 10.10.40.10) *to* pfSense
at its MONITORING gateway `10.10.40.1`. MONITORING→any is `Pass`; SERVICES→MONITORING on 22
is correctly blocked by default-deny, so the pull uses the allowed outbound direction.

## Steps

1. **Enable SSH on pfSense.** System → Advanced → Admin Access → check **Secure Shell**.

2. **Generate a dedicated keypair on the Wazuh box** (private key never leaves Wazuh):

       ssh-keygen -t ed25519 -f /root/.ssh/suricata_pull -C wazuh-eve-pull -N ""

3. **Install the public key on the pfSense `admin` user, locked with a forced command** so
   it can do exactly one thing — read the EVE file. pfSense UI → System → User Manager →
   `admin` → Authorized SSH Keys, paste a single line (replace the placeholder with the
   real `suricata_pull.pub` contents):

       command="/bin/cat /var/log/suricata/suricata_vtnet147403/eve.json",restrict ssh-ed25519 <PUBLIC_KEY> wazuh-eve-pull

   `restrict` disables shell, port-forwarding, and agent forwarding — the key can only run
   that one `cat`. Confirm the EVE path matches your interface dir under
   `/var/log/suricata/suricata_<iface><id>/`.

4. **Verify the pull works** from Wazuh:

       ssh -i /root/.ssh/suricata_pull admin@10.10.40.1 | head

   You should see EVE JSON lines and nothing else.

5. **Install the incremental shipper** `/usr/local/bin/pull-eve.sh` (`chmod 0700`,
   root-owned). It pulls the full file, appends only lines beyond a saved offset, and
   resets if the remote file shrinks (log rotation):

       #!/bin/sh
       SRC_KEY=/root/.ssh/suricata_pull
       SRC=admin@10.10.40.1
       DST=/var/log/suricata-pfsense/eve.json
       MARK=/var/log/suricata-pfsense/.lastcount
       mkdir -p /var/log/suricata-pfsense
       TMP=$(mktemp)
       ssh -i "$SRC_KEY" -o StrictHostKeyChecking=accept-new "$SRC" > "$TMP" || { rm -f "$TMP"; exit 1; }
       NEW=$(wc -l < "$TMP")
       OLD=$(cat "$MARK" 2>/dev/null || echo 0)
       if [ "$NEW" -lt "$OLD" ]; then OLD=0; fi      # remote rotated → re-read from top
       if [ "$NEW" -gt "$OLD" ]; then
         tail -n +"$((OLD + 1))" "$TMP" >> "$DST"
         echo "$NEW" > "$MARK"
       fi
       rm -f "$TMP"

   Prove dedup: run it twice back-to-back; the line count should not double.

6. **Schedule it every minute.** Do **not** use `echo | crontab -` from the noVNC console
   (quote-mangling). Write the line to a file with single quotes, then load it:

       printf '* * * * * /usr/local/bin/pull-eve.sh\n' > /root/eve.cron
       crontab /root/eve.cron

7. **Ingest the local file in Wazuh.** In `/var/ossec/etc/ossec.conf` add a `localfile`
   block and **remove any stale syslog `<remote>` block** from a prior syslog attempt:

       <localfile>
         <log_format>json</log_format>
         <location>/var/log/suricata-pfsense/eve.json</location>
       </localfile>

8. **Validate the config offline, then restart:**

       /var/ossec/bin/wazuh-analysisd -t      # must report config OK
       systemctl restart wazuh-manager

9. **Verify end to end.**
   - Decode + rule match on a real line:

         /var/ossec/bin/wazuh-logtest

     paste an alert line; expect Phase 2 (Suricata decoder) and a Phase 3 rule match (e.g.
     rule 86601, groups `ids,suricata`).
   - Dashboard → Threat Hunting, query `rule.groups:suricata` — alerts appear, charted
     over time, sourced from the pfSense puller's agent.

## Gotchas
- **Syslog is a dead end.** pfSense emits Suricata EVE under the FreeBSD `LOCAL1` facility,
  which pfSense remote-logging does not forward. Use the file pull, not syslog.
- **noVNC cannot copy and mangles pasted quotes/slashes.** Transcribing the SSH public key
  by eye introduced a wrong character (`Permission denied (publickey)`). Route key material
  through a real terminal (e.g. `scp` the `.pub` to the Proxmox host — an allowed outbound
  direction — and copy it there). Install cron via a single-quoted file + `crontab <file>`,
  not `echo | crontab -`.
- **`wazuh-logtest -t` is not a config validator** — that flag does not exist. Use
  `wazuh-analysisd -t`.
- **The Wazuh collector tails new appends only**, not the existing backlog. Pre-existing
  lines are skipped until a manager restart plus freshly-appended alerts flow through.
- **Revert `<logall>` to `no`** on the manager after any diagnostics — leaving it `yes`
  grows the disk.

## Result (2026-06-25)
Pipeline live end to end: `pull-eve.sh` ships EVE every minute over the forced-command key,
Wazuh ingests it via `localfile`, and the dashboard shows **1,783 Suricata alerts**
(rule 86601). Nothing is installed on the firewall. Current alerts are STREAM-engine
baseline events; attack-signature detection is Phase E.
