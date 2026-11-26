# Homarr (Sleek Drag-and-Drop Dashboard)

Homarr is an intuitive, modern dashboard with 40+ built-in app integrations, 20,000+ icons, and drag-and-drop customization without requiring manual YAML editing.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `ghcr.io/homarr-labs/homarr:latest` |
| **Web Interface** | `http://<SERVER_IP>:7575` |
| **Integrations** | qBittorrent, Jellyfin, Pi-hole, Sonarr, Radarr, Prowlarr, Speedtest |

---

## Deployment

1. Copy `.env.example`:
   ```bash
   cp .env.example .env
   sed -i "s/change_this_to_a_random_32_byte_hex_key/$(openssl rand -hex 32)/g" .env
   ```
2. Start the container:
   ```bash
   docker compose up -d
   ```
3. Open `http://<SERVER_IP>:7575` and create your administrative account.
