# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Role Overview

Ansible role that installs [Alacritty](https://github.com/alacritty/alacritty) and deploys a TOML config. Supports Arch Linux, Debian/Ubuntu, and Steam Deck (SteamOS).

## Key Variables (`defaults/main.yml`)

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall and remove config |
| `alacritty.font.size` | `16` | Font size in the generated config |
| `alacritty.font.family` | `DejaVuSansMono Nerd Font Mono` | Font family |
| `alacritty_steamos_version` | `0.16.1` | Alacritty version pulled from `archive.archlinux.org` on Steam Deck, pinned for glibc compatibility |

## Task Flow

`tasks/main.yml` -> `install.yml` or `uninstall.yml` based on `install | bool`

**install.yml:**
1. Stat `/etc/steamos-release`
2. If it exists, include `steamos.yml` (SteamOS code path, takes priority over the generic Arch path)
3. Else include `debian.yml` (Debian family) or `archlinux.yml` (Arch family, non-SteamOS)
4. Install the `alacritty` package via `ansible.builtin.package` for non-Arch families (Arch installs happen in `archlinux.yml`/`steamos.yml` instead)
5. Template `~/.config/alacritty/alacritty.toml` from `templates/alacritty.toml.j2`

**debian.yml:** enables the `universe` apt repository (Alacritty itself installs via the generic `package` task in `install.yml`)

**archlinux.yml:** `pacman` install of `alacritty`

**steamos.yml:** No `become` anywhere (SteamOS root is read-only and there's no compiler toolchain or Flatpak listing for Alacritty).
- Downloads `alacritty-{{ alacritty_steamos_version }}-1-x86_64.pkg.tar.zst` from `archive.archlinux.org` — this is a plain zstd tarball, not installed via `pacman`
- Extracts just `usr/bin/alacritty` from it into `~/.local/bin/alacritty` via `unarchive` (`remote_src: true`, `--strip-components=2`)
- Deploys `files/Alacritty.svg` to `~/.local/share/icons/` and `templates/alacritty.desktop.j2` to `~/.local/share/applications/`, then runs `kbuildsycoca6` (if present) to refresh KDE's launcher cache so the entry shows up immediately

**uninstall.yml:** removes the SteamOS binary/desktop entry/icon (when `/etc/steamos-release` exists) or the package via `ansible.builtin.package` (everywhere else), then always removes `~/.config/alacritty`

## Why the SteamOS version is pinned

SteamOS's root filesystem is read-only, has no `pacman` write access, and ships no compiler toolchain or dev headers, ruling out building from source. Alacritty isn't published on Flathub either. The only viable path is extracting the binary directly from an archived Arch `.pkg.tar.zst` file. The current Arch `extra` build requires a newer glibc than SteamOS ships (`GLIBC_2.43` vs. SteamOS's `2.41`), so `alacritty_steamos_version` is pinned to `0.16.1`, the newest version confirmed to run against SteamOS's glibc. Bump it once SteamOS updates glibc.

## Testing

```bash
yamllint .
ansible-lint
molecule test
molecule test -s steamdeck
molecule converge
molecule destroy
```

Localhost scenario (runs directly against the local machine — used to validate the SteamOS path on real hardware):

```bash
molecule converge -s localhost
molecule verify -s localhost
```

## CI

- **Lint**: yamllint + ansible-lint
- **Molecule**: Ubuntu + Arch Linux via Docker (`molecule/default`)
- **Steam Deck**: Arch container with `/etc/steamos-release` stubbed (`molecule/steamdeck`)
- **Release**: publishes to Ansible Galaxy on merge to `main`
