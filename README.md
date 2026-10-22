# linux-update

One command, `update`, that updates your Linux system without asking you to confirm anything:
system packages, Flatpak apps, or just one app. When it's done it reboots, powers off, or does
nothing: it asks you, or you choose up front with a flag.

Works on Arch, Fedora and Debian/Ubuntu (and distros based on them). The only thing you'll be asked
for is your password.

- [Install](#install)
- [Quick reference](#quick-reference)
- [What to update](#what-to-update)
- [What to do when it's done](#what-to-do-when-its-done)
- [Every combination](#every-combination)
- [What it runs on each distro](#what-it-runs-on-each-distro)
- [Errors and edge cases](#errors-and-edge-cases)
- [Uninstall](#uninstall)

## Install

Pick the line for your distro. It saves the script as `/usr/local/bin/update`.

Arch / Arch-based (uses yay if installed, otherwise paru, otherwise pacman):

```bash
sudo curl -fsSL https://raw.githubusercontent.com/GiovMaggi/linux-update/main/update-arch -o /usr/local/bin/update && sudo chmod +x /usr/local/bin/update
```

Fedora / Fedora-based:

```bash
sudo curl -fsSL https://raw.githubusercontent.com/GiovMaggi/linux-update/main/update-fedora -o /usr/local/bin/update && sudo chmod +x /usr/local/bin/update
```

Debian / Ubuntu / based on them (Linux Mint, Pop!_OS, ...):

```bash
sudo curl -fsSL https://raw.githubusercontent.com/GiovMaggi/linux-update/main/update-debian -o /usr/local/bin/update && sudo chmod +x /usr/local/bin/update
```

To get a newer version later, run the same line again.

## Quick reference

```
update [main | flatpak | <app>] [-r | -p | -n]
```

| Command | What it does |
|---|---|
| `update` | update everything, then ask what to do |
| `update main` | update system packages only |
| `update flatpak` | update every Flatpak app only |
| `update <app>` | update just that app (e.g. `update steam`) |
| `... -r` | reboot when done, without asking |
| `... -p` | power off when done, without asking |
| `... -n` | do nothing when done, without asking |
| `update -h` | show the help |

## What to update

You can give **at most one** of these. Leave it out to update everything.

### `update`: everything

Updates your system packages, then every Flatpak app. If Flatpak isn't installed, that part is
skipped silently.

### `update main`: system packages only

Updates only what comes from your distro's package manager. Flatpak apps aren't touched.

- Arch: official packages with pacman, plus AUR packages if you have yay or paru
- Fedora: `sudo dnf upgrade`
- Debian/Ubuntu: `sudo apt update` then `sudo apt upgrade`

### `update flatpak`: Flatpak apps only

Updates every installed Flatpak app (and the runtimes they use). System packages aren't touched.

### `update <app>`: just one app

Replace `<app>` with the name of the app, for example `update steam`, `update firefox` or
`update spotify`. It looks for the app in this order:

1. **System package.** If a system package with exactly that name is installed (for example
   `steam` from pacman/AUR, dnf or apt), it updates only that package. Upper/lower case doesn't
   matter: `update Steam` finds the `steam` package.
2. **Flatpak app.** If there's no such system package, it searches your installed Flatpak apps for
   that word anywhere in the app's name or ID, ignoring upper/lower case. For example `update steam`
   finds `com.valvesoftware.Steam`, and `update chrome` finds `com.google.Chrome`. If several apps
   match, all of them are updated, and it prints which ones.
3. **Nothing found.** If it's neither a system package nor a Flatpak app, it says so, updates
   nothing, and exits without rebooting or powering off (even with `-r` or `-p`).

The system package always wins: if `nano` is installed as a system package and you also have a
Flatpak with "nano" in its name, `update nano` updates only the system package.

> **Arch note:** updating a single package means refreshing the package lists and updating only that
> package while the rest stay old (a "partial upgrade"). Arch doesn't officially support that, and
> it can occasionally break other apps. It's fine for a quick fix, but plain `update` or
> `update main` is the safe habit.

## What to do when it's done

After updating, it shows a summary and then either asks you or follows a flag.

**No flag:** it asks:

```
Choose action: [r]eboot / [p]oweroff / [n]othing (default):
```

Type `r` (or `reboot`) to reboot, `p` (or `poweroff`) to power off, or just press Enter (or anything
else) to do nothing.

**With a flag:** it doesn't ask:

| Flag | When the update is done |
|---|---|
| `-r` | reboots |
| `-p` | powers off |
| `-n` | does nothing |

Use only one flag. It can go **before or after** the name: `update steam -r` and
`update -r steam` are the same.

> `-r` and `-p` reboot / power off even if some update step failed. If you want to read the
> summary first, leave the flag out.

## Every combination

| Command | Updates | Then |
|---|---|---|
| `update` | system packages + Flatpak apps | asks |
| `update -r` | system packages + Flatpak apps | reboots |
| `update -p` | system packages + Flatpak apps | powers off |
| `update -n` | system packages + Flatpak apps | does nothing |
| `update main` | system packages | asks |
| `update main -r` | system packages | reboots |
| `update main -p` | system packages | powers off |
| `update main -n` | system packages | does nothing |
| `update flatpak` | Flatpak apps | asks |
| `update flatpak -r` | Flatpak apps | reboots |
| `update flatpak -p` | Flatpak apps | powers off |
| `update flatpak -n` | Flatpak apps | does nothing |
| `update steam` | just Steam (system package, else Flatpak) | asks |
| `update steam -r` | just Steam | reboots |
| `update steam -p` | just Steam | powers off |
| `update steam -n` | just Steam | does nothing |
| `update -h` / `update --help` | nothing (shows help) | — |

(`steam` stands for any app name.)

## What it runs on each distro

Every prompt from the package manager is answered automatically.

### Arch (`update-arch`)

It uses `yay` if it's installed, otherwise `paru`, otherwise `sudo pacman` (no AUR in that case).

| Command | Runs |
|---|---|
| `update main` | `yay -Syu --noconfirm --ask 20` |
| `update <app>` (system package) | `yay -Sy --needed --noconfirm --ask 20 <app>` |

- `--noconfirm` answers every prompt with its default.
- `--ask 20` flips the two defaults that would otherwise cancel the update: it removes conflicting
  packages and skips packages with broken dependencies.
- `--needed` leaves the app alone if it's already up to date.

### Fedora (`update-fedora`)

| Command | Runs |
|---|---|
| `update main` | `sudo dnf upgrade -y --allowerasing --no-best` |
| `update <app>` (system package) | `sudo dnf upgrade -y --allowerasing --no-best <app>` |

- `-y` answers yes to every prompt.
- `--allowerasing` removes conflicting packages instead of cancelling the update.
- `--no-best` skips a package whose newest version can't be installed, instead of cancelling.

### Debian / Ubuntu (`update-debian`)

| Command | Runs |
|---|---|
| `update main` | `sudo apt update`, then `sudo apt upgrade -y` |
| `update <app>` (system package) | `sudo apt update`, then `sudo apt install --only-upgrade -y <app>` |

- `-y` answers yes to every prompt.
- Package setup dialogs are turned off (`DEBIAN_FRONTEND=noninteractive`).
- When a package ships a new version of a config file you changed, your version is kept
  (`--force-confdef --force-confold`).
- `--only-upgrade` means an app that isn't installed is never installed by accident.

### Flatpak (all distros)

| Command | Runs |
|---|---|
| `update flatpak` (and the Flatpak part of plain `update`) | `flatpak update -y` |
| `update <app>` (Flatpak) | `flatpak update -y <matching app IDs>` |

## Errors and edge cases

| Situation | What happens |
|---|---|
| A step fails (no internet, broken mirror, ...) | the others still run; the summary lists every step that failed, with "Scroll up to see the error messages" |
| Everything worked | the summary says `All updates completed successfully!` |
| `update <app>` and the app isn't installed | `'<app>' is not installed as a system package or a Flatpak app. Nothing was updated.` and it stops |
| `update flatpak` but Flatpak isn't installed | `Flatpak is not installed. Nothing was updated.` and it stops |
| Two names, e.g. `update main flatpak` | error, nothing is updated (one thing at a time) |
| Two flags, e.g. `update -r -p` | error, nothing is updated |
| Unknown flag, e.g. `update -x` | error, nothing is updated |

When it stops with an error, it never reboots or powers off.

## Uninstall

```bash
sudo rm /usr/local/bin/update
```
