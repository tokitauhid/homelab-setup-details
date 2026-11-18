# TrueNAS Network Speaker Stack (AirPlay + DLNA + Spotify Connect)

A multi-protocol audio stack sharing the TrueNAS host sound card (`ALC892 Analog`, ALSA card `Generic`) concurrently across all receivers via ALSA `dmix` (`shared_speakers` over `ipc: host`).

---

## Services & Roles

| Service | Image | Protocol / Client | Port(s) |
| :--- | :--- | :--- | :--- |
| **AirPlay Receiver** (`truenas-airplay`) | `mikebrady/shairport-sync:latest` | AirPlay (RAOP) for Linux PipeWire (`raop-discover`) & Apple devices | `7000/tcp`, UDP `6001-6003` (`host`) |
| **DLNA Renderer** (`truenas-dlna`) | `giof71/gmrender-resurrect:latest` | UPnP / DLNA for Android (Symfonium, BubbleUPnP, Jellyfin) | `host` (`49494/tcp` / UPnP) |
| **Spotify Connect** (`truenas-spotify`) | `ghcr.io/devgianlu/go-librespot:latest` | Native Spotify Connect via host Avahi mDNS | `host` (Avahi Zeroconf) |

---

## Architecture Notes

- **Concurrent Audio Sharing (`asound.conf`):** All three containers share `/dev/snd` with `ipc: host` and `group_add: [29]` (`audio`). ALSA `dmix_generic` (`ipc_key 1024`, `ipc_perm 0666`, `ipc_gid 29`, `44100 Hz`) exposes `pcm.shared_speakers` and overrides `pcm.!default`, allowing simultaneous playback without `Device or resource busy`.
- **Host Avahi Integration:** Both `shairport-sync` (`ENABLE_AVAHI=0`) and `go-librespot` mount `/var/run/dbus` and `/var/run/avahi-daemon` from the host so mDNS advertisements use the host Avahi daemon cleanly.
- **Boot Unmute:** Ensure the TrueNAS card `Master` mixer channel is unmuted at `100%` on boot (`amixer -c 1 sset Master 100% unmute`).

---

## Linux Client Setup (PipeWire AirPlay Discovery)

- **Fedora:** Ships `/usr/share/pipewire/pipewire.conf.d/50-raop.conf` (`libpipewire-module-raop-discover`) enabled by default with `avahi-daemon`.
- **Arch (`omarchy`):** Install `pipewire-zeroconf` + `avahi` and enable `avahi-daemon`.
- **Low-Latency Tuning (`~/.config/pipewire/pipewire.conf.d/50-raop.conf`):** Override `raop-discover` with `raop.latency.ms = 200` to cut AirPlay latency from `1750 ms` down to `~500 ms`:

```conf
context.properties = {
    module.raop = false
}

context.modules = [
    { name = libpipewire-module-raop-discover
      args = {
          stream.rules = [
              { matches = [ { raop.ip = "~.*" } ]
                actions = {
                    create-stream = {
                        raop.latency.ms = 200
                    }
                }
              }
          ]
      }
    }
]
```
