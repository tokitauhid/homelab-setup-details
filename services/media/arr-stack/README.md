# The Ultimate Arr Automation Stack

A synchronized, automated media acquisition and management pipeline comprising Radarr, Sonarr, Lidarr, Bazarr, Profilarr, Prowlarr, FlareSolverr, qBittorrent, Seerr, and Ombi on a shared bridge network with atomic hardlinking.

---

## Service & Port Matrix

| Service | Port | Description |
| :--- | :--- | :--- |
| **Radarr** | `7878` | Movie collection manager and automated downloader |
| **Sonarr** | `8989` | TV show collection manager and season watcher |
| **Lidarr** | `8686` | Music collection manager |
| **Prowlarr** | `9696` | Centralized indexer and tracker proxy manager |
| **FlareSolverr** | `8191` | Cloudflare captcha and challenge bypass proxy |
| **qBittorrent (Arr)** | `8091` *(WebUI)*, `6882` *(P2P)* | Dedicated torrent client for automated pipelines |
| **Seerr** | `5055` | Media discovery and request platform for Jellyfin |
| **Ombi** | `3579` | Alternative media request management portal |
| **Bazarr** | `6767` | Automated subtitle downloader for Radarr and Sonarr |
| **Profilarr** | `6868` | Automated custom format & quality profile sync for Radarr/Sonarr |

---

## Recommended Directory Structure (Trash-Guides Standard)

To allow instant, zero-copy **atomic hardlinks** between downloads and your library without duplicate disk space consumption, structure `DATA_ROOT` as:

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

All containers mount `/data` identically, enabling instant moving/linking across directories.

---

## Integration Workflow

1. **Prowlarr -> Indexers**:
   - Add your indexers and trackers in Prowlarr (`http://<SERVER_IP>:9696`).
   - Add FlareSolverr (`http://flaresolverr:8191`) as a proxy tag in Prowlarr for Cloudflare-protected trackers.
   - Under **Settings** > **Apps**, connect Radarr, Sonarr, and Lidarr using their respective API keys and container hostnames (`http://radarr:7878`, `http://sonarr:8989`, `http://lidarr:8686`).

2. **Download Client**:
   - In Radarr/Sonarr/Lidarr: Add **qBittorrent** using Host: `qbittorrent-arr` and Port: `8080` (internal) or host port `8091`.

3. **Seerr / Ombi -> Media Server**:
   - Connect Seerr (`http://<SERVER_IP>:5055`) to your **Jellyfin** instance and link Radarr/Sonarr for 1-click user requests.

---

## Deployment

```bash
cp .env.example .env
docker compose up -d
```
