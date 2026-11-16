# Installation

craftctl targets headless 64-bit Linux:

- Raspberry Pi OS Lite (64-bit)
- Debian 12+ (64-bit x86-64 and ARM64)
- Ubuntu Server 22.04+ (64-bit x86-64 and ARM64)

Other Debian derivatives may work but are not officially supported. 32-bit
ARM (`armv7l`) is not supported: modern PaperMC and Java require 64-bit.

## Requirements

- 64-bit OS (`x86_64` or `aarch64`)
- `systemd` as init system (`/run/systemd/system` must exist)
- A normal (non-root) user with `sudo` access
- Outbound HTTPS to `github.com`, `fill.papermc.io`, `fill-data.papermc.io`
  and `api.adoptium.net` (for Java downloads)

Dependencies (installed automatically): `bash`, `curl`/`wget`, `git`,
`jq`, `tar`, `gzip`, `openssl`, `ca-certificates`, `procps`.

## Install

```bash
wget -qO- https://raw.githubusercontent.com/ferranox/craftctl/main/install | bash
```

or with curl:

```bash
curl -fsSL https://raw.githubusercontent.com/ferranox/craftctl/main/install | bash
```

The installer will:

1. Refuse to run as root (run as your normal user; `sudo` is used internally).
2. Detect OS and architecture and stop on unsupported systems.
3. Verify `systemd` availability.
4. Install missing dependencies via APT.
5. Create the unprivileged `minecraft` system user.
6. Create `/opt/craftctl`, `/etc/craftctl`, `/var/lib/craftctl/servers`,
   `/var/backups/craftctl`.
7. Download the repository tarball into a temp staging directory
   (the pipe-friendly installer never relies on `$0`/CWD).
8. Install files, `/usr/local/bin/craftctl`, and `craftctl@.service`.
9. Preserve existing world/config/backup data on re-install.

Re-running the installer safely repairs/updates the application without
touching worlds, configs or backups.

## Post-install

```bash
craftctl --setup     # check deps, write default config, prep Java
craftctl --create survival
craftctl --start survival
```

## Layout

```text
/opt/craftctl/                    Application files
/usr/local/bin/craftctl           Executable
/etc/craftctl/                    Global + per-server RCON credentials
/etc/craftctl/craftctl.conf       Global defaults (never overwritten on update)
/etc/craftctl/servers/<n>.rcon   Per-server RCON password (mode 600)
/var/lib/craftctl/servers/<name>/ Server dirs (jar, world, configs)
/var/backups/craftctl/<name>/     Per-server backups
/etc/systemd/system/craftctl@.service  systemd template
```

All Minecraft processes run as `minecraft`, never as root.

## Uninstall

```bash
craftctl --uninstall          # keeps worlds, configs, backups
craftctl --uninstall --purge  # deletes EVERYTHING after double confirmation
```

## Troubleshooting

- `Unsupported distribution`: you are not on Debian/Ubuntu/Raspberry Pi OS.
  See `docs/troubleshooting.md`.
- `systemd does not appear to be running`: you are likely in a container
  without systemd; craftctl requires systemd.
- Download failures: check outbound HTTPS and proxy settings.
