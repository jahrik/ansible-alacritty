# AGENTS.md

Guidance for AI agents working in `ansible-alacritty`.

## Overview

Ansible role to install and configure [Alacritty](https://alacritty.org/) with the [Srcery](https://srcery.sh/) colorscheme. Supports Arch Linux, Debian/Ubuntu, macOS, and SteamOS.

## Task Flow

`tasks/main.yml` conditionally includes `install.yml` or `uninstall.yml` based on the `install` boolean.

### Installation Details

- **Arch Linux:** Installed via `pacman`.
- **Debian/Ubuntu:** Enables the `universe` repo, installs via `apt`.
- **macOS:** Installed via `homebrew_cask` (`become: false`).
- **SteamOS:**
  - The root filesystem is read-only without a compiler toolchain.
  - Alacritty is extracted from an archived Arch `.pkg.tar.zst` directly to `~/.local/bin/alacritty`.
  - Deploys a desktop entry (`~/.local/share/applications/`) and icon (`~/.local/share/icons/`).
  - Runs `kbuildsycoca6` to refresh KDE's launcher cache.
  - **Pinned Version:** `alacritty_steamos_version` is pinned because newer Arch builds require a newer glibc than SteamOS provides. Update this once SteamOS updates glibc.

## Testing

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
```

### Scenarios

- **`default`:** Ubuntu + Arch Linux via Docker.
- **`steamdeck`:** Arch container with `/etc/steamos-release` stubbed.
  ```bash
  molecule test -s steamdeck
  ```
- **`localhost`:** Runs against the local machine. Used in CI for the macOS runner and for testing on real Steam Deck hardware.
  ```bash
  molecule converge -s localhost
  molecule verify -s localhost
  ```

## CI/CD

- **Linting:** Runs `yamllint` and `ansible-lint`.
- **Molecule:** Runs `default` and `steamdeck` scenarios via Docker.
- **macOS:** Runs `ansible-playbook` directly against the `macos-latest` GHA runner using the `localhost` scenario.
- **Release:** Automatically publishes to Ansible Galaxy on merge to `main`.
