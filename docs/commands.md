# Commands

Full reference for `craftctl`. See also `craftctl --help`.

## General

```bash
craftctl --help
craftctl --version
craftctl --update                  # update craftctl itself (preserves data)
craftctl --uninstall [--purge]     # --purge deletes worlds/backups (confirms twice)
```

## Setup

```bash
craftctl --setup
```

Checks OS/arch/systemd, installs deps, creates the `minecraft` user and
directories, writes `/etc/craftctl/craftctl.conf` if missing, and ensures a
default Java runtime.

## Server creation and management

```bash
craftctl --create survival
craftctl --create survival --version 1.21.11
craftctl --create survival --version 26.1 --memory 4G
craftctl --create survival --version 1.21.11 --memory 4G --port 25566 --rcon-port 25585
craftctl --delete survival [--yes]
craftctl --list
craftctl --versions [--minecraft 1.21.11]
craftctl --info survival
```

- `--version` defaults to the latest stable PaperMC version (Fill API).
- Unknown versions are rejected (never silently substituted).
- `--memory` like `2G`/`4096M` (default `2G`).
- Ports default to the next free port from 25565 (game) / 25575 (RCON) and
  must be unique across servers.
- `--versions` lists stable releases only (no `-rc`/`-pre`).

## Lifecycle

```bash
craftctl --start survival
craftctl --stop survival
craftctl --restart survival
craftctl --status survival
craftctl --start --all
craftctl --stop --all
craftctl --restart --all
craftctl --status --all
```

Wraps `systemctl start|stop|restart craftctl@<name>`. PaperMC receives
SIGTERM and shuts down gracefully (worlds saved). `restart` policy is
`on-failure` with a 10s delay.

## Maintenance

```bash
craftctl --backup survival [--keep 7]
craftctl --restore survival [--file /var/backups/craftctl/survival/survival-20240101-120000.tar.gz] [--yes]
craftctl --update-server survival [--version 1.21.11]
craftctl --logs survival [--lines 100] [--follow]
```

- `--update` updates **craftctl**; `--update-server` updates the **PaperMC jar**.
- Backups are timestamped `tar.gz` per server, verified after creation, and
  pruned to `--keep`/`BACKUP_RETENTION` entries.
- Online backups freeze saving (`save-off`/`save-all flush`) via RCON and
  re-enable afterwards.
- Restore refuses to run while the server is online and refuses without
  explicit confirmation unless `--yes` is given. A pre-restore safety backup
  is kept.
- `--logs` prefers `journalctl -u craftctl@<name>`, falling back to
  `logs/latest.log`.

## Console commands

```bash
craftctl --send survival "say Hello everyone!"
```

Sends a command via RCON (localhost only). The server must be running and an
RCON client (`mcrcon` or `python3`) must be present.

## Whitelist

```bash
craftctl --whitelist add survival Alex
craftctl --whitelist remove survival Alex
craftctl --whitelist list survival
craftctl --whitelist on survival
craftctl --whitelist off survival
craftctl --whitelist status survival
```

- Online: uses RCON (`whitelist add/remove/list/on/off`).
- Offline: edits `whitelist.json` / `server.properties` directly (Mojang UUID
  lookup is best-effort).
- Reports an error if the operation was not confirmed by the server.

## Exit codes

- `0` success
- `1` runtime failure (server missing, API error, service failure)
- `2` usage/argument error
