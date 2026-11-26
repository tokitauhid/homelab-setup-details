# TrueNAS Homelab: Modular Docker Architecture

![OS: TrueNAS CE 25.10.8](https://img.shields.io/badge/OS-TrueNAS%20CE%2025.10.8-0095D5.svg?logo=truenas&logoColor=white)
![Compute: AMD Ryzen 7 5800X](https://img.shields.io/badge/Compute-AMD%20Ryzen%207%205800X-ED1C24.svg?logo=amd&logoColor=white)
![Orchestration: Dockge + Docker Compose](https://img.shields.io/badge/Orchestration-Dockge%20%2B%20Docker%20Compose-2496ED.svg?logo=docker&logoColor=white)
![Security: Tailscale + Cloudflare](https://img.shields.io/badge/Security-Zero%20Trust%20Mesh-black.svg?logo=tailscale&logoColor=white)
![Status: Sanitized & Modular](https://img.shields.io/badge/Status-Sanitized%20%26%20Modular-green)

A modular, CPU-optimized, fully self-hosted home server running on **TrueNAS Community Edition (SCALE)** with ZFS storage pools and Dockge-managed Docker Compose stacks. Designed for clean service decoupling, zero-copy atomic hardlinks, automated 3-layer Vaultwarden + Google Drive backups, and zero-trust remote access via Tailscale and Cloudflare Tunnel.

---

## System Specifications

| Component | Specification | Purpose / Role |
| :--- | :--- | :--- |
| **Operating System** | TrueNAS Community Edition `25.10.8` (`Linux 6.12.113-production+truenas`) | ZFS storage appliance and Docker host |
| **CPU** | AMD Ryzen 7 5800X (8 Cores / 16 Threads) | High-performance compute, audio & media workloads |
| **RAM** | 16 GB DDR4 (15 GiB usable) | Shared across ZFS ARC & containerized services |
| **GPU** | NVIDIA GeForce GT 710 (`GK208B`) | Host display console |
| **Storage Pools (ZFS)** | `main_pool` (2.72 TB), `Jellyfin` (928 GB), `boot-pool` (236 GB) | `main_pool`: configs, Dockge stacks, user data; `Jellyfin`: media & downloads with atomic hardlinks; `boot-pool`: OS |
| **Container Management** | Docker Engine + Dockge (`:31014`) | Stack orchestration in `/mnt/main_pool/Docker_Configs/stacks/` |

---

## Network & Ingress Architecture

```
                    ┌───────────────── Internet ─────────────────┐
                    │                                             │
                    ▼                                             ▼
        ┌───────────────────────┐                     ┌───────────────────────┐
        │ Cloudflare Tunnel     │                     │ Tailscale Mesh VPN    │
        │ (Zero-Trust Ingress)  │                     │ (Subnet Router + Exit)│
        └───────────┬───────────┘                     └───────────┬───────────┘
                    │                                             │
                    └──────────────────────┬──────────────────────┘
                                           │
 ─────────────── LAN Subnet 192.168.0.0/24 ┼ ──────────────────────────────────────────
                                           │
   ┌───────────────────────────────────────┴───────────────────────────────────────┐
   │                                                                               │
   ├── [DNS & Security] ─────► AdGuard Home (:53, :8080) ── Vaultwarden (:8222)    │
   │                                                                               │
   ├── [Media & Audio] ──────► Jellyfin (:8096) ── qBittorrent (:8090, :6881)      │
   │                           QUI (:7476)                                         │
   │                           Audio Stack: AirPlay (:7000) ── DLNA ── Spotify     │
   │                                                                               │
   ├── [Arr Pipeline] ───────► Radarr (:7878) ── Sonarr (:8989) ── Bazarr (:6767)  │
   │                           Profilarr (:6868) ── Prowlarr (:9696)               │
   │                           FlareSolverr (:8191) ── Seerr (:5055)               │
   │                                                                               │
   ├── [Dashboards & Utils] ─► Dockge (:31014) ── Homepage (:3000)                 │
   │                           Firefox (:3001, :3002) ── QDirStat (:3010, :3011)   │
   │                           VS Code (:3020, :3021) ── Dozzle (:8888)            │
   │                           Watchtower (03:00 cron) ── Syncthing (:8384)        │
   │                           Speedtest (:8440, :8444) ── iPerf3 (:5201)          │
   │                                                                               │
   └── [Gaming] ─────────────► Crafty Controller (:8443, :8000, :25565-25570,      │
                               :19132/udp) ── Playit Tunnel                        │
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
├── readme.md                                     # Main homelab guide (this file)
│
└── services/
    ├── networking/
    │   ├── nginx-proxy-manager/                  # [Guide](./services/networking/nginx-proxy-manager/README.md)
    │   ├── pihole/                               # [Guide](./services/networking/pihole/README.md)
    │   ├── adguard-home/                         # [Guide](./services/networking/adguard-home/README.md)
    │   ├── tailscale/                            # [Guide](./services/networking/tailscale/README.md)
    │   └── cloudflare-tunnel/                    # [Guide](./services/networking/cloudflare-tunnel/README.md)
    │
    ├── media/
    │   ├── audio-stack/                          # [Guide](./services/media/audio-stack/README.md)
    │   ├── jellyfin/                             # [Guide](./services/media/jellyfin/README.md)
    │   ├── qbittorrent/                          # [Guide](./services/media/qbittorrent/README.md)
    │   ├── qui/                                  # [Guide](./services/media/qui/README.md)
    │   └── arr-stack/                            # [Guide](./services/media/arr-stack/README.md)
    │
    ├── ai-knowledge/
    │   ├── searxng/                              # [Guide](./services/ai-knowledge/searxng/README.md)
    │   ├── ollama-webui/                         # [Guide](./services/ai-knowledge/ollama-webui/README.md)
    │   └── obsidian-livesync/                    # [Guide](./services/ai-knowledge/obsidian-livesync/README.md)
    │
    ├── utilities/
    │   ├── homepage/                             # [Guide](./services/utilities/homepage/README.md)
    │   ├── homarr/                               # [Guide](./services/utilities/homarr/README.md)
    │   ├── nextexplorer/                         # [Guide](./services/utilities/nextexplorer/README.md)
    │   ├── syncthing/                            # [Guide](./services/utilities/syncthing/README.md)
    │   ├── firefox/                              # [Guide](./services/utilities/firefox/README.md)
    │   ├── vscode/                               # [Guide](./services/utilities/vscode/README.md)
    │   ├── qdirstat/                             # [Guide](./services/utilities/qdirstat/README.md)
    │   ├── dozzle/                               # [Guide](./services/utilities/dozzle/README.md)
    │   ├── watchtower/                           # [Guide](./services/utilities/watchtower/README.md)
    │   ├── speedtest-tracker/                    # [Guide](./services/utilities/speedtest-tracker/README.md)
    │   ├── vaultwarden/                          # [Guide](./services/utilities/vaultwarden/README.md)
    │   └── iperf3/                               # [Guide](./services/utilities/iperf3/README.md)
    │
    └── gaming/
        ├── crafty-minecraft/                     # [Guide](./services/gaming/crafty-minecraft/README.md)
        └── redroid/                              # [Guide](./services/gaming/redroid/README.md)
```

---

## Service & Port Map

All port mappings are conflict-free. Ports marked with `*` are customized for the live TrueNAS deployment.

### Active TrueNAS Production Services

| Category | Service | Description | Port(s) | Folder Guide |
| :--- | :--- | :--- | :--- | :--- |
| **Networking** | **AdGuard Home** | Network-wide ad blocker and local DNS server | `53`, `8080`* | [View Guide](./services/networking/adguard-home/README.md) |
| **Networking** | **Tailscale** | Mesh VPN node, subnet router & exit node | `host` | [View Guide](./services/networking/tailscale/README.md) |
| **Networking** | **Cloudflare Tunnel** | Zero-trust public egress without router port-forwarding | `none` | [View Guide](./services/networking/cloudflare-tunnel/README.md) |
| **Media** | **Jellyfin** | Media server for movies, TV shows, and music | `8096` | [View Guide](./services/media/jellyfin/README.md) |
| **Media (Audio)** | **TrueNAS Audio Stack** | AirPlay (`7000`), DLNA Renderer & Spotify Connect sharing ALSA `dmix` | `7000`, `host` | [View Guide](./services/media/audio-stack/README.md) |
| **Media** | **qBittorrent** | Torrent client sharing `/mnt/Jellyfin` for hardlinks | `8090`*, `6881` | [View Guide](./services/media/qbittorrent/README.md) |
| **Media** | **QUI** | Autobrr reactive web frontend for qBittorrent | `7476` | [View Guide](./services/media/qui/README.md) |
| **Media (Arr)** | **Radarr** | Movie management and automated downloading | `7878` | [View Guide](./services/media/arr-stack/README.md) |
| **Media (Arr)** | **Sonarr** | TV series management and automated downloading | `8989` | [View Guide](./services/media/arr-stack/README.md) |
| **Media (Arr)** | **Bazarr** | Automated subtitle downloader for Radarr and Sonarr | `6767` | [View Guide](./services/media/arr-stack/README.md) |
| **Media (Arr)** | **Profilarr** | Automated custom format & quality profile sync | `6868` | [View Guide](./services/media/arr-stack/README.md) |
| **Media (Arr)** | **Prowlarr** | Universal indexer manager and tracker proxy | `9696` | [View Guide](./services/media/arr-stack/README.md) |
| **Media (Arr)** | **FlareSolverr** | Cloudflare challenge bypass proxy for indexers | `8191` | [View Guide](./services/media/arr-stack/README.md) |
| **Media (Arr)** | **Seerr** | Media request and discovery platform for Jellyfin | `5055` | [View Guide](./services/media/arr-stack/README.md) |
| **Utility** | **Dockge** | TrueNAS Compose stack manager (`ix-dockge`) | `31014` | — |
| **Utility** | **Homepage** | Unified dashboard with real-time Docker widgets | `3000` | [View Guide](./services/utilities/homepage/README.md) |
| **Utility** | **Firefox** | Sandboxed containerized browser over WebRTC | `3001`*, `3002`* | [View Guide](./services/utilities/firefox/README.md) |
| **Utility** | **QDirStat** | Graphical disk usage analyzer for `/mnt/Jellyfin` *(created / on-demand)* | `3010`*, `3011`* | [View Guide](./services/utilities/qdirstat/README.md) |
| **Utility** | **VS Code** | Full desktop VS Code over Selkies WebRTC with native Account Sync | `3020`*, `3021`* | [View Guide](./services/utilities/vscode/README.md) |
| **Utility** | **Dozzle** | Real-time web Docker container log viewer | `8888`* | [View Guide](./services/utilities/dozzle/README.md) |
| **Utility** | **Watchtower** | Automated daily Docker image updater (`03:00` cron) | `cron` | [View Guide](./services/utilities/watchtower/README.md) |
| **Utility** | **Syncthing** | Peer-to-peer file synchronization *(created / on-demand)* | `8384`, `22000`, `21027` | [View Guide](./services/utilities/syncthing/README.md) |
| **Utility** | **Speedtest Tracker** | Automated bandwidth & latency benchmarking with graphs | `8440`*, `8444`* | [View Guide](./services/utilities/speedtest-tracker/README.md) |
| **Utility** | **Vaultwarden** | Self-hosted Bitwarden server with automated nightly GDrive backup | `8222`* | [View Guide](./services/utilities/vaultwarden/README.md) |
| **Utility** | **iPerf3 Server** | LAN throughput and bandwidth testing | `5201` | [View Guide](./services/utilities/iperf3/README.md) |
| **Gaming** | **Crafty Controller** | Minecraft server management panel + Playit tunnel | `8443`, `8000`, `25565-25570`, `19132` | [View Guide](./services/gaming/crafty-minecraft/README.md) |

### Optional / Archived Stack Templates

Preserved under `services/` and `docker-compose.yml` as ready-to-deploy templates (not currently running on TrueNAS):

| Category | Service | Description | Template Port(s) | Folder Guide |
| :--- | :--- | :--- | :--- | :--- |
| **Networking** | **Nginx Proxy Manager** | Reverse proxy, SSL cert automation & local routing | `80`, `81`, `443` | [View Guide](./services/networking/nginx-proxy-manager/README.md) |
| **Networking** | **Pi-hole** | Alternative network-wide ad blocker and DNS resolver | `53`, `8053`* | [View Guide](./services/networking/pihole/README.md) |
| **Media (Arr)** | **Lidarr** | Music collection management | `8686` | [View Guide](./services/media/arr-stack/README.md) |
| **Media (Arr)** | **Ombi** | Alternative request portal and user management | `3579` | [View Guide](./services/media/arr-stack/README.md) |
| **AI / Data** | **Ollama** | Local LLM runtime API (CPU-optimized for Ryzen 7 5800X) | `11434` | [View Guide](./services/ai-knowledge/ollama-webui/README.md) |
| **AI / Data** | **Open WebUI** | ChatGPT-style interface with RAG & model management | `3001`* | [View Guide](./services/ai-knowledge/ollama-webui/README.md) |
| **AI / Data** | **SearXNG + Redis** | Private metasearch engine aggregating 70+ engines | `8081`* | [View Guide](./services/ai-knowledge/searxng/README.md) |
| **AI / Data** | **Obsidian LiveSync** | Self-hosted CouchDB synchronization for Obsidian | `5984` | [View Guide](./services/ai-knowledge/obsidian-livesync/README.md) |
| **Utility** | **Homarr** | Customizable drag-and-drop dashboard with icons | `7575` | [View Guide](./services/utilities/homarr/README.md) |
| **Utility** | **NextExplorer** | Web-based file browser and remote server manager | `8085`* | [View Guide](./services/utilities/nextexplorer/README.md) |
| **Gaming** | **Redroid** | Android-in-a-container emulation via ADB/Scrcpy | `5555` | [View Guide](./services/gaming/redroid/README.md) |

---

## Quickstart Guide

### 1. Prerequisites (TrueNAS Community Edition + Dockge)

All production stacks on TrueNAS are managed through **Dockge** (`http://<SERVER_IP>:31014`), which stores stack definitions in `/mnt/main_pool/Docker_Configs/stacks/<stack_name>/compose.yaml`:

- **Timezone:** `TZ=Asia/Dhaka`
- **Service Permissions (`apps`):** `PUID=568` and `PGID=568` (or `user: "568:568"` for non-`PUID` images)
- **User Workspace Permissions:** `PUID=3000` and `PGID=3000` (used by `vscode`)
- **Persistent Configs:** `/mnt/main_pool/Docker_Configs/<app_name>`

### 2. Deploying via Dockge (Recommended)

1. Open Dockge at `http://<SERVER_IP>:31014` and click **+ Compose**.
2. Paste a sanitized stack definition from `services/<category>/<service>/docker-compose.yml` (adjusting paths to `/mnt/main_pool/Docker_Configs/...` and `PUID`/`PGID` to `568` as needed).
3. Click **Deploy**.

### 3. Deploying via Docker Compose CLI (Standalone / Portable)

For deploying individual stacks or the full reference compose from a terminal:

```bash
# Option A: Individual stack
cd services/media/jellyfin
cp .env.example .env
docker compose up -d

# Option B: Unified master compose
cp .env.example .env
docker compose --env-file .env up -d
```

---

## Storage & Hardlink Architecture

To prevent duplicate file copying and enable instantaneous, zero-overhead **atomic hardlinks** when torrent downloads complete, all media containers share the `/mnt/Jellyfin` ZFS dataset mounted identically at `/media`:

```plaintext
/mnt/Jellyfin/   (mounted as /media in Jellyfin, qBittorrent, Radarr, Sonarr, Bazarr)
├── downloads/
├── movies/
├── tv/
└── music/
```

Standard permissions on `/mnt/Jellyfin`:
- **Owner / Group:** `apps:apps` (`568:568`)
- **Directories:** `2770` (`drwxrws---`, setgid inherits `apps` group)
- **Files:** `770` (`-rwxrwx---`)

---

## Backup & Disaster Recovery

- **ZFS Snapshots & Stack Definitions:** Persistent app configs and Dockge stack files reside on `main_pool` under `/mnt/main_pool/Docker_Configs/`.
- **Automated Vaultwarden 3-Layer Nightly Pipeline:** TrueNAS runs a nightly cron (`23:50`) executing SQLite `.backup` + `PRAGMA integrity_check;`, `bw` `encrypted_json` exports (14-day retention) to `/mnt/main_pool/user/Backups/vaultwarden/`, and live delta-sync to Bitwarden Cloud, followed by TrueNAS Google Drive Cloud Sync (`00:00`). See [Vaultwarden Guide](./services/utilities/vaultwarden/README.md) for full details.

---

## Troubleshooting & Maintenance

| Issue | Likely Cause | Recommended Resolution |
| :--- | :--- | :--- |
| **Port 80/443 Conflict on AdGuard Home** | TrueNAS Web UI binds host ports `80` and `443` | Map AdGuard Home Web UI to `8080:80` (already configured) |
| **Arr Stack Hardlink Failed** | Different volume mount points between Arr and qBittorrent | Ensure Jellyfin, qBittorrent, Radarr, Sonarr, and Bazarr all mount `/mnt/Jellyfin:/media` identically |
| **Permission Denied in Media / Config Volume** | Ownership does not match `apps` (`568:568`) | Run `chown -R apps:apps /mnt/Jellyfin/<path>` and `chmod 2770` (dirs) / `770` (files) |
| **Ollama Memory Pressure** | Large model (7B+) exceeding shared 16 GB RAM | Use quantized models (`llama3.2:3b`, `phi4-mini`, `gemma2:2b`) |
| **Redroid Won't Boot** | TrueNAS SCALE kernel lacks `binder_linux`/`ashmem_linux` modules | Run Redroid on a host/VM with Android binder kernel support |
| **Remote Access Unavailable** | Tailscale subnet route not approved | Open [Tailscale Admin](https://login.tailscale.com/admin/machines) > Device > Edit route settings > Enable `192.168.0.0/24` |
