# Cloudflare Tunnel (`cloudflared`)

Cloudflare Tunnel creates an encrypted outbound-only tunnel between your local environment and Cloudflare's edge network, exposing services without opening router firewall ports or sharing public IP addresses.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `cloudflare/cloudflared:latest` |
| **Outbound Connection** | Port `443/tcp` and `7844/udp` (QUIC) |
| **Public Ports Required** | **None** (Fully ingress firewalled) |

---

## Setup Guide

1. Log into your [Cloudflare Zero Trust Dashboard](https://one.dash.cloudflare.com/).
2. Navigate to **Networks** > **Tunnels** > **Create a tunnel** (Select **Cloudflared**).
3. Assign a name (e.g. `homelab-gateway`).
4. Copy the tunnel token provided in the Docker command modal.
5. Create your `.env` file:
   ```bash
   cp .env.example .env
   ```
6. Paste the token into `CLOUDFLARE_TUNNEL_TOKEN` inside `.env`.
7. Start the container:
   ```bash
   docker compose up -d
   ```
8. In the Cloudflare dashboard, add **Public Hostnames** pointing to your local services (e.g. `jellyfin.yourdomain.com` -> `http://192.168.0.225:8096`).
