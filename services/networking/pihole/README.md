# Pi-hole (DNS Sinkhole & Ad Blocker)

Pi-hole provides network-wide ad, tracker, and malicious domain blocking by acting as your LAN's primary DNS resolver.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `pihole/pihole:latest` |
| **Web Interface** | `http://<SERVER_IP>:8053/admin` |
| **DNS Port** | `53/tcp`, `53/udp` |
| **DHCP Port** | `67/udp` (optional) |

---

## Directory Structure

```plaintext
pihole/
├── docker-compose.yml
├── .env.example
├── README.md
├── etc-pihole/       # Blocklists, query database, and FTL configurations
└── etc-dnsmasq.d/    # Custom DNS static records and DHCP lease parameters
```

---

## Port 53 Host Conflict Resolution (Arch Linux)

On Arch Linux and systemd-based distributions, `systemd-resolved` frequently binds port 53 by default. To free port 53 for Pi-hole:

1. **Disable DNS stub listener**:
   Edit `/etc/systemd/resolved.conf`:
   ```ini
   [Resolve]
   DNSStubListener=no
   ```

2. **Restart systemd-resolved**:
   ```bash
   sudo systemctl restart systemd-resolved
   ```

---

## Deployment & Setup

1. **Copy and adjust environment configuration**:
   ```bash
   cp .env.example .env
   ```

2. **Start the container**:
   ```bash
   docker compose up -d
   ```

3. **Configure LAN Devices**:
   - Update your home router's DHCP DNS settings to point primary DNS to your server IP (e.g. `192.168.0.225`).
   - Access the admin panel at `http://<SERVER_IP>:8053/admin`.
