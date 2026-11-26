# Firefox (Containerized Remote Web Browser)

A full-fledged Firefox instance running inside Docker, streamed to any modern browser over WebRTC/KasmVNC for secure, sandboxed remote browsing.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `lscr.io/linuxserver/firefox:latest` |
| **Web Interface** | `http://<SERVER_IP>:3002` |
| **HTTPS Interface** | `https://<SERVER_IP>:3003` |

---

## Deployment

1. Copy `.env.example`:
   ```bash
   cp .env.example .env
   ```
2. Start the browser:
   ```bash
   docker compose up -d
   ```
3. Open `http://<SERVER_IP>:3002` in any browser.
