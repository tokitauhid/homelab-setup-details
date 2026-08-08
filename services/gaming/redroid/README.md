# Redroid (Android-in-a-Container)

Redroid (Remote-Android) is a GPU-accelerated / software-emulated Android environment running inside Docker with native ADB support and Scrcpy compatibility.

---

## Arch Linux Host Kernel Modules

Redroid requires the Android `binder_linux` and `ashmem_linux` kernel drivers. On Arch Linux, install `linux-zen` kernel or the corresponding AUR DKMS modules:

```bash
# Load kernel modules
sudo modprobe binder_linux devices="binder,hwbinder,vndbinder"
sudo modprobe ashmem_linux
```

---

## Deployment & Connecting with Scrcpy

1. Start Redroid:
   ```bash
   docker compose up -d
   ```
2. Connect ADB from your workstation or laptop:
   ```bash
   adb connect <SERVER_IP>:5555
   ```
3. Mirror and control Android screen with `scrcpy`:
   ```bash
   scrcpy -s <SERVER_IP>:5555 --audio-codec=raw
   ```
