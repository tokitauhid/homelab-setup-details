# The Arch Homelab: Unified Docker Architecture

![OS: Arch Linux](https://img.shields.io/badge/OS-Arch%20Linux-1793d1.svg?logo=arch-linux&logoColor=white)
![Hardware: CPU Only](https://img.shields.io/badge/Compute-CPU%20Only%20(Ryzen%203600)-blue)
![Architecture: Docker](https://img.shields.io/badge/Architecture-Docker%20Compose-2496ED.svg?logo=docker&logoColor=white)
![Management: Portainer](https://img.shields.io/badge/Management-Portainer-13BEF9.svg?logo=portainer&logoColor=white)
![Status: Active](https://img.shields.io/badge/Status-Production-green)

Self-hosted, CPU-optimized home server environment on vanilla Arch Linux. All 18 services run as independent Docker Compose stacks managed through **Portainer**, rooted under `/home/${USER}/docker/` for clean ownership, predictable backups, and secure remote access via Tailscale and Cloudflare Tunnel.

---

## System Specifications

| Component  | Details                                          |
| ---------- | ------------------------------------------------ |
| **OS**     | Vanilla Arch Linux (rolling release)             |
| **CPU**    | AMD Ryzen 5 3600 — 6 cores / 12 threads         |
| **RAM**    | 16 GB DDR4                                       |
| **GPU**    | NVIDIA GT 710 — display output only, no passthrough, no inference |
| **Network**| LAN `192.168.0.0/24`                             |
| **Root**   | `/home/${USER}/docker/`                          |

> [!IMPORTANT]
> Every stack expects a `.env` file alongside its `docker-compose.yml`. At minimum, define `TZ`, and any service-specific auth tokens before first deployment. Sensitive tokens should **never** be committed to version control.

---

## Network Architecture

```
┌─────────── Internet ───────────┐
│                                │
│  Cloudflare Tunnel (egress)    │
│        ↕                       │
│  Nginx Proxy Manager           │
│   :80 / :81 / :443            │
│        ↕                       │
├────── LAN 192.168.0.0/24 ─────┤
│                                │
│  Pi-hole (:53, :8053)          │◄── DNS for all LAN clients
│                                │
│  ┌── Media ──────────────────┐ │
│  │ Jellyfin        :8096     │ │
│  │ qBittorrent     :8082     │ │
│  │ QUI             :8083     │ │
│  └───────────────────────────┘ │
│                                │
│  ┌── AI & Knowledge ────────┐  │
│  │ Ollama          :11434   │  │
│  │ Open WebUI      :3001    │  │
│  │ SearXNG         :8081    │  │
│  │ Obsidian Sync   :5984    │  │
│  └──────────────────────────┘  │
│                                │
│  ┌── Utility ───────────────┐  │
│  │ Homepage        :3000    │  │
│  │ NextExplorer    :8085    │  │
│  │ Syncthing       :8384    │  │
│  │ Firefox         :3002    │  │
│  │ Speedtest       :8084    │  │
│  └──────────────────────────┘  │
│                                │
│  ┌── Gaming & Emulation ────┐  │
│  │ Crafty          :8443    │  │
│  │ Redroid         :5555    │  │
│  └──────────────────────────┘  │
│                                │
│  Tailscale (mesh VPN) ─────────┼──►  Remote access
└────────────────────────────────┘
```

---

## Service and Port Map

All web interfaces use dedicated, non-overlapping ports. Ports marked with `*` have been shifted from upstream defaults to avoid conflicts.

| Category    | Service             | Description                              | Port(s)              | Default Port |
| ----------- | ------------------- | ---------------------------------------- | --------------------- | ------------ |
| Networking  | Nginx Proxy Manager | Reverse proxy and SSL termination        | `80`, `81`, `443`     | —            |
| Networking  | Pi-hole             | Network-wide ad blocking and local DNS   | `53`, `8053`*         | `80`         |
| Networking  | Tailscale           | Secure mesh VPN for remote access        | `host network`        | —            |
| Networking  | Cloudflare Tunnel   | Outbound tunnel to Cloudflare proxy      | `none`                | —            |
| Media       | Jellyfin            | Media server — movies, TV, music (CPU transcode) | `8096`         | `8096`       |
| Media       | qBittorrent         | Torrent client for Linux ISOs            | `8082`*, `6881`       | `8080`       |
| Media       | QUI                 | Modern web frontend for qBittorrent      | `8083`*               | `80`         |
| AI / Data   | Ollama              | Local LLM inference API                  | `11434`               | `11434`      |
| AI / Data   | Open WebUI          | ChatGPT-style interface for Ollama       | `3001`*               | `8080`       |
| AI / Data   | Obsidian LiveSync   | CouchDB backend for Obsidian note sync   | `5984`                | `5984`       |
| AI / Data   | SearXNG             | Privacy-respecting metasearch engine     | `8081`*               | `8080`       |
| Utility     | Homepage            | Service dashboard and link aggregator    | `3000`                | `3000`       |
| Utility     | NextExplorer        | Web-based file manager for server files  | `8085`*               | `80`         |
| Utility     | Syncthing           | Continuous file sync across devices      | `8384`, `22000`       | `8384`       |
| Utility     | Firefox             | Containerized browser for remote use     | `3002`*               | `3000`       |
| Utility     | Speedtest Tracker   | Automated ISP speed monitoring           | `8084`*               | `80`         |
| Gaming      | Crafty Controller   | Minecraft server management              | `8443`, `25565`       | `8443`       |
| Gaming      | Redroid             | Android-in-a-container emulator          | `5555`                | `5555`       |

---

## Deployment Guide

### Prerequisites

```bash
# Ensure Docker and Docker Compose plugin are installed
sudo pacman -S docker docker-compose
sudo systemctl enable --now docker

# Add your user to the docker group (log out and back in after)
sudo usermod -aG docker $USER

# Create the root directory structure
mkdir -p ~/docker
```

> [!NOTE]
> **Portainer** is used for day-to-day container management and monitoring. Install it first if you don't already have it running — see [Portainer CE docs](https://docs.portainer.io/start/install-ce).

---

### 1. Core Infrastructure

<details>
<summary>Nginx Proxy Manager</summary>

**Purpose:** Reverse proxy with a web GUI for SSL certificates and domain routing.

```yaml
# ~/docker/npm/docker-compose.yml
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: unless-stopped
    ports:
      - "80:80"
      - "81:81"       # Admin panel
      - "443:443"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

**First run:** Access `http://192.168.0.x:81` — default credentials are `admin@example.com` / `changeme`.

</details>

<details>
<summary>Pi-hole</summary>

**Purpose:** DNS-level ad and tracker blocking for the entire network.

```yaml
# ~/docker/pihole/docker-compose.yml
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "8053:80"     # Web UI shifted from :80 to avoid NPM conflict
    environment:
      - TZ=${TZ:-Asia/Dhaka}
      - WEBPASSWORD=${PIHOLE_PASSWORD}
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
```

> [!TIP]
> Set your router's DHCP DNS to this server's LAN IP so all devices benefit automatically.

</details>

<details>
<summary>Tailscale</summary>

**Purpose:** WireGuard-based mesh VPN for secure remote access to all services.

```yaml
# ~/docker/tailscale/docker-compose.yml
services:
  tailscale:
    image: tailscale/tailscale:latest
    container_name: tailscale
    network_mode: host
    restart: unless-stopped
    cap_add:
      - net_admin
      - sys_module
    volumes:
      - ./state:/var/lib/tailscale
      - /dev/net/tun:/dev/net/tun
    environment:
      - TS_AUTHKEY=${TAILSCALE_AUTH_KEY}
      - TS_EXTRA_ARGS=--advertise-exit-node
```

</details>

<details>
<summary>Cloudflare Tunnel</summary>

**Purpose:** Expose select services (e.g., Jellyfin) through Cloudflare's proxy without opening router ports.

```yaml
# ~/docker/cloudflared/docker-compose.yml
services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    restart: unless-stopped
    command: tunnel run
    environment:
      - TUNNEL_TOKEN=${CF_TUNNEL_TOKEN}
```

> [!WARNING]
> Only expose services you intend to be publicly accessible. Configure allowed hostnames in the Cloudflare Zero Trust dashboard.

</details>

---

### 2. AI and Knowledge Suite

<details>
<summary>Ollama + Open WebUI</summary>

**Purpose:** Run LLMs locally on CPU and interact with them through a polished chat UI.

```yaml
# ~/docker/ollama/docker-compose.yml
services:
  ollama:
    image: ollama/ollama:latest
    container_name: ollama
    restart: unless-stopped
    ports:
      - "11434:11434"
    volumes:
      - ./data:/root/.ollama

  open-webui:
    image: ghcr.io/open-webui/open-webui:main
    container_name: open-webui
    restart: unless-stopped
    ports:
      - "3001:8080"   # Shifted from :8080 to avoid conflicts
    environment:
      - OLLAMA_BASE_URL=http://ollama:11434
      - WEBUI_SECRET_KEY=${WEBUI_SECRET}
    volumes:
      - ./open-webui-data:/app/backend/data
    depends_on:
      - ollama
```

> [!TIP]
> For stable CPU-only inference on a Ryzen 3600, prefer quantized models: `phi4-mini`, `llama3.2:3b`, or `gemma:2b`. Larger models (7B+) will run but may degrade other services.

</details>

<details>
<summary>Obsidian LiveSync (CouchDB)</summary>

**Purpose:** Real-time note synchronization across all Obsidian clients.

```yaml
# ~/docker/obsidian-livesync/docker-compose.yml
services:
  obsidian-livesync:
    image: couchdb:3
    container_name: obsidian-livesync
    restart: unless-stopped
    ports:
      - "5984:5984"
    environment:
      - COUCHDB_USER=${COUCHDB_USER}
      - COUCHDB_PASSWORD=${COUCHDB_PASSWORD}
    volumes:
      - ./data:/opt/couchdb/data
      - ./config:/opt/couchdb/etc/local.d
```

</details>

<details>
<summary>SearXNG</summary>

**Purpose:** Self-hosted, privacy-respecting metasearch engine aggregating results from multiple providers.

```yaml
# ~/docker/searxng/docker-compose.yml
services:
  searxng:
    image: searxng/searxng:latest
    container_name: searxng
    restart: unless-stopped
    ports:
      - "8081:8080"   # Shifted from :8080
    volumes:
      - ./settings:/etc/searxng
    environment:
      - SEARXNG_BASE_URL=http://192.168.0.x:8081/
```

</details>

---

### 3. Media and Downloads

<details>
<summary>Jellyfin</summary>

**Purpose:** Media server for movies, TV shows, and music with CPU-based transcoding.

```yaml
# ~/docker/jellyfin/docker-compose.yml
services:
  jellyfin:
    image: lscr.io/linuxserver/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    ports:
      - "8096:8096"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=${TZ:-Asia/Dhaka}
    volumes:
      - ./config:/config
      - ./media/movies:/data/movies
      - ./media/tvshows:/data/tvshows
      - ./media/music:/data/music
```

</details>

<details>
<summary>qBittorrent + QUI</summary>

**Purpose:** Torrent client for Linux ISOs™ with a modern alternative web interface.

```yaml
# ~/docker/qbittorrent/docker-compose.yml
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: unless-stopped
    ports:
      - "8082:8080"   # Web UI shifted from :8080
      - "6881:6881/tcp"
      - "6881:6881/udp"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=${TZ:-Asia/Dhaka}
    volumes:
      - ./config:/config
      - ./downloads:/downloads

  qui:
    image: nginx:alpine
    container_name: qui-frontend
    restart: unless-stopped
    ports:
      - "8083:80"
    volumes:
      - ./qui/html:/usr/share/nginx/html:ro
```

</details>

---

### 4. Utility Services

<details>
<summary>Homepage</summary>

**Purpose:** Unified dashboard to aggregate and organize links to all services.

```yaml
# ~/docker/homepage/docker-compose.yml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    restart: unless-stopped
    ports:
      - "3000:3000"
    volumes:
      - ./config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock:ro  # Docker integration
```

</details>

<details>
<summary>NextExplorer</summary>

**Purpose:** Modern web-based file browser for managing server files remotely.

```yaml
# ~/docker/nextexplorer/docker-compose.yml
services:
  nextexplorer:
    image: filebrowser/filebrowser:latest
    container_name: nextexplorer
    restart: unless-stopped
    ports:
      - "8085:80"     # Shifted from :80
    volumes:
      - /home/${USER}:/srv       # Root browsing path
      - ./database.db:/database.db
      - ./config.json:/.filebrowser.json
```

</details>

<details>
<summary>Syncthing</summary>

**Purpose:** Continuous file synchronization across devices, primarily for config files.

```yaml
# ~/docker/syncthing/docker-compose.yml
services:
  syncthing:
    image: lscr.io/linuxserver/syncthing:latest
    container_name: syncthing
    restart: unless-stopped
    ports:
      - "8384:8384"   # Web UI
      - "22000:22000/tcp"
      - "22000:22000/udp"
      - "21027:21027/udp"
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=${TZ:-Asia/Dhaka}
    volumes:
      - ./config:/config
      - ./data:/data/sync
```

</details>

<details>
<summary>Firefox</summary>

**Purpose:** Standalone containerized browser accessible via web — useful when mobile browsers misbehave.

```yaml
# ~/docker/firefox/docker-compose.yml
services:
  firefox:
    image: lscr.io/linuxserver/firefox:latest
    container_name: firefox
    restart: unless-stopped
    ports:
      - "3002:3000"   # Shifted from :3000 to avoid Homepage conflict
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=${TZ:-Asia/Dhaka}
    volumes:
      - ./config:/config
    shm_size: "1gb"
```

</details>

<details>
<summary>Speedtest Tracker</summary>

**Purpose:** Automated ISP speed tests with historical graphs to catch degradation.

```yaml
# ~/docker/speedtest-tracker/docker-compose.yml
services:
  speedtest-tracker:
    image: lscr.io/linuxserver/speedtest-tracker:latest
    container_name: speedtest-tracker
    restart: unless-stopped
    ports:
      - "8084:80"     # Shifted from :80
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=${TZ:-Asia/Dhaka}
      - DB_CONNECTION=sqlite
    volumes:
      - ./config:/config
```

</details>

---

### 5. Gaming and Emulation

<details>
<summary>Crafty Controller</summary>

**Purpose:** Web-based management panel for Minecraft server instances.

```yaml
# ~/docker/crafty/docker-compose.yml
services:
  crafty:
    image: registry.gitlab.com/crafty-controller/crafty-4:latest
    container_name: crafty
    restart: unless-stopped
    ports:
      - "8443:8443"   # Admin panel (HTTPS)
      - "25565:25565"  # Minecraft default
    volumes:
      - ./backups:/crafty/backups
      - ./logs:/crafty/logs
      - ./servers:/crafty/servers
      - ./config:/crafty/app/config
      - ./import:/crafty/import
```

</details>

<details>
<summary>Redroid</summary>

**Purpose:** Android-in-a-container for running Android apps without a physical device.

```yaml
# ~/docker/redroid/docker-compose.yml
services:
  redroid:
    image: redroid/redroid:latest
    container_name: redroid
    restart: unless-stopped
    privileged: true
    ports:
      - "5555:5555"
    volumes:
      - ./data:/data
```

> [!CAUTION]
> Redroid on Arch requires kernel modules `binder_linux` and `ashmem_linux`. Install `linux-zen` or the appropriate AUR modules, then load them:
> ```bash
> sudo modprobe binder_linux
> sudo modprobe ashmem_linux
> ```

</details>

---

## Routing and Access

### Local DNS (via Nginx Proxy Manager)

Map services to friendly hostnames through NPM proxy host entries:

| Domain              | Target                   | SSL  |
| ------------------- | ------------------------ | ---- |
| `jellyfin.local`    | `192.168.0.x:8096`       | ✗    |
| `search.local`      | `192.168.0.x:8081`       | ✗    |
| `ai.local`          | `192.168.0.x:3001`       | ✗    |
| `home.local`        | `192.168.0.x:3000`       | ✗    |
| `files.local`       | `192.168.0.x:8085`       | ✗    |
| `sync.local`        | `192.168.0.x:8384`       | ✗    |

### Remote Access

| Method             | Scope                                         |
| ------------------ | --------------------------------------------- |
| **Tailscale**      | Full access to all services via mesh VPN       |
| **Cloudflare Tunnel** | Selective public access (e.g., Jellyfin only) |

---

## Backup Strategy

### What to Back Up

| Priority | Data                         | Path                         |
| -------- | ---------------------------- | ---------------------------- |
| Critical | Compose files and `.env`     | `~/docker/*/docker-compose.yml`, `~/docker/*/.env` |
| Critical | Pi-hole config               | `~/docker/pihole/etc-pihole/` |
| High     | Obsidian sync database       | `~/docker/obsidian-livesync/data/` |
| High     | Jellyfin metadata            | `~/docker/jellyfin/config/`  |
| Medium   | Homepage config              | `~/docker/homepage/config/`  |
| Medium   | NPM proxy host definitions   | `~/docker/npm/data/`         |

### Quick Backup Script

```bash
#!/bin/bash
# ~/docker/backup.sh
BACKUP_DIR="/path/to/backup/$(date +%Y-%m-%d)"
mkdir -p "$BACKUP_DIR"

# Back up all compose files and env vars
find ~/docker -name "docker-compose.yml" -o -name ".env" | \
  tar czf "$BACKUP_DIR/compose-configs.tar.gz" -T -

# Back up critical service data
for svc in pihole obsidian-livesync jellyfin/config homepage/config npm/data; do
  tar czf "$BACKUP_DIR/$(echo $svc | tr '/' '-').tar.gz" -C ~/docker "$svc"
done

echo "Backup complete: $BACKUP_DIR"
```

---

## Maintenance

### Common Commands

```bash
# Pull latest images and recreate all stacks
cd ~/docker && for d in */; do (cd "$d" && docker compose pull && docker compose up -d); done

# View logs for a specific service
docker logs -f --tail 100 <container_name>

# Prune unused images and volumes
docker system prune -af --volumes
```

### Update Workflow (via Portainer)

1. Open **Portainer** → Stacks
2. Select the stack to update
3. Click **Pull and redeploy** → Confirm
4. Verify the container is healthy in the container list

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
| ------- | ------------ | --- |
| Service won't start | Port conflict with another container | Check the port map above; ensure no duplicates with `docker ps --format "{{.Names}}: {{.Ports}}"` |
| Container exits immediately | Missing or malformed `.env` | Verify `.env` exists in the service directory and all required variables are set |
| Permission denied on volumes | Ownership mismatch | Run `chown -R 1000:1000 ~/docker/<service>` |
| Pi-hole DNS not working | Port 53 already bound by `systemd-resolved` | Disable it: `sudo systemctl disable --now systemd-resolved` |
| Redroid fails to start | Missing kernel modules | Load `binder_linux` and `ashmem_linux`; consider switching to `linux-zen` |
| Ollama OOM or sluggish | Model too large for 16 GB RAM | Use smaller quantized models (≤ 3B parameters) |
| Can't access remotely | Tailscale not connected | Check `tailscale status`; re-authenticate if needed |

---
