# Watchtower (Automated Docker Image Updater)

Automatically checks for updated Docker images on a cron schedule, recreates containers with their existing configuration, and prunes old unused images.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `nickfedor/watchtower:latest` |
| **Schedule** | Daily at `03:00` (`0 0 3 * * *`, `Asia/Dhaka`) |
| **Cleanup** | Enabled (`WATCHTOWER_CLEANUP=true`) |

---

## Deployment

```bash
cp .env.example .env
docker compose up -d
```
