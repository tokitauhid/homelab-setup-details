# qBittorrent (Standalone Client with ThemePark)

Modern, lightweight BitTorrent client with integrated ThemePark Catppuccin-Mocha styling and rich WebUI features.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `lscr.io/linuxserver/qbittorrent:latest` |
| **Web Interface** | `http://<SERVER_IP>:8082` |
| **Default Login** | Username: `admin` <br> Temporary Password: *(generated in container startup logs)* |
| **Theme** | Catppuccin Mocha via `ghcr.io/themepark-dev/theme.park:qbittorrent` |

---

## Initial Setup & Credentials

On initial startup, qBittorrent generates a temporary random password for the `admin` user printed to its logs:

```bash
docker logs qbittorrent 2>&1 | grep "A temporary password"
```

Log in via `http://<SERVER_IP>:8082`, navigate to **Options** > **Web UI**, and update your username and password.

---

## Deployment

1. Copy `.env.example`:
   ```bash
   cp .env.example .env
   ```
2. Start the container:
   ```bash
   docker compose up -d
   ```
