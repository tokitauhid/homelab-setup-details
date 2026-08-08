# Speedtest Tracker (ISP Performance Monitor)

Automated internet speed and latency monitoring application that runs periodic speed tests against Ookla's Speedtest network with historical charts and SLA tracking.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `lscr.io/linuxserver/speedtest-tracker:latest` |
| **Web Interface** | `http://<SERVER_IP>:8084` |
| **Database** | Built-in SQLite |

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
3. Open `http://<SERVER_IP>:8084` and view the automated benchmark graph.
