# Phase 2 — Desktop Experience (Waybar, Launcher, Noctalia)

**Prerequisite:** Phase 1 completed successfully.

You should already have:

- Arch Linux (minimal)
- Working Wi-Fi (Broadcom `wl`)
- Wayland + GPU acceleration verified
- Niri launching from TTY via `niri-session`
- `foot` terminal
- Firefox installed

Phase 2 focuses on turning the *working shell* from Phase 1 into a **comfortable daily driver desktop**.

---

## Goals (Phase 2)

By the end of Phase 2, you should have:

- A visible top bar (Waybar)
- A working application launcher
- Consistent fonts and icons
- Noctalia theming applied
- Sensible defaults for daily use

---

## Non-goals (Phase 2)

These are intentionally deferred:

- Login manager / autologin
- Advanced power tuning
- Laptop-specific quirks (backlight keys, lid close behavior)
- Scripting / automation

---

## Step 0 — Recovery checkpoints (rsync)

Before making visual or configuration-heavy changes, create a simple recovery checkpoint. This project intentionally avoids Btrfs/Snapper/Limine-style boot snapshots in favor of **explicit, transparent rsync checkpoints** that work on ext4 and match the minimalism of Phase 1.

### 0.1 Create a snapshots directory

- sudo mkdir -p /snapshots
- sudo chown root\:root /snapshots
- sudo chmod 700 /snapshots

### 0.2 Install rsync (if missing)

- sudo pacman -S rsync

### 0.3 Create a named checkpoint (example: before Waybar)

> **Important:** If your destination directory lives on the same filesystem as `/` (e.g., `/snapshots/...`), you **must** exclude `/snapshots` or rsync will recursively copy snapshots into snapshots.

Use this command:

```bash
sudo rsync -aAXH --numeric-ids --one-file-system --info=progress2 \
  --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/snapshots/*","/var/cache/*","/var/tmp/*"} \
  / \
  /snapshots/phase2-pre-waybar/
```

This takes a few minutes and is meant to be done **only at milestones**.

### 0.4 Restore from a checkpoint (if needed)

1. Boot into Arch (TTY is sufficient)
2. Restore the snapshot:

- sudo rsync -aAXH --numeric-ids --one-file-system --info=progress2 \
  /snapshots/phase2-pre-waybar/ \
  /

3. Reboot

> These checkpoints are not automated and not scheduled. They are **intentional recovery points** taken before risky changes.
>
> **Note:** Restoring `/` while running from the same root filesystem can overwrite in-use files. Prefer restoring from a live ISO (or at minimum, stop your session and restore from a TTY with minimal services running). They are **intentional recovery points** taken before risky changes.

---

## Step 1 — Waybar

### Purpose

Waybar provides:

- workspace visibility
- clock / status indicators
- tray icons

This is the first *visual* layer added on top of Niri.

---

### 1.1 Install Waybar

```bash
sudo pacman -S waybar
```

> If prompted for font providers during dependency resolution, choose **ttf-dejavu**. Nerd Fonts are handled explicitly elsewhere.

---

### 1.2 First launch (no config)

Niri’s default configuration will automatically spawn Waybar **if it exists**.

From TTY:

```bash
niri-session
```

Expected result:

- Waybar appears at the top of the screen
- **No workspace indicators yet** (this is expected)

> **Important:** Waybar does not know about Niri workspaces unless explicitly configured. Unlike sway/i3, workspace indicators will **not** appear by default.

If Waybar does **not** appear automatically, start it manually inside Niri:

```bash
waybar &
```

---

### 1.3 Verify Waybar is running

Inside a terminal:

```bash
ps aux | grep waybar
```

You should see a running `waybar` process.

---

### 1.4 Create Waybar config directories

```bash
mkdir -p ~/.config/waybar
```

We will add:

- `config` (behavior)
- `style.css` (appearance)

in later steps. For now, Waybar will run with defaults.

---

### 1.5 Known-good checkpoint (optional but recommended)

If this is your first time running Waybar on this system, consider taking a recovery checkpoint now:

```bash
sudo rsync -aAXH --numeric-ids --one-file-system --info=progress2 \
  --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/snapshots/*","/var/cache/*","/var/tmp/*"} \
  / \
  /snapshots/phase2-waybar-running/
```

---

### Step 1 done when:

- Waybar is visible
- Workspace indicators update as windows open/close
- System remains responsive

We will address fonts, icons, and layout next.

---

## Step 2 — Waybar configuration (workspaces + clock)

This step adds **explicit workspace indicators** and a basic clock to Waybar. Nothing fancy yet.

---

### 2.1 Create Waybar config file

Create the config file:

```bash
nano ~/.config/waybar/config
```

Paste the following minimal configuration:

```json
{
  "layer": "top",
  "position": "top",
  "modules-left": ["niri/workspaces"],
  "modules-center": [],
  "modules-right": ["clock"],

  "niri/workspaces": {
    "format": "{index}",
    "on-click": "activate"
  },

  "clock": {
    "format": "{:%Y-%m-%d %H:%M:%S}"
  }
}
```

Save and exit.

---

### 2.2 Restart Waybar

Inside Niri, restart Waybar:

```bash
pkill waybar
waybar &
```

---

### 2.3 Verify behavior

You should now see:

- numbered workspaces (1, 2, 3, …) on the left
- clock on the right
- clicking a workspace activates it

#### Clock formatting note

Waybar’s clock uses a **fmt-style** time format inside braces. This is correct:

- `{:%Y-%m-%d %H:%M:%S}`

If you omit the braces/colon (e.g., `%Y-%m-%d %H:%M:%S`), Waybar may print the characters literally.

##### Seconds not updating?

By default, Waybar’s clock **updates once per minute**. If you include seconds in the format without changing the update interval, you may see a static seconds value that only resets when the minute rolls over.

To update every second, set an explicit interval:

```json
"clock": {
  "interval": 1,
  "format": "{:%Y-%m-%d %H:%M:%S}"
}
```

> **Note:** Updating every second is fine on modern systems, but it causes more frequent redraws. If you prefer less churn, omit seconds or use a larger interval.

#### Desktop appearance warning

You may see a log message like:

- `Unable to receive desktop appearance: ... No such interface “org.freedesktop.portal.Settings” ...`

This is a **common, non-fatal** warning when the desktop portal settings interface is unavailable; Waybar can still run normally.

If you want to address it, ensure portals are installed and restart them:

```bash
sudo pacman -S --needed xdg-desktop-portal xdg-desktop-portal-wlr
systemctl --user restart xdg-desktop-portal
```

This confirms Waybar ↔ Niri integration is working.

---

### Step 2 done when:

- Workspace numbers appear
- Clicking a number switches workspaces
- Clock updates once per minute

---

## Step 3 — Application Launcher

### Purpose

A launcher allows starting applications without using a terminal.

Candidate:

- wofi (Wayland-native, simple)

---

## Step 3 — Fonts & Icons

This step ensures text and symbols render correctly and prepares Waybar for icon-based modules later.

---

### 3.1 Install fonts

Install a clean baseline font plus a Nerd Font for symbols:

```bash
sudo pacman -S --needed ttf-dejavu ttf-jetbrains-mono-nerd
```

> `ttf-dejavu` satisfies generic font needs. `JetBrains Mono Nerd` provides glyphs/icons used by Waybar modules.

---

### 3.2 Verify fonts are available

```bash
fc-list | grep -i "JetBrainsMono"
```

You should see multiple entries ending in `JetBrainsMono Nerd Font`.

---

### 3.3 Set Waybar font

Create a minimal Waybar stylesheet:

```bash
nano ~/.config/waybar/style.css
```

Paste:

```css
* {
  font-family: "JetBrainsMono Nerd Font", "DejaVu Sans", sans-serif;
  font-size: 12px;
}
```

Save and exit.

---

### 3.4 Reload Waybar

```bash
pkill waybar
waybar &
```

---

### Step 3 done when:

- Text renders cleanly (no tofu boxes)
- Waybar uses JetBrains Mono
- No errors when restarting Waybar

---


## Step 4 — Noctalia

Noctalia is a sleek Wayland shell setup built on Quickshell, with native support for Niri. ([github.com](https://github.com/noctalia-dev/noctalia-shell?utm_source=chatgpt.com))

> **Minimalism note:** We will **test** Noctalia first, then decide whether it replaces Waybar or just complements it. Keep changes reversible.

---

### 4.0 Recovery checkpoint (recommended)

```bash
sudo rsync -aAXH --numeric-ids --one-file-system --info=progress2 \
  --exclude={"/dev/*","/proc/*","/sys/*","/tmp/*","/run/*","/mnt/*","/media/*","/lost+found","/snapshots/*","/var/cache/*","/var/tmp/*"} \
  / \
  /snapshots/phase2-pre-noctalia/
```

---

### 4.1 Install Noctalia (Arch)

Noctalia’s docs recommend installing via AUR using your helper (examples use `paru`). ([docs.noctalia.dev](https://docs.noctalia.dev/getting-started/installation/?utm_source=chatgpt.com))

Install and build yay

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```


Using `yay`:

```bash
yay -S noctalia-shell
```

---

### 4.2 First test launch (inside Niri)

From a terminal inside your Niri session:

```bash
# optional: stop Waybar to avoid UI overlap during testing
pkill waybar

# start Noctalia
qs -c noctalia-shell
```

(If `qs` is not found after install, log out/in and try again.)

---

### 4.3 Make Noctalia start automatically (chosen)

You have decided to **replace Waybar with Noctalia**. Waybar will remain installed but will not autostart.

#### 4.3.1 Edit Niri config

```bash
nano ~/.config/niri/config.kdl
```

Ensure you have **exactly one** Noctalia startup entry, and **no Waybar startup entries**.

Add (or keep) this line:

```kdl
spawn-at-startup "qs" "-c" "noctalia-shell"
```

If you see any of the following, remove or comment them out:

```kdl
spawn-at-startup "waybar"
```

#### 4.3.2 Validate config

```bash
niri validate
```

Validation must pass with no errors.
```

---

### 4.4 Decide: keep Waybar or replace it

- **Replace Waybar:** keep Waybar installed but do not start it automatically.
- **Keep Waybar:** restart it after testing:

```bash
waybar &
```

---

## Step 5 — Session polish

Potential items:

- Autostart ordering
- Restart behavior
- Cleanup of unused packages (e.g., sway)

---

## Phase 2 completion checklist

Phase 2 is complete when all of the following are true:

- Niri launches reliably via `niri-session`
- Noctalia starts automatically and renders correctly
- Waybar is **not** autostarted (installed only as fallback)
- Fonts render correctly (no placeholder/tofu glyphs)
- Launcher functionality is available via Noctalia
- System feels usable for daily work

---

## Phase 2 wrap-up notes

At the end of Phase 2, the system has intentionally:

- No display manager
- No automatic snapshot tooling
- No background automation

All desktop behavior is explicit, minimal, and debuggable.

This state is considered **stable and repeatable**.

---

## Optional cleanup (recommended before Phase 3)

If you want to remove packages used only for validation:

```bash
sudo pacman -Rns sway
```

Waybar may be kept installed as a fallback or removed later.

---

---

## Phase 3 preview

- Power management
- Laptop ergonomics
- Optional display manager
- Backup / snapshots
- Long-term maintenance notes

