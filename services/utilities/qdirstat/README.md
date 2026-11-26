# QDirStat (Graphical Disk Usage Analyzer)

Graphical directory statistics and treemap disk usage analyzer running in Docker and accessible via browser to inspect large files and storage pools.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `lscr.io/linuxserver/qdirstat:latest` |
| **HTTP Interface** | `http://<SERVER_IP>:3010` |
| **HTTPS Interface** | `https://<SERVER_IP>:3011` |
| **Mounted Pool** | `/mnt/Jellyfin` → `/Jellyfin_Pool` |

---

## Deployment

```bash
cp .env.example .env
docker compose up -d
```

Open `http://<SERVER_IP>:3010` in your browser.
