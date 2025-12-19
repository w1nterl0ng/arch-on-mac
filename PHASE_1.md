# Phase 1 — Arch Linux Minimal + Niri (TTY)

**Goal:** Build a clean, repeatable Arch Linux base that boots to TTY and launches Niri via `niri-session`, with working Wi‑Fi, GPU acceleration, and a minimal toolset suitable for older MacBook Pro hardware.

End state:
- Arch Linux installed (minimal)
- Broadcom Wi‑Fi working
- Wayland + GPU acceleration verified
- Niri launches from TTY
- `foot` terminal
- Firefox installed

---

## Hardware assumptions

- Older MacBook Pro (≈2011–2014)
- Broadcom BCM4331 Wi‑Fi
- Intel CPU, NVIDIA Kepler dGPU (nouveau)
- UEFI boot
- Optional temporary internet via iPhone USB tether

---

## Design principles

- Minimal, explicit setup
- No display manager
- No automatic snapshot tooling
- Prefer understanding over abstraction

---

## Step 0 — Boot Arch ISO

1. Boot from Arch Linux ISO
2. Set keyboard layout if needed
3. Update keyring and tools:

```bash
pacman -Sy
pacman -Sy archlinux-keyring archinstall
```

Enable NTP:

```bash
timedatectl set-ntp true
```

Identify disks:

```bash
lsblk
```

---

## Step 0.1 — Wipe disk (example: /dev/sda)

⚠️ Verify disk name before proceeding.

```bash
gdisk /dev/sda
# x → z → y → y
```

---

## Step 1 — Run archinstall (Minimal)

```bash
archinstall
```

Recommended selections:
- Profile: **Minimal**
- Bootloader: **systemd-boot**
- Filesystem: **ext4**
- Kernel: **linux**
- Network: **NetworkManager**
- Audio: **PipeWire**
- Users: normal user, member of `wheel`

### Additional packages

Add explicitly:
- linux-headers
- broadcom-wl-dkms
- networkmanager
- kmod
- usbutils
- git
- nano

Install and reboot.

---

## Step 2 — First boot networking

Enable NetworkManager:

```bash
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
```

### Optional: iPhone USB tether

```bash
sudo modprobe ipheth
nmcli device
ping -c 3 archlinux.org
```

---

## Step 3 — Broadcom Wi‑Fi (BCM4331, wl)

Load driver:

```bash
sudo modprobe wl
```

Confirm driver:

```bash
lspci -k | grep -A3 -i network
```

Expected:
- Kernel driver in use: wl

Connect Wi‑Fi:

```bash
sudo rfkill unblock all
nmcli device wifi list
nmcli device wifi connect "SSID" password "PASSWORD"
```

Verify:

```bash
ping -c 3 archlinux.org
```

---

## Step 4 — GPU + Wayland baseline

Install graphics stack:

```bash
sudo pacman -S --needed mesa libglvnd wayland wayland-protocols xorg-xwayland vulkan-icd-loader
```

Install utilities:

```bash
sudo pacman -S --needed seatd foot wl-clipboard mesa-utils
```

Enable seatd:

```bash
sudo systemctl enable seatd
sudo systemctl start seatd
sudo usermod -aG seat $(whoami)
```

Log out/in or reboot.

---

## Step 5 — Wayland sanity check (temporary)

Install sway:

```bash
sudo pacman -S sway
```

Launch:

```bash
sway
```

Inside sway:

```bash
glxinfo -B
```

Expected renderer example:
- `OpenGL renderer string: NVE7`

Exit sway:

```bash
Mod+Shift+E
```

---

## Step 6 — Install and test Niri

```bash
sudo pacman -S niri
```

If prompted for portal implementation, choose:
- `xdg-desktop-portal-wlr`

Enable portal:

```bash
systemctl --user enable xdg-desktop-portal
systemctl --user start xdg-desktop-portal
```

Start Niri once to generate config:

```bash
niri-session
```

Exit session.

---

## Step 7 — Configure terminal

Optionally, you could just install `alacritty`.

```bash
sudo pacman -S alacritty
```

Or, stick with `foot`

Edit Niri config:

```bash
nano ~/.config/niri/config.kdl
```

Ensure terminal startup uses `foot` (not alacritty):

```kdl
spawn-at-startup "foot"
```

Validate:

```bash
niri validate
```

---

## Step 8 — Browser

```bash
sudo pacman -S firefox
```

---

## Phase 1 completion checklist

- Wi‑Fi works without tether
- `niri-session` launches from TTY
- `foot` terminal opens
- Firefox runs
- GPU acceleration verified

Phase 1 is complete and repeatable.

