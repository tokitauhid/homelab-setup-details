# TrueNAS Network Speaker Stack (PipeWire RTP + PulseAudio TCP + DLNA + Spotify Connect)

A multi-protocol audio stack sharing the TrueNAS host sound card (`ALC892 Analog`, ALSA card `Generic`) concurrently across all receivers via ALSA `dmix` (`shared_speakers` over `ipc: host`).

---

## Services & Roles

| Service | Image | Protocol / Client | Port(s) |
| :--- | :--- | :--- | :--- |
| **PipeWire Audio Receiver** (`truenas-pulse`) | `alpine:latest` (`pipewire`, `wireplumber`, `pipewire-pulse`) | PipeWire RTP (`rtp-sink` → `rtp-source`) + PulseAudio TCP for Linux PC & Laptop | `46000/udp`, `46002/udp`, `4713/tcp` |
| **DLNA Renderer** (`truenas-dlna`) | `giof71/gmrender-resurrect:latest` | UPnP / DLNA for Android (Symfonium, BubbleUPnP, Jellyfin) | `host` (`49494/tcp` / UPnP) |
| **Spotify Connect** (`truenas-spotify`) | `ghcr.io/devgianlu/go-librespot:latest` | Native Spotify Connect via host Avahi mDNS | `host` (Avahi Zeroconf) |

---

## Architecture Notes

- **Concurrent Audio Sharing (`asound.conf`):** All three containers share `/dev/snd` with `ipc: host` and `group_add: [29]` (`audio`). ALSA `dmix_generic` (`ipc_key 1024`, `ipc_perm 0666`, `ipc_gid 29`, `48000 Hz`) exposes `pcm.shared_speakers` and overrides `pcm.!default`, allowing simultaneous playback without `Device or resource busy`.
- **Ultra-Low-Latency Desktop Audio (`truenas-pulse`):** Uses PipeWire's native `libpipewire-module-rtp-source` (UDP `:46000` at ~35 ms for wired PC, `:46002` at ~57 ms for Wi-Fi laptop) locked directly to the TrueNAS ALC892 clock via PipeWire's adaptive DLL resampler (eliminating `module-pulse-tunnel` clock-drift underruns/crackling), plus `pipewire-pulse` on TCP `:4713`.
- **Host Avahi Integration:** `go-librespot` mounts `/var/run/dbus` and `/var/run/avahi-daemon` from the host so mDNS advertisements use the host Avahi daemon cleanly.
- **Boot Unmute:** Ensure the TrueNAS card `Master` mixer channel is unmuted at `100%` on boot (`amixer -c 1 sset Master 100% unmute`).

---

## Linux Client Setup (PipeWire)

Create `~/.config/pipewire/pipewire.conf.d/50-truenas-rtp.conf` on each Linux client (use `destination.port = 46000` on `fedora-pc` or `46002` on `omarchy` laptop):

```conf
context.modules = [
    { name = libpipewire-module-rtp-sink
      args = {
          destination.ip = "192.168.0.225"
          destination.port = 46000
          net.mtu = 1400
          sess.name = "TrueNAS-Speakers-PC"
          sess.media = "audio"
          sess.min-ptime = 2
          sess.max-ptime = 8
          audio.format = "S16BE"
          audio.rate = 48000
          audio.channels = 2
          audio.position = [ FL FR ]
          stream.props = {
              node.name = "rtp_sink.truenas"
              node.description = "TrueNAS-Speakers"
              media.class = "Audio/Sink"
              device.icon-name = "audio-speakers"
              sess.latency.msec = 7.083333
          }
      }
    }
]
```

Apply and set as default output:

```bash
systemctl --user restart pipewire pipewire-pulse wireplumber
pactl set-default-sink rtp_sink.truenas
```
