# ALACRITTY

[![CICD](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.alacritty-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/alacritty/)

Installs [Alacritty](https://alacritty.org/) ([GitHub](https://github.com/alacritty/alacritty)) terminal emulator and deploys a TOML config with the [Srcery](https://srcery.sh/) colorscheme to `~/.config/alacritty/alacritty.toml`. Supports Arch Linux, Debian/Ubuntu, macOS, and Steam Deck. Font rendering requires [DejaVu Sans Mono Nerd Font](https://github.com/ryanoasis/nerd-fonts), installed automatically via the `jahrik.nerd_fonts` dependency.

## OS Support

| Platform | Install method |
|---|---|
| Arch Linux | `pacman` |
| Debian / Ubuntu | `apt`, after enabling the `universe` repository |
| macOS | Homebrew cask (`become: false`) |
| Steam Deck / SteamOS | Static binary extracted from an archived Arch `.pkg.tar.zst` package to `~/.local/bin` |

SteamOS has a read-only root filesystem with no `pacman` write access and no compiler toolchain, ruling out both package managers and building from source. The binary is extracted directly from an archived Arch package file — no root, no build required. A desktop entry and icon are deployed to `~/.local/share/applications` and `~/.local/share/icons` so Alacritty appears in KDE's app launcher, and `kbuildsycoca6` is run (if present) to refresh the launcher cache immediately. The SteamOS version is pinned to a build compatible with SteamOS's glibc; bump `alacritty_steamos_version` once SteamOS ships a newer glibc.

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall Alacritty and remove `~/.config/alacritty` |
| `alacritty.font.size` | `16` | Font size in the generated config |
| `alacritty.font.family` | `DejaVuSansMono Nerd Font Mono` | Font family |
| `alacritty_steamos_version` | `0.16.1` | Alacritty version pulled from `archive.archlinux.org` on Steam Deck; pinned for glibc compatibility |

## Example Playbook

```yaml
- hosts: all
  roles:
    - jahrik.alacritty
```

To uninstall:

```yaml
- hosts: all
  vars:
    install: false
  roles:
    - jahrik.alacritty
```

## Testing

```bash
molecule test
```

Step by step:

```bash
molecule converge
molecule verify
molecule destroy
```

Steam Deck scenario (Arch container with `/etc/steamos-release` stubbed):

```bash
molecule test -s steamdeck
```

Localhost scenario (runs directly against the local machine, e.g. a real Steam Deck or macOS):

```bash
molecule converge -s localhost
molecule verify -s localhost
```

## License

GPLv2

## Author Information

jahrik@gmail.com
