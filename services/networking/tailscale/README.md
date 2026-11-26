# Tailscale (Mesh VPN & Subnet Router)

Tailscale provides zero-config WireGuard mesh VPN connectivity, enabling seamless access to your homelab services from anywhere without exposing public ports.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `tailscale/tailscale:latest` |
| **Network Mode** | `host` |
| **Features** | Subnet routing (`192.168.0.0/24`), Exit Node |

---

## Host Prerequisites (Linux / Arch)

Ensure IP forwarding is enabled on the host so Tailscale can route LAN subnet traffic:

```bash
# Enable runtime IP forwarding
sudo sysctl -w net.ipv4.ip_forward=1
sudo sysctl -w net.ipv6.conf.all.forwarding=1

# Make permanent across reboots
echo 'net.ipv4.ip_forward = 1' | sudo tee /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

---

## Deployment

1. Generate an auth key in the [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys) (select **Reusable** and **Pre-approved** if desired).
2. Configure `.env`:
   ```bash
   cp .env.example .env
   ```
3. Start the service:
   ```bash
   docker compose up -d
   ```
4. In the Tailscale Admin Console:
   - Locate your device.
   - Under **Edit route settings**, approve the `192.168.0.0/24` subnet route and/or exit node option.
