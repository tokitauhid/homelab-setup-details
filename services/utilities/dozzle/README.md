# Dozzle (Real-Time Docker Log Viewer)

Lightweight, web-based real-time log viewer for all Docker containers with search, filtering, and live CPU/RAM metrics.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `amir20/dozzle:latest` |
| **Web Interface** | `http://<SERVER_IP>:8888` |

---

## Deployment

```bash
docker compose up -d
```

Open `http://<SERVER_IP>:8888` in your browser to inspect live container logs.
