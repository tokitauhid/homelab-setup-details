# Homepage (Unified Dashboard & Widget Aggregator)

A modern, highly customizable application dashboard and startpage with direct Docker socket integration, API health checks, and system resource monitors.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `ghcr.io/gethomepage/homepage:latest` |
| **Web Interface** | `http://<SERVER_IP>:3000` |
| **Docker Socket** | `/var/run/docker.sock` (mounted read-only) |

---

## Deployment & Configuration

1. Copy `.env.example`:
   ```bash
   cp .env.example .env
   ```
2. Start the dashboard:
   ```bash
   docker compose up -d
   ```
3. Edit the YAML files in `./config/` (`services.yaml`, `widgets.yaml`, `settings.yaml`, `bookmarks.yaml`) to add your homelab services.
