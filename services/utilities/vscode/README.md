# Visual Studio Code (Full Desktop IDE in Browser)

Full official Microsoft Visual Studio Code desktop application running inside Docker and streamed to any modern browser via Selkies WebRTC (`lscr.io/linuxserver/vscode`). Unlike `code-server` (Code - OSS), this runs the genuine VS Code binary with native **Microsoft/GitHub Account Sync**, **Profiles**, **official Marketplace**, and **GitHub Copilot**.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `lscr.io/linuxserver/vscode:latest` |
| **HTTP Interface** | `http://<SERVER_IP>:3020` |
| **HTTPS Interface** | `https://<SERVER_IP>:3021` (self-signed) |
| **Public Ingress** | Cloudflare Tunnel (`https://vscode.yourdomain.com`) |
| **Workspace Mount** | `/mnt/main_pool/user` → `/config/workspace` |

---

## Deployment

1. Copy `.env.example` and set strong credentials:
   ```bash
   cp .env.example .env
   ```
2. Start the container:
   ```bash
   docker compose up -d
   ```
3. Open `https://<SERVER_IP>:3021` (or `http://<SERVER_IP>:3020`) in your browser.

---

## Account Sync Persistence

In headless containers without an unlocked desktop keyring, VS Code may prompt you to sign back into Settings Sync after container restarts. To persist your login token inside `/config`:

1. Open the Command Palette (`Ctrl+Shift+P`).
2. Run **`Preferences: Configure Runtime Arguments`** (`argv.json`).
3. Add `"password-store": "basic"` and restart VS Code.

---

## Recommended Selkies Smoothness Settings

- **Video:** `H.264`, `60 FPS`, `CRF 25` (`26`–`28` over WAN), **Turbo ON**, **Use Paint-Overs ON** (`CRF 18`), **FullColor 4:4:4 OFF**, **10-Bit Color OFF**, **Multithreaded CPU Encoding ON**.
- **Screen:** **Use CSS cursors ON**, **Raw pointer motion OFF**, **HiDPI OFF** on 2K/4K scaled laptops, **Scale Locally ON**.
