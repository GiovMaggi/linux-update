# linux-update

One command, `update`, that updates your whole system (system packages + Flatpak apps) without asking for
confirmation, then reboots, powers off, or does nothing (you choose, or pick it up front with a flag).

| Script | Distro | What it runs |
|---|---|---|
| `update-arch` | Arch-based | `yay -Syu` (or `paru -Syu`, or `sudo pacman -Syu` if neither is installed) + `flatpak update` |
| `update-fedora` | Fedora-based | `sudo dnf upgrade` + `flatpak update` |
| `update-debian` | Debian/Ubuntu-based | `sudo apt update` + `sudo apt upgrade` + `flatpak update` |

All prompts are answered automatically. You'll only be asked for your password.

## Install

Pick the line for your distro. It saves the script as `/usr/local/bin/update`.

Arch (uses yay if installed, otherwise paru, otherwise pacman):

```bash
sudo curl -fsSL https://raw.githubusercontent.com/GiovMaggi/linux-update/main/update-arch -o /usr/local/bin/update && sudo chmod +x /usr/local/bin/update
```

Fedora:

```bash
sudo curl -fsSL https://raw.githubusercontent.com/GiovMaggi/linux-update/main/update-fedora -o /usr/local/bin/update && sudo chmod +x /usr/local/bin/update
```

Debian / Ubuntu:

```bash
sudo curl -fsSL https://raw.githubusercontent.com/GiovMaggi/linux-update/main/update-debian -o /usr/local/bin/update && sudo chmod +x /usr/local/bin/update
```

Then just type:

```bash
update
```

That updates everything. To update only part of it, name what you want:

| Command | What it updates |
|---|---|
| `update` | everything (system packages + Flatpak apps) |
| `update main` | system packages only (Arch: pacman + AUR, Fedora: `dnf upgrade`, Debian/Ubuntu: `apt update` + `apt upgrade`) |
| `update flatpak` | every Flatpak app only |
| `update steam` | just that app: the system package `steam` if it's installed, otherwise every Flatpak app with `steam` in its name or ID |

At the end it asks whether to reboot, power off, or do nothing. To choose up front and not be asked, add a flag
(before or after the name, e.g. `update steam -r` or `update main -n`):

| Flag | When the update is done |
|---|---|
| none | asks you |
| `-r` | reboots |
| `-p` | powers off |
| `-n` | does nothing |

`update -h` shows all of this in the terminal.
