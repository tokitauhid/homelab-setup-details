# AdGuard Home (Network-Wide Ad Blocker & DNS Server)

Self-hosted network-wide ad/tracker blocker and local DNS server with DoH/DoT upstream support and clean web administration.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `adguard/adguardhome:latest` |
| **DNS Port** | `53` (TCP/UDP) |
| **Web Admin UI** | `http://<SERVER_IP>:8080` |

> **Note:** Only run one DNS server on host port `53` at a time (AdGuard Home *or* Pi-hole).

---

## Deployment

1. Copy `.env.example`:
   ```bash
   cp .env.example .env
   ```
2. Start AdGuard Home:
   ```bash
   docker compose up -d
   ```
3. Open `http://<SERVER_IP>:8080` to complete initial setup.
