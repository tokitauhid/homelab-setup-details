# Crafty Controller 4 & Playit.gg (Minecraft Server Manager)

Crafty Controller 4 is a comprehensive web panel and launcher for hosting, scheduling, and updating Minecraft Java and Bedrock servers with automated backups and Playit.gg public tunneling.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Crafty Web Panel** | `https://<SERVER_IP>:8443` |
| **Default Java Port** | `25565/tcp` |
| **Default Bedrock Port** | `19132/udp` |
| **Tunneling Agent** | `Playit.gg` (Runs in host network mode) |

---

## Deployment & Setup

1. Configure `.env`:
   ```bash
   cp .env.example .env
   ```
2. Start the stack:
   ```bash
   docker compose up -d
   ```
3. Open `https://<SERVER_IP>:8443` (accept self-signed SSL warning) and complete admin registration.
4. If using **Playit.gg**, view the playit container logs to claim your tunnel:
   ```bash
   docker logs minecraft-playit
   ```
