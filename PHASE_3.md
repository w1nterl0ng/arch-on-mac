# Phase 3 — System Hardening, Ergonomics, and Dotfiles

**Prerequisite:** Phase 1 and Phase 2 completed successfully on at least one clean machine.

At this point you have:

- Arch Linux (minimal)
- Niri + Noctalia working reliably
- Wayland + GPU acceleration verified
- Networking stable
- A documented, repeatable base

Phase 3 focuses on **making the system comfortable, maintainable, and reproducible** — without yet customizing it for your personal workflow.

---

## Goals (Phase 3)

By the end of Phase 3, you should have:

- Sensible power management for a laptop
- Predictable input behavior (keyboard, touchpad)
- A dotfiles strategy backed by git
- A small set of baseline daily-use applications
- A system that feels "finished" but not "personalized"

---

## Non-goals (Phase 3)

These are intentionally deferred:

- Backup / snapshot tooling (Snapper, Btrfs, etc.)
- Personal workflow apps (VPN, work tools)
- Heavy UI theming changes
- Automation and scripting

---

## Step 1 — Power management

### Purpose

Ensure the laptop behaves well on battery without fighting the compositor or shell.

### Recommendation

Use **power-profiles-daemon** (simple, modern, integrates well with Wayland).

```bash
sudo pacman -S power-profiles-daemon
sudo systemctl enable power-profiles-daemon
sudo systemctl start power-profiles-daemon
```

Verify:

```
powerprofilesctl get
```

If `powerprofilesctl` errors with `gi.repository` missing, install:

```
sudo pacman -S python-gobject
```

---

## Step 2 — Input devices (keyboard & touchpad)

### Purpose

Make typing and pointer behavior predictable and sane.

Install tools:

```bash
sudo pacman -S libinput
```

Most configuration will be done later via:

- Niri config
- Noctalia settings

At this stage, only verify devices are detected.

---

## Step 3 — Dotfiles (git-backed configuration)

### Purpose

Make your configuration:

- reproducible
- versioned
- portable across machines

### Strategy (recommended)

Use a **bare git repository** in `$HOME`.

```bash
git init --bare $HOME/.dotfiles
git --git-dir=$HOME/.dotfiles --work-tree=$HOME config status.showUntrackedFiles no
```

Alias for convenience (add to shell rc later):

```bash
alias dotfiles='git --git-dir=$HOME/.dotfiles --work-tree=$HOME'
```

From here, you can start tracking:

- `~/.config/niri/`
- `~/.config/noctalia/`
- `~/.config/waybar/` (even if dormant)
- shell config

Commit early and often.

---

## Step 4 — Display manager (greetd + tuigreet)

This step removes the need to manually type `niri-session` after boot, while keeping the system minimal and debuggable.

### 4.1 Install

```bash
sudo pacman -S greetd tuigreet
```

### 4.2 Configure greetd to launch Niri

Edit:

```bash
sudo nano /etc/greetd/config.toml
```

Use this config:

```toml
[terminal]
vt = 1

[default_session]
command = "tuigreet --time --remember --remember-session --cmd 'dbus-run-session niri-session'"
user = "greeter"
```

Notes:
- `dbus-run-session` is recommended so portal/dbus-dependent components behave normally.
- `--remember-session` will remember the last session command; we keep it pinned to `niri-session`.

### 4.3 Enable greetd

```bash
sudo systemctl enable greetd
sudo systemctl start greetd
```

Reboot to test.

### 4.4 Troubleshooting (common)

#### Symptom: login returns to greeter or you see `org.freedesktop.systemd1` / `Failed to start niri.service`

This usually means the greetd PAM stack is not creating a systemd user session.

Check:

```bash
sudo nano /etc/pam.d/greetd
```

Ensure it includes `system-local-login` and a `pam_systemd` session line, for example:

```pam
auth      include   system-local-login
account   include   system-local-login
password  include   system-local-login
session   include   system-local-login
session   required  pam_systemd.so
```

Then restart greetd:

```bash
sudo systemctl restart greetd
```

#### Logs

```bash
sudo journalctl -u greetd -b --no-pager | tail -200
```

If `niri.service` never started, `journalctl --user -u niri.service` may be empty.

---

### 4.5 Recovery / bypass

If greetd fails or you want to temporarily bypass it:

1) Switch to another TTY (e.g., Ctrl+Alt+F2)
2) Log in and disable greetd:

```bash
sudo systemctl disable --now greetd
```

You can then start your session manually:

```bash
niri-session
```

---

## Step 5 — Baseline applications

Install a small set of non-work-specific apps:

```bash
sudo pacman -S \
  unzip \
  zip \
  htop \
  neovim \
  file \
  ripgrep \
  fd
```

These support daily system use without committing to a workflow.

---

## Step 5 — Cleanup and validation

Optional but recommended:

- Remove packages used only during setup:

```bash
sudo pacman -Rns sway
```

- Reboot and confirm:
  - Niri starts cleanly
  - Noctalia autostarts
  - Input and power behavior feel reasonable

---

## Phase 3 completion checklist

Phase 3 is complete when all of the following are true:

- System boots to a login prompt (greetd + tuigreet)
- Logging in launches `niri-session` reliably
- Noctalia autostarts and renders correctly
- Power profiles work (`powerprofilesctl get` succeeds)
- Input devices behave predictably
- Dotfiles repo is initialized, committed, and pushed
- Baseline tools are installed
- System feels stable and "complete" without being personalized

---

## Phase 3 wrap-up

At the end of Phase 3, you now have:

- A minimal Arch system that boots cleanly
- A Wayland-native compositor (Niri)
- A lightweight shell (Noctalia)
- A minimal display manager (greetd + tuigreet)
- Git-backed configuration (dotfiles)

This system is:
- reproducible
- debuggable
- easy to rebuild
- intentionally boring

Nothing in Phase 3 is specific to your personal workflow.

---

## Phase 4 preview — Personal workflow (final phase)

Phase 4 is where the system becomes *yours*:

- VPN configuration
- Work and personal applications
- Web apps
- App launcher tuning
- Settings and UX polish
- Quality-of-life automation

Phase 4 is intentionally flexible and iterative.

