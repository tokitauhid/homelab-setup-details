# NextExplorer (Web-Based File Browser)

A fast, responsive web interface for browsing, downloading, editing, and managing files across server directories directly in the browser.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `nxzai/explorer:latest` |
| **Web Interface** | `http://<SERVER_IP>:8085` |
| **Browse Root** | Mounted at `/mnt/Home` |

---

## Deployment

1. Set `NEXTEXPLORER_BROWSE_ROOT` to the directory on your host you wish to manage:
   ```bash
   cp .env.example .env
   ```
2. Start the container:
   ```bash
   docker compose up -d
   ```
