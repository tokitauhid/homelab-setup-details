# QUI (Modern Web Interface for qBittorrent)

QUI (by Autobrr) is an ultra-fast, modern frontend for managing one or multiple qBittorrent instances with cross-seeding workflows and reactive UI components.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `ghcr.io/autobrr/qui:latest` |
| **Web Interface** | `http://<SERVER_IP>:8083` |
| **Upstream Target** | qBittorrent Web API (`http://<SERVER_IP>:8080` or `http://<SERVER_IP>:8082`) |

---

## Deployment & Linking

1. Copy environment template:
   ```bash
   cp .env.example .env
   ```
2. Start QUI:
   ```bash
   docker compose up -d
   ```
3. Open `http://<SERVER_IP>:8083` and add your qBittorrent instance URL, port, and credentials.
