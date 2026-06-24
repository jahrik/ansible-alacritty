# jahrik.alacritty

[![CI/CD](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-alacritty/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.alacritty-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/alacritty/)

Installs the [Alacritty](https://alacritty.org/) terminal emulator and deploys a TOML config with the [Srcery](https://srcery.sh/) colorscheme to `~/.config/alacritty/alacritty.toml`.

Depends on `jahrik.nerd_fonts` to install [DejaVu Sans Mono Nerd Font](https://github.com/ryanoasis/nerd-fonts).

## Supported Platforms

| Platform | Install Method |
|---|---|
| **Arch Linux** | `pacman` |
| **Debian / Ubuntu** | `apt` (universe repository) |
| **macOS** | Homebrew cask (`become: false`) |
| **SteamOS** | Extracts archived Arch `.pkg.tar.zst` to `~/.local/bin` |

## Usage

Include the role in your playbook:

```yaml
- hosts: all
  roles:
    - jahrik.alacritty
```

To uninstall and remove the config:

```yaml
- hosts: all
  vars:
    install: false
  roles:
    - jahrik.alacritty
```

## Variables

Override these variables to customize the installation:

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall. |
| `alacritty.font.size` | `16` | Font size in the generated config. |
| `alacritty.font.family` | `DejaVuSansMono Nerd Font Mono` | Font family in the generated config. |
| `alacritty_steamos_version` | `0.16.1` | Pinned version for SteamOS glibc compatibility. |

## Testing

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
```

### Additional Molecule Commands

```bash
# Test the SteamOS scenario
molecule test -s steamdeck

# Test against the local machine (e.g. macOS or a real Steam Deck)
molecule converge -s localhost
```

## License

GPLv2

## Author

jahrik@gmail.com
