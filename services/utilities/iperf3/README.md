# iPerf3 (LAN Bandwidth Benchmark Server)

iPerf3 server for active measurements of maximum achievable bandwidth and network throughput across your home Wi-Fi and Ethernet LAN.

---

## Service Details

| Attribute | Details |
| :--- | :--- |
| **Image** | `networkstatic/iperf3:latest` |
| **Ports** | `5201/tcp`, `5201/udp` |

---

## Testing from Client Machines

From any client machine with `iperf3` installed:

```bash
# TCP Bandwidth test (10 seconds)
iperf3 -c <SERVER_IP>

# Reverse direction (Server -> Client)
iperf3 -c <SERVER_IP> -R

# Multi-stream test (4 parallel connections)
iperf3 -c <SERVER_IP> -P 4
```
