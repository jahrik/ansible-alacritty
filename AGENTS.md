# ansible-alacritty

Ansible role that installs [Alacritty](https://alacritty.org/) ([GitHub](https://github.com/alacritty/alacritty)) terminal emulator and deploys a TOML config with the [Srcery](https://srcery.sh/) colorscheme. Supports Arch Linux, Debian/Ubuntu, macOS, and Steam Deck (SteamOS). Depends on `jahrik.nerd_fonts` for [DejaVu Sans Mono Nerd Font](https://github.com/ryanoasis/nerd-fonts).

## Key Variables

| Variable | Default | Description |
|---|---|---|
| `install` | `true` | Set to `false` to uninstall and remove config |
| `alacritty.font.size` | `16` | Font size in the generated config |
| `alacritty.font.family` | `DejaVuSansMono Nerd Font Mono` | Font family |
| `alacritty_steamos_version` | `0.16.1` | Alacritty version pulled from `archive.archlinux.org` on Steam Deck; pinned for glibc compatibility |

## Task Flow

`tasks/main.yml` -> `install.yml` or `uninstall.yml` based on `install | bool`

**install.yml:**
1. Include `steamos.yml` when `ansible_distribution == 'SteamOS'`
2. Include `debian.yml` when `ansible_os_family == 'Debian'` and not SteamOS
3. Include `archlinux.yml` when `ansible_os_family == 'Archlinux'` and not SteamOS
4. Include `darwin.yml` when `ansible_os_family == 'Darwin'`
5. Install `alacritty` via `ansible.builtin.package` for families not handled by the above (generic fallback)
6. Create `~/.config/alacritty/` and template `alacritty.toml.j2` -> `~/.config/alacritty/alacritty.toml`

**archlinux.yml:** `community.general.pacman` installs `alacritty` with `update_cache: true`

**debian.yml:** enables the `universe` apt repository (Alacritty itself installs via the generic `package` task in `install.yml`)

**darwin.yml:** `community.general.homebrew_cask` installs the `alacritty` cask, `become: false`

**steamos.yml:** No `become` anywhere (SteamOS root is read-only).
- Downloads `alacritty-{{ alacritty_steamos_version }}-1-x86_64.pkg.tar.zst` from `archive.archlinux.org` — extracted as a plain tarball, not installed via `pacman`
- Extracts just `usr/bin/alacritty` into `~/.local/bin/alacritty` via `unarchive` (`remote_src: true`, `--strip-components=2`)
- Deploys `files/Alacritty.svg` to `~/.local/share/icons/` and `templates/alacritty.desktop.j2` to `~/.local/share/applications/`
- Runs `kbuildsycoca6` (if present) to refresh KDE's launcher cache

**uninstall.yml:** removes the SteamOS binary/desktop entry/icon (when SteamOS), uninstalls the Homebrew cask (Darwin), or removes via `ansible.builtin.package` (everywhere else), then removes `~/.config/alacritty`

## Why the SteamOS version is pinned

SteamOS's root filesystem is read-only and ships no compiler toolchain, ruling out `pacman` and building from source. Alacritty is not on Flathub. The only viable path is extracting the binary from an archived Arch `.pkg.tar.zst`. Current Arch builds require a newer glibc than SteamOS ships, so `alacritty_steamos_version` is pinned to `0.16.1` — the newest version confirmed to run against SteamOS's glibc. Bump it once SteamOS updates glibc.

## Testing

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
molecule test -s steamdeck
molecule converge
molecule destroy
```

Localhost scenario (runs directly against the local machine — used to validate the SteamOS path on real hardware, and used in CI on the macOS runner):

```bash
molecule converge -s localhost
molecule verify -s localhost
```

## CI

- **Lint**: yamllint + ansible-lint
- **Molecule**: Ubuntu + Arch Linux via Docker (`molecule/default`)
- **Steam Deck**: Arch container with `/etc/steamos-release` stubbed (`molecule/steamdeck`)
- **macOS**: `ansible-playbook` against the GHA runner directly (`macos-latest`), using `molecule/localhost`
- **Release**: publishes to Ansible Galaxy on merge to `main` (needs: molecule, steamdeck, macos)
