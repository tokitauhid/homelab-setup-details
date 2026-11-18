# Arch Homelab: Modular Docker Architecture

![OS: Arch Linux](https://img.shields.io/badge/OS-Arch%20Linux-1793d1.svg?logo=arch-linux&logoColor=white)
![Compute: AMD Ryzen 3600](https://img.shields.io/badge/Compute-AMD%20Ryzen%205%203600-ED1C24.svg?logo=amd&logoColor=white)
![Architecture: Modular Docker](https://img.shields.io/badge/Architecture-Modular%20Docker%20Compose-2496ED.svg?logo=docker&logoColor=white)
![Security: Tailscale + Cloudflare](https://img.shields.io/badge/Security-Zero%20Trust%20Mesh-black.svg?logo=tailscale&logoColor=white)
![Status: Production Ready](https://img.shields.io/badge/Status-Sanitized%20%26%20Modular-green)

A modular, CPU-optimized, fully self-hosted home server architecture built on vanilla Arch Linux. Designed for clean service decoupling, zero-copy atomic hardlinks, predictable backup workflows, and secure remote access via Tailscale and Cloudflare Zero Trust Tunnels.

---

## System Specifications

| Component | Specification | Purpose / Role |
| :--- | :--- | :--- |
| **Operating System** | Vanilla Arch Linux (Rolling Kernel) | Base OS host with minimal overhead |
| **CPU** | AMD Ryzen 5 3600 (6 Cores / 12 Threads) | High-performance CPU compute & transcode |
| **RAM** | 16 GB DDR4 3200MHz | Shared across containers & local LLM cache |
| **GPU** | NVIDIA GT 710 | Host display console |
| **Storage Architecture** | Atomic Hardlink Hierarchy (`/data`) | Instant, zero-copy torrent to media promotion |
| **Container Engine** | Docker Engine + Docker Compose v2 | Modular container execution |

---

## Network & Ingress Architecture

```
                    ┌───────────────── Internet ─────────────────┐
                    │                                             │
                    ▼                                             ▼
        ┌───────────────────────┐                     ┌───────────────────────┐
        │ Cloudflare Tunnel     │                     │ Tailscale Mesh VPN    │
        │ (Zero-Trust Ingress)  │                     │ (Subnet Router)       │
        └───────────┬───────────┘                     └───────────┬───────────┘
                    │                                             │
                    └──────────────────────┬──────────────────────┘
                                           │
                                           ▼
                    ┌─────────────────────────────────────────────┐
                    │      Nginx Proxy Manager (:80, :443, :81)    │
                    └──────────────────────┬──────────────────────┘
                                           │
 ─────────────── LAN Subnet 192.168.0.0/24 ┼ ──────────────────────────────────────────
                                           │
   ┌───────────────────────────────────────┴───────────────────────────────────────┐
   │                                                                               │
   ├── [DNS & Security] ─────► Pi-hole (:53, :8053) [Local DNS Sinkhole]          │
   │                                                                               │
   ├── [Media Suite] ────────► Jellyfin (:8096) ── qBittorrent (:8082) ── QUI (:8083)
   │                           Audio Stack: AirPlay (:7000) ── DLNA ── Spotify Connect
   │                                                                               │
   ├── [Arr Pipeline] ───────► Radarr (:7878) ── Sonarr (:8989) ── Lidarr (:8686) │
   │                           Prowlarr (:9696) ── FlareSolverr (:8191)            │
   │                           qBit-Arr (:8091) ── Seerr (:5055) ── Ombi (:3579)   │
   │                                                                               │
   ├── [AI & Knowledge] ─────► Ollama (:11434) ── Open WebUI (:3001)               │
   │                           SearXNG (:8081) ── Obsidian LiveSync (:5984)        │
   │                                                                               │
   ├── [Dashboards & Utils] ─► Homepage (:3000) ── Homarr (:7575)                  │
   │                           NextExplorer (:8085) ── Syncthing (:8384)           │
   │                           Firefox Web (:3002) ── Speedtest (:8084)            │
   │                           iPerf3 (:5201)                                      │
   │                                                                               │
   └── [Gaming] ─────────────► Crafty Controller (:8443) ── Redroid (:5555)        │
                                                                                   │
 ──────────────────────────────────────────────────────────────────────────────────
```

---

## Directory Organization

The homelab is decomposed into categorized, self-contained service stacks under `services/`. Each directory contains its own `docker-compose.yml`, `.env.example`, and dedicated documentation:

```plaintext
homelab-setup-details/
├── .env.example                                  # Master environment configuration template
├── .gitignore                                    # Secret and runtime file exclusions
├── docker-compose.yml                            # Unified master compose specification
├── README.md                                     # Main homelab guide (this file)
│
└── services/
    ├── networking/
    │   ├── nginx-proxy-manager/                  # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/nginx-proxy-manager/README.md)
    │   ├── pihole/                               # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/pihole/README.md)
    │   ├── tailscale/                            # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/tailscale/README.md)
    │   └── cloudflare-tunnel/                    # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/cloudflare-tunnel/README.md)
    │
    ├── media/
    │   ├── audio-stack/                          # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/audio-stack/README.md)
    │   ├── jellyfin/                             # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/jellyfin/README.md)
    │   ├── qbittorrent/                          # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/qbittorrent/README.md)
    │   ├── qui/                                  # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/qui/README.md)
    │   └── arr-stack/                            # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md)
    │
    ├── ai-knowledge/
    │   ├── searxng/                              # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/ai-knowledge/searxng/README.md)
    │   ├── ollama-webui/                         # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/ai-knowledge/ollama-webui/README.md)
    │   └── obsidian-livesync/                    # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/ai-knowledge/obsidian-livesync/README.md)
    │
    ├── utilities/
    │   ├── homepage/                             # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/homepage/README.md)
    │   ├── homarr/                               # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/homarr/README.md)
    │   ├── nextexplorer/                         # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/nextexplorer/README.md)
    │   ├── syncthing/                            # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/syncthing/README.md)
    │   ├── firefox/                              # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/firefox/README.md)
    │   ├── speedtest-tracker/                    # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/speedtest-tracker/README.md)
    │   └── iperf3/                               # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/iperf3/README.md)
    │
    └── gaming/
        ├── crafty-minecraft/                     # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/gaming/crafty-minecraft/README.md)
        └── redroid/                              # [Guide](file:///home/tokit/Projects/homelab-setup-details/services/gaming/redroid/README.md)
```

---

## Service & Port Map

All port mappings are conflict-free. Ports marked with `*` have been customized from upstream defaults:

| Category | Service | Description | Port(s) | Folder Guide |
| :--- | :--- | :--- | :--- | :--- |
| **Networking** | **Nginx Proxy Manager** | Reverse proxy, SSL cert automation & local routing | `80`, `81`, `443` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/nginx-proxy-manager/README.md) |
| **Networking** | **Pi-hole** | Network-wide ad blocking and local DNS resolver | `53`, `8053`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/pihole/README.md) |
| **Networking** | **Tailscale** | Mesh VPN node & subnet router | `host` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/tailscale/README.md) |
| **Networking** | **Cloudflare Tunnel** | Zero-trust public egress without router port-forwarding | `none` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/networking/cloudflare-tunnel/README.md) |
| **Media** | **Jellyfin** | High-performance media server (movies, TV, music) | `8096`, `8920` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/jellyfin/README.md) |
| **Media (Audio)** | **TrueNAS Audio Stack** | AirPlay (`7000`), DLNA Renderer, & Spotify Connect sharing ALSA `dmix` | `7000`, `host` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/audio-stack/README.md) |
| **Media** | **qBittorrent (Standalone)** | Standalone download client with ThemePark Catppuccin | `8082`*, `6881` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/qbittorrent/README.md) |
| **Media** | **QUI** | Autobrr reactive web frontend for qBittorrent | `8083`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/qui/README.md) |
| **Media (Arr)** | **Radarr** | Movie management and automated downloading | `7878` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **Media (Arr)** | **Sonarr** | TV series management and automated downloading | `8989` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **Media (Arr)** | **Lidarr** | Music collection management | `8686` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **Media (Arr)** | **Prowlarr** | Universal indexer manager and tracker proxy | `9696` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **Media (Arr)** | **FlareSolverr** | Cloudflare challenge bypass proxy for indexers | `8191` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **Media (Arr)** | **qBittorrent (Arr)** | Dedicated download client for automated media pipeline | `8091`*, `6882`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **Media (Arr)** | **Seerr** | Media request and discovery platform for Jellyfin | `5055` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **Media (Arr)** | **Ombi** | Alternative request portal and user management | `3579` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/media/arr-stack/README.md) |
| **AI / Data** | **Ollama** | Local LLM runtime API (CPU-optimized for Ryzen 3600) | `11434` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/ai-knowledge/ollama-webui/README.md) |
| **AI / Data** | **Open WebUI** | ChatGPT-style interface with RAG & model management | `3001`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/ai-knowledge/ollama-webui/README.md) |
| **AI / Data** | **SearXNG + Redis** | Private metasearch engine aggregating 70+ engines | `8081`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/ai-knowledge/searxng/README.md) |
| **AI / Data** | **Obsidian LiveSync** | Self-hosted CouchDB synchronization for Obsidian | `5984` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/ai-knowledge/obsidian-livesync/README.md) |
| **Utility** | **Homepage** | Unified dashboard with real-time docker widgets | `3000` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/homepage/README.md) |
| **Utility** | **Homarr** | Customizable drag-and-drop dashboard with icons | `7575` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/homarr/README.md) |
| **Utility** | **NextExplorer** | Web-based file browser and remote server manager | `8085`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/nextexplorer/README.md) |
| **Utility** | **Syncthing** | Continuous peer-to-peer file synchronization | `8384`, `22000` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/syncthing/README.md) |
| **Utility** | **Firefox** | Sandboxed containerized browser over WebRTC | `3002`*, `3003`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/firefox/README.md) |
| **Utility** | **Speedtest Tracker** | Automated bandwidth & latency benchmarking with graphs | `8084`*, `8444`* | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/speedtest-tracker/README.md) |
| **Utility** | **iPerf3 Server** | LAN throughput and bandwidth testing | `5201` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/utilities/iperf3/README.md) |
| **Gaming** | **Crafty Controller** | Minecraft server management panel + Playit tunnel | `8443`, `25565` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/gaming/crafty-minecraft/README.md) |
| **Gaming** | **Redroid** | Android-in-a-container emulation via ADB/Scrcpy | `5555` | [View Guide](file:///home/tokit/Projects/homelab-setup-details/services/gaming/redroid/README.md) |

---

## Quickstart Guide

### 1. Prerequisites (Arch Linux)

```bash
# Install Docker and Docker Compose v2 plugin
sudo pacman -S docker docker-compose

# Enable and start Docker service
sudo systemctl enable --now docker

# Add your user to the docker group (log out and back in to apply)
sudo usermod -aG docker $USER
```

### 2. Configure Environment

Copy the global configuration template and define your preferences:

```bash
cp .env.example .env
```

Review and configure:
- `TZ` (e.g. `Asia/Dhaka`, `UTC`, `America/New_York`)
- `PUID` and `PGID` (run `id` in your terminal, usually `1000`)
- `DOCKER_ROOT` (base path for persistent container data)
- Auth keys for Tailscale and Cloudflare Tunnel

### 3. Deploying Stacks

#### Option A: Deploying Individual Stacks (Recommended)
Navigate to any service directory under `services/` and deploy:

```bash
# Example: Deploying Jellyfin
cd services/media/jellyfin
cp .env.example .env
docker compose up -d

# Example: Deploying the Arr Automation Stack
cd ../arr-stack
cp .env.example .env
docker compose up -d
```

#### Option B: Deploying the Unified Master Compose
To spin up all services simultaneously:

```bash
docker compose --env-file .env up -d
```

---

## Storage & Hardlink Architecture (Trash-Guides Standard)

To prevent duplicate file copying and enable instantaneous, zero-overhead hardlinks when torrent downloads complete, maintain a unified volume layout:

```plaintext
/data/
├── torrents/
│   ├── movies/
│   ├── tv/
│   └── music/
└── media/
    ├── movies/
    ├── tvshows/
    └── music/
```

Mount `/data` identically in `radarr`, `sonarr`, `lidarr`, and `qbittorrent-arr` so the Linux kernel can link files instantly across directories without rewriting disk sectors.

---

## Backup & Disaster Recovery

### Automated Backup Script

Save as `backup.sh` to create timestamped archives of all Compose definitions, configurations, and databases:

```bash
#!/usr/bin/env bash
set -euo pipefail

BACKUP_DEST="/path/to/external_backup/$(date +%Y-%m-%d)"
mkdir -p "$BACKUP_DEST"

echo "=== Backing up Compose Definitions & Environments ==="
tar --exclude='.git' -czf "$BACKUP_DEST/compose_configs.tar.gz" .

echo "=== Backing up Critical Application Data ==="
# Adjust paths to match your DOCKER_ROOT directory
for svc in pihole/etc-pihole obsidian-livesync/data jellyfin/config homepage/config npm/data; do
    if [ -d "$HOME/docker/$svc" ]; then
        tar -czf "$BACKUP_DEST/$(echo "$svc" | tr '/' '-').tar.gz" -C "$HOME/docker" "$svc"
    fi
done

echo "Backup complete! Stored in: $BACKUP_DEST"
```

---

## Troubleshooting & Maintenance

| Issue | Likely Cause | Recommended Resolution |
| :--- | :--- | :--- |
| **Port 53 Bind Failure** | `systemd-resolved` is listening on port 53 | Set `DNSStubListener=no` in `/etc/systemd/resolved.conf` and restart the service |
| **Arr Stack Hardlink Failed** | Different volume mount points between Arr and qBittorrent | Ensure both containers share the unified `/data` root mount |
| **Ollama Memory Pressure** | Large model (7B+) exceeding available 16 GB RAM | Use quantized models (`llama3.2:3b`, `phi4-mini`, `gemma2:2b`) |
| **Redroid Won't Boot** | Host missing Android kernel modules | Load modules: `sudo modprobe binder_linux ashmem_linux` (switch to `linux-zen` kernel if needed) |
| **Remote Access Unavailable** | Tailscale route not approved | Open [Tailscale Admin](https://login.tailscale.com/admin/machines) > Device > Edit route settings > Enable subnet |
| **Permission Denied in Volume** | Ownership does not match `PUID:PGID` | Set correct ownership: `sudo chown -R 1000:1000 ~/docker/<service>` |
