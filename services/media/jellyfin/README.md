# Jellyfin (Open Source Media System)

Jellyfin is the community-driven, free-software media server alternative to Plex and Emby, enabling rich streaming across Smart TVs, mobile devices, and browsers without subscription paywalls.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `lscr.io/linuxserver/jellyfin:latest` |
| **Web Interface** | `http://<SERVER_IP>:8096` |
| **Transcoding** | CPU-based H.264/HEVC or hardware accelerated (VAAPI `/dev/dri`) |

---

## Hardware Transcoding (AMD / Intel GPU)

If your CPU supports QuickSync or your AMD Ryzen APU supports VAAPI:
1. Ensure the user is in the `video` and `render` groups:
   ```bash
   sudo usermod -aG video,render $USER
   ```
2. Verify `/dev/dri` exists on your system.
3. In Jellyfin: Navigate to **Dashboard** > **Playback** > **Transcoding** and select **VAAPI** or **AMD AMF**.

---

## Setup & Deployment

1. Copy environment template:
   ```bash
   cp .env.example .env
   ```
2. Adjust your media folder paths (`MOVIES_DIR`, `TV_DIR`, `MUSIC_DIR`).
3. Deploy:
   ```bash
   docker compose up -d
   ```
4. Access `http://<SERVER_IP>:8096` and follow the initial setup wizard to link your libraries.
