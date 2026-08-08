# Syncthing (Continuous P2P File Sync)

Open-source, decentralized continuous file synchronization system that synchronizes files in real time between laptops, phones, servers, and desktops without third-party cloud intermediaries.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `lscr.io/linuxserver/syncthing:latest` |
| **Web Interface** | `http://<SERVER_IP>:8384` |
| **Sync Protocol** | `22000/tcp`, `22000/udp` |
| **Local Discovery** | `21027/udp` |

---

## Deployment

1. Configure `.env`:
   ```bash
   cp .env.example .env
   ```
2. Start the service:
   ```bash
   docker compose up -d
   ```
3. Navigate to `http://<SERVER_IP>:8384` to pair your devices via their unique Device IDs.
