# TrueNAS Network Speaker Stack (PulseAudio TCP + DLNA + Spotify Connect)

A multi-protocol audio stack sharing the TrueNAS host sound card (`ALC892 Analog`, ALSA card `Generic`) concurrently across all receivers via ALSA `dmix` (`shared_speakers` over `ipc: host`).

---

## Services & Roles

| Service | Image | Protocol / Client | Port(s) |
| :--- | :--- | :--- | :--- |
| **PulseAudio TCP** (`truenas-pulse`) | `alpine:latest` (`pulseaudio`) | PipeWire `pulse-tunnel` for Linux Desktop & Laptop | `4713/tcp` (host) |
| **DLNA Renderer** (`truenas-dlna`) | `giof71/gmrender-resurrect:latest` | UPnP / DLNA for Android (Symfonium, BubbleUPnP, Jellyfin) | `host` (`49494/tcp` / UPnP) |
| **Spotify Connect** (`truenas-spotify`) | `ghcr.io/devgianlu/go-librespot:latest` | Native Spotify Connect via host Avahi mDNS | `host` (Avahi Zeroconf) |

---

## Architecture Notes

- **Concurrent Audio Sharing (`asound.conf`):** All three containers share `/dev/snd` with `ipc: host` and `group_add: [29]` (`audio`). ALSA `dmix_generic` (`ipc_key 1024`, `ipc_perm 0666`, `ipc_gid 29`) exposes `pcm.shared_speakers` and overrides `pcm.!default`, allowing simultaneous playback without `Device or resource busy`.
- **Low-Latency Desktop Audio (`truenas-pulse`):** Uses `module-alsa-sink device=shared_speakers tsched=0 fragments=2 fragment_size=4096` (~46 ms sink latency) and `module-native-protocol-tcp port=4713 auth-ip-acl=127.0.0.1;192.168.0.0/24 auth-anonymous=1 auth-cookie-enabled=0`.
- **Host Avahi Integration:** `go-librespot` mounts `/var/run/dbus` and `/var/run/avahi-daemon` from the host so mDNS advertisements use the host Avahi daemon cleanly.
- **Boot Unmute:** Ensure the TrueNAS card `Master` mixer channel is unmuted at `100%` on boot (`amixer -c 1 sset Master 100% unmute`).

---

## Linux Client Setup (PipeWire)

Create `~/.config/pipewire/pipewire.conf.d/50-truenas-pulse.conf` on each Linux client (use `pulse.latency = 40` on wired Ethernet or `80` on Wi-Fi):

```conf
context.modules = [
    { name = libpipewire-module-pulse-tunnel
      args = {
          pulse.server.address = "tcp:192.168.0.225:4713"
          pulse.latency = 40
          reconnect.interval.ms = 3000
          tunnel.mode = "sink"
          node.name = "pulse_tunnel.truenas"
          node.description = "TrueNAS-Speakers"
          stream.props = {
              device.icon-name = "audio-speakers"
          }
      }
    }
]
```

Apply and set as default output:

```bash
systemctl --user restart pipewire pipewire-pulse wireplumber
pactl set-default-sink pulse_tunnel.truenas
```
