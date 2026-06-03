# configs/

Only sanitized configuration exports belong here. Never commit pfSense `config.xml` with
password hashes or pre-shared keys; strip secrets first and replace with `REDACTED`.

Raw exports (e.g., `config-backup*.xml`) are gitignored. If you're adding a config file,
verify it contains no passwords, API keys, or pre-shared keys before staging.

## Subdirectories

| Dir | Contents |
|---|---|
| `proxmox/` | Proxmox node/container config snippets (sanitized) |
| `pfsense/` | pfSense configuration exports (sanitized only) |
| `suricata/` | Custom Suricata rule files and `suricata.yaml` overrides |
| `docker/` | Any Docker Compose files used in the lab |
