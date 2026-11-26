# Vaultwarden (Self-Hosted Bitwarden Password Manager)

Lightweight, Rust-based Bitwarden-compatible server (`vaultwarden/server:latest`) deployed on TrueNAS (`<SERVER_IP>`) via Dockge with LAN self-signed TLS (`ROCKET_TLS`), public Cloudflare Zero Trust Tunnel ingress, and an automated three-layer nightly backup & Bitwarden Cloud sync pipeline.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `vaultwarden/server:latest` |
| **Container Name** | `vaultwarden` (`user: "568:568"` / `apps:apps`) |
| **LAN Web Vault** | `https://<SERVER_IP>:8222` *(self-signed TLS via `ROCKET_TLS`)* |
| **Public URL** | `https://vaultwarden.yourdomain.com` *(Cloudflare Zero Trust Tunnel)* |
| **Dockge Stack Path** | `/mnt/main_pool/Docker_Configs/stacks/vaultwarden/compose.yaml` |
| **Persistent Data** | `/mnt/main_pool/Docker_Configs/vaultwarden` (`db.sqlite3`, `rsa_key.pem`, `ssl/`, `attachments/`, `sends/`) |
| **Registration** | `SIGNUPS_ALLOWED=false` *(locked down after creating account)* |

---

## TLS & Cloudflare Tunnel Architecture

### Why `ROCKET_TLS` Is Configured
Bitwarden's Web Vault requires a browser HTTPS secure context (`window.crypto.subtle`) — plain `http://<SERVER_IP>:8222` will reject login and account creation.
- Self-signed TLS certs (`ssl/cert.pem`, `ssl/key.pem`, 3650 days, SAN: `IP:<SERVER_IP>,DNS:truenas.local,DNS:truenas,DNS:localhost`) are mounted at `/data/ssl/` inside the container and loaded via:
  ```text
  ROCKET_TLS={certs="/data/ssl/cert.pem",key="/data/ssl/key.pem"}
  ```
- **Cloudflare Tunnel Configuration (`vaultwarden.yourdomain.com`):**
  - **Service Type:** `HTTPS` → `<SERVER_IP>:8222`
  - **TLS Settings:** **No TLS Verify** = **ON** *(required so `cloudflared` trusts Vaultwarden's self-signed origin cert without throwing `502 Bad Gateway` / `x509: certificate signed by unknown authority`)*.

---

## Automated Nightly Backup & `systemd-creds` Architecture

TrueNAS Cloud Sync (`Google Drive - /mnt/main_pool`) runs daily at **`00:00`** and explicitly excludes `/Docker_Configs/**` (only backing up stack Compose `.yaml`/`.env` files) plus `*.sqlite3-wal` / `*.sqlite3-shm`. To back up Vaultwarden safely without storing plaintext credentials on any synced pool:

### Directory Isolation
| Path | Synced to Google Drive? | Purpose |
| :--- | :--- | :--- |
| `/mnt/main_pool/Docker_Configs/vaultwarden/backup-tools/` | **No** (excluded, `root:root 0700`) | Contains `bw` CLI binary (`2026.11.0`), `vw-backup.sh`, `vw-cloud-sync.py`, `vw-seal-creds`, `cloud-sync-map.json`, and encrypted `bw.cred`. |
| `/var/lib/systemd/credential.secret` & `/data/systemd-credential.secret` | **No** (`boot-pool`, `0400`) | Host-bound `systemd-creds` secret key (backed up on TrueNAS upgrade-persistent `/data`). |
| `/mnt/main_pool/user/Backups/vaultwarden/` | **Yes** (`00:00` Google Drive sync) | Encrypted SQLite snapshots, encrypted `bw` JSON exports, and `LAST_BACKUP_STATUS.txt`. |

### Schedule & Three Backup Layers (`23:50` Daily — TrueNAS Cron Job ID `1`)
Runs every night at **`23:50`** (10 minutes before the `00:00` Google Drive sync) via `/mnt/main_pool/Docker_Configs/vaultwarden/backup-tools/vw-backup.sh`:

1. **Layer 1 — Native SQLite Hot-Backup (`sqlite/`):**
   - Uses `sqlite3 .backup` into RAM (`/dev/shm/vw-stage-*`) so live writes never corrupt the snapshot.
   - Runs `PRAGMA integrity_check;` and checks non-zero file size.
   - Archives `db.sqlite3`, `rsa_key.pem`, and optional `attachments/`, `sends/`, `config.json` into `vaultwarden-sqlite-YYYY-MM-DD.tar.gz` + `latest-sqlite.tar.gz` (14-day retention).
2. **Layer 2 — Bitwarden CLI Encrypted Export (`bw-exports/`):**
   - Decrypts `bw.cred` strictly into RAM via `systemd-creds decrypt --name=vaultwarden-bw` (zero plaintext password on disk).
   - Uses RAM-only `BITWARDENCLI_APPDATA_DIR` (`/dev/shm/vw-bw-*`) and trusts the self-signed cert via `NODE_EXTRA_CA_CERTS`.
   - Runs headless `bw login --apikey` (`BW_CLIENTID` / `BW_CLIENTSECRET`) + `bw unlock` (so accounts with 2FA enabled work completely unattended) → `bw sync` → `bw export --format encrypted_json` (password-protected, **not** account-restricted, so it can be imported directly into `bitwarden.com` or any new server) → `bw logout`.
   - Saves `vaultwarden-bw-YYYY-MM-DD.json` + `latest-bw.json` (14-day retention) after validating `"encrypted": true`.
3. **Layer 3 — Live Bitwarden Cloud Delta-Sync (`https://vault.bitwarden.com`):**
   - Uses a separate RAM-only `BITWARDENCLI_APPDATA_DIR` (`/dev/shm/vw-bw-cloud-*`) with public CAs (`NODE_EXTRA_CA_CERTS` unset).
   - Logs into `vault.bitwarden.com` using the sealed OAuth 2.0 API key (`cloud_client_id: user.<uuid>`, `cloud_client_secret: <client_secret>`) and unlocks with `cloud_password`.
   - Invokes `vw-cloud-sync.py` to compare folders and items between local Vaultwarden and `vault.bitwarden.com` (tracked via `cloud-sync-map.json` + normalized content fingerprint):
     - Creates any missing folders/items on `vault.bitwarden.com`.
     - Updates any changed items in place.
     - Skips identical items (zero duplicates).
   - Reports `BW_CLOUD_SYNC: OK (created=X, updated=Y, unchanged=Z, total=N)` in `LAST_BACKUP_STATUS.txt`.

### One-Time Credential Sealing (`vw-seal-creds`)
Run interactively once via SSH on TrueNAS so your Master Password is entered invisibly and sealed via `systemd-creds`:
```bash
ssh <user>@<SERVER_IP> -t 'sudo /mnt/main_pool/Docker_Configs/vaultwarden/backup-tools/vw-seal-creds'
```

---

## 3-Channel Failure Alerting

If **any** step in `vw-backup.sh` fails (SQLite integrity error, `systemd-creds` failure, `bw` login/sync/export error, or disk write failure), an `ERR` trap fires:
1. **Instant Email (`you@example.com`):** Sends `[TrueNAS] Vaultwarden Backup FAILED` via TrueNAS's configured Gmail OAuth (`midclt call -j mail.send`) with timestamp, exit code, and failed line/command.
2. **TrueNAS Cron Jobs UI Status:** Prints failure details to `stderr` (`stderr: false`) and exits non-zero so **System → Advanced → Cron Jobs** marks the run `FAILED`.
3. **Status File:** Writes `STATUS: FAILED` diagnostics to `/mnt/main_pool/user/Backups/vaultwarden/LAST_BACKUP_STATUS.txt` (synced to Google Drive).

---

## Disaster Recovery / Restore

- **Restore full Vaultwarden instance (from `sqlite/vaultwarden-sqlite-YYYY-MM-DD.tar.gz`):**
  1. Stop the `vaultwarden` container.
  2. Extract the `.tar.gz` contents into `/mnt/main_pool/Docker_Configs/vaultwarden/` and fix permissions (`chown -R 568:568 /mnt/main_pool/Docker_Configs/vaultwarden`).
  3. Start the container.
- **Import directly into Bitwarden Cloud or a fresh server (from `bw-exports/vaultwarden-bw-YYYY-MM-DD.json`):**
  1. Open **Tools → Import Data** in Bitwarden (`vault.bitwarden.com` or a new Vaultwarden server).
  2. Choose **Bitwarden (json)**, select the `.json` file, and enter your Master Password to decrypt and import all items, TOTP keys, passkeys, and folders.
