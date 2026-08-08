# Nginx Proxy Manager (NPM)

Nginx Proxy Manager provides a web-based interface for managing reverse proxy hosts, redirect hosts, streams, and automated SSL certificate acquisition via Let's Encrypt.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `jc21/nginx-proxy-manager:latest` |
| **Admin Web UI** | `http://<SERVER_IP>:81` |
| **HTTP Traffic** | `http://<SERVER_IP>:80` |
| **HTTPS Traffic** | `https://<SERVER_IP>:443` |
| **Default Login** | Email: `admin@example.com` <br> Password: `changeme` |

---

## Directory Structure

```plaintext
nginx-proxy-manager/
├── docker-compose.yml
├── .env.example
├── README.md
├── data/              # SQLite database and Nginx virtual host configurations
└── letsencrypt/       # Let's Encrypt SSL certificates and renewal keys
```

---

## Deployment Instructions

1. **Configure Environment**:
   ```bash
   cp .env.example .env
   ```

2. **Start the Stack**:
   ```bash
   docker compose up -d
   ```

3. **Initial Configuration**:
   - Navigate to `http://<SERVER_IP>:81`.
   - Log in with the default credentials (`admin@example.com` / `changeme`).
   - Immediately update the administrator email and choose a strong password.

4. **Adding Proxy Hosts**:
   - Under **Hosts** > **Proxy Hosts** > **Add Proxy Host**:
     - **Domain Names**: `jellyfin.local` (or your chosen hostname).
     - **Scheme**: `http`
     - **Forward Hostname / IP**: Container LAN IP (e.g. `192.168.0.225` or container network alias).
     - **Forward Port**: Target container port (e.g. `8096` for Jellyfin).
     - Enable **Block Common Exploits** and **Websockets Support** if required.
