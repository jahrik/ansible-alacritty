# ansible-alacritty

[![CICD](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml)

Installs [Alacritty](https://github.com/alacritty/alacritty) and deploys a TOML config (Srcery colorscheme) to `~/.config/alacritty/alacritty.toml`. Supports Arch Linux, Debian/Ubuntu, and Steam Deck.

Configuration is based on [en0's dotfiles](https://github.com/en0/dotfiles).

## OS Support

| Platform | Install method |
|---|---|
| Arch Linux | `pacman` |
| Debian / Ubuntu | `apt`, after enabling the `universe` repository |
| Steam Deck / SteamOS | Static binary extracted from an archived Arch package, to `~/.local/bin` |

Steam Deck is detected automatically via `/etc/steamos-release`. Because SteamOS has a read-only root filesystem and no compiler toolchain, the binary is extracted directly from an `.pkg.tar.zst` Arch package file (no `pacman`, no root, no build) and installed entirely under the user's home directory. A desktop entry and icon are deployed to `~/.local/share/applications` and `~/.local/share/icons` so Alacritty shows up in KDE's app launcher, and `kbuildsycoca6` is run (if present) to refresh the launcher cache immediately.

## Role Variables

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall Alacritty and remove `~/.config/alacritty` |
| `alacritty.font.size` | `16` | Font size in the generated config |
| `alacritty.font.family` | `DejaVuSansMono Nerd Font Mono` | Font family |
| `alacritty_steamos_version` | `0.16.1` | Alacritty version pulled from `archive.archlinux.org` on Steam Deck. Pinned to a build compatible with SteamOS's glibc; bump once SteamOS's glibc catches up to newer Alacritty builds |

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

Localhost scenario (runs directly against the local machine, e.g. a real Steam Deck):

```bash
molecule converge -s localhost
molecule verify -s localhost
```

## License

GPLv2

## Author Information

jahrik@gmail.com
