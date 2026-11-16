# craftctl

A lightweight, Bash-only Minecraft server manager for headless Linux.

`craftctl` runs multiple PaperMC servers on one machine — Raspberry Pi OS
Lite (64-bit), Debian, or Ubuntu Server — with systemd, automatic Java
selection, safe backups, and simple whitelist management.

- Repo: <https://github.com/ferranox/craftctl>
- Default server software: [PaperMC](https://papermc.io/) via the official
  [Fill API](https://fill.papermc.io/) (`https://fill.papermc.io/v3`)
- No Docker, no web server, no desktop, no background GUI.

## Features

- 100% Bash CLI with long options and `--help` for everything.
- Multiple isolated servers (own directory, jar, world, ports, Java, backups).
- PaperMC version selection; latest stable by default; checksums verified.
- Automatic Java major selection (8/11/16/17/21/25) with APT + Temurin fallback.
- `systemd` lifecycle (`craftctl@<name>.service`), graceful shutdown,
  `on-failure` restarts, journal + file logs.
- RCON on localhost only with unique per-server credentials.
- Offline-capable whitelist management.
- Timestamped, verified, prunable backups with safe restore.
- Pipe-friendly installer, idempotent updates, safe-by-default uninstall.
- ShellCheck + Bats tested, GitHub Actions CI.

## Supported operating systems

| OS | Arch | Status |
|---|---|---|
| Raspberry Pi OS Lite (64-bit) | `aarch64` | Primary target, prioritised |
| Debian 12+ | `x86_64`, `aarch64` | Supported |
| Ubuntu Server 22.04+ | `x86_64`, `aarch64` | Supported |

32-bit ARM (`armv7l`/`armhf`), non-systemd systems, Docker-only hosts, and
generic "has apt" derivatives are **not** supported — the installer explains
why instead of failing halfway. Details: `docs/installation.md`.

## Requirements

- 64-bit Linux with `systemd` (`/run/systemd/system` must exist).
- A normal user with `sudo` (do **not** run as root).
- Outbound HTTPS to `github.com`, `fill.papermc.io`, `fill-data.papermc.io`,
  `api.adoptium.net`.

Dependencies are installed automatically:
`bash`, `curl`/`wget`, `git`, `jq`, `tar`, `gzip`, `openssl`,
`ca-certificates`, `procps`.

## Installation

```bash
wget -qO- https://raw.githubusercontent.com/ferranox/craftctl/main/install | bash
```

or:

```bash
curl -fsSL https://raw.githubusercontent.com/ferranox/craftctl/main/install | bash
```

Re-running the installer repairs/updates without touching worlds, configs,
or backups. Layout:

```text
/opt/craftctl/                    Application files
/usr/local/bin/craftctl           Executable
/etc/craftctl/                    Global + RCON credentials
/var/lib/craftctl/servers/        Server data
/var/backups/craftctl/            Backups
```

Full details: `docs/installation.md`.

## Initial setup

```bash
craftctl --setup
```

Checks OS/arch/systemd, installs deps, creates the unprivileged `minecraft`
system user (all servers run as `minecraft`, never root), creates directories,
writes `/etc/craftctl/craftctl.conf` if missing, and prepares a default Java.

## Creating a Minecraft server

```bash
craftctl --create survival
craftctl --create survival --version 1.21.11
craftctl --create survival --version 26.1 --memory 4G
craftctl --create creative --version 1.21.11 --memory 2G --port 25566 --rcon-port 25585
```

What happens: latest stable build is resolved via Fill, the right Java is
selected/installed, a unique RCON password is generated (localhost only),
`server.properties`/`eula.txt`/`config.conf` are written, and the jar
checksum (`sha256` from Fill) is verified. Unknown versions are rejected —
never silently substituted. See `craftctl --versions`.

List and inspect:

```bash
craftctl --list
craftctl --info survival
```

## Choosing a Paper version

```bash
craftctl --versions
craftctl --versions --minecraft 1.21.11
```

PaperMC is the default and only bundled implementation. Versions come from
`GET /v3/projects/paper` (stable releases only in `--versions`); builds from
`GET /v3/projects/paper/versions/<ver>/builds?channel=STABLE`. A proper
`User-Agent` (`craftctl/<ver> (https://github.com/ferranox/craftctl)`) is sent
as required by PaperMC. Details: `docs/commands.md`.

## Java compatibility

Verified against <https://docs.papermc.io/paper/getting-started/>:

| Paper version | Java |
|---|---|
| 1.7.10 – 1.11 | 8 |
| 1.12 – 1.16.4 | 11 |
| 1.16.5 | 16 |
| 1.17 – 1.19 | 17 |
| 1.20 – 1.21.x | 21 |
| 26.1+ | 25 |

If the Fill version metadata reports a different minimum, the API value wins
(with a warning). Each server pins its own `JAVA_BIN`; the system default
Java is never changed. APT packages are preferred; otherwise official Eclipse
Temurin JREs are fetched over HTTPS from `api.adoptium.net` (x64/aarch64).
Java 8/16 may be unavailable in distro repos — that is exactly when Temurin
is used. No random binaries, no global `JAVA_HOME` surgery.

## Starting, stopping and restarting servers

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

These wrap `systemctl start|stop|restart craftctl@survival`. PaperMC gets
SIGTERM and saves worlds before exiting (`TimeoutStopSec=90`). Logs:

```bash
craftctl --logs survival --lines 200
craftctl --logs survival --follow
```

## Managing multiple servers

Every server is independent:

```text
/var/lib/craftctl/servers/
  survival/  (server.jar, world/, server.properties, config.conf, ...)
  creative/  (...)
```

Names (`[A-Za-z0-9_-]{1,32}`) map 1:1 to directories (no traversal possible).
Ports are unique across game + RCON. Java/memory/flags are per-server.

## Whitelist management

```bash
craftctl --whitelist add survival Alex
craftctl --whitelist remove survival Alex
craftctl --whitelist list survival
craftctl --whitelist on survival
craftctl --whitelist off survival
craftctl --whitelist status survival
```

Online servers use RCON (`whitelist ...`); offline servers edit
`whitelist.json`/`server.properties` directly (Mojang UUID lookup is
best-effort). Failures are reported, never silently claimed as success.
Usernames must match `^[A-Za-z0-9_]{3,16}$`. RCON never leaves localhost and
passwords are never printed.

## Backups and restoration

```bash
craftctl --backup survival --keep 7
craftctl --restore survival --file /var/backups/craftctl/survival/survival-20260101-120000.tar.gz
craftctl --restore survival --yes
```

Online backups freeze saving (`save-off` + `save-all flush`) and resume
afterwards. Archives exclude `logs/`/`cache`, are verified with `tar -tzf`,
and pruned to `BACKUP_RETENTION`. Restores require the server stopped plus
explicit confirmation, keep a pre-restore safety backup, and verify the
archive first.

## Configuration

Global: `/etc/craftctl/craftctl.conf` (`DEFAULT_VERSION`, `DEFAULT_MEMORY`,
`DEFAULT_PORT`, `BACKUP_RETENTION`, `JAVA_DIR`). Per-server:
`/var/lib/craftctl/servers/<name>/config.conf` + `server.properties`.
RCON: `/etc/craftctl/servers/<name>.rcon` (mode 600). Full reference:
`docs/configuration.md`.

## Updating craftctl

```bash
craftctl --update
```

Re-runs the installer path (or `git pull` for checkouts). Worlds, configs,
and backups are preserved; global config is never overwritten.

Update a game server instead:

```bash
craftctl --update-server survival
craftctl --update-server survival --version 1.21.11
```

Takes a pre-update backup, downloads the latest STABLE build (checksum
verified), adjusts Java if the version needs it, and restarts only if it was
running.

## Uninstallation

```bash
craftctl --uninstall           # keeps worlds, configs, backups (default)
craftctl --uninstall --purge   # deletes everything, asks twice
```

The `minecraft` user is left in place (remove manually if desired).

## Troubleshooting

Start here: `docs/troubleshooting.md`. Quick diagnostics:

```bash
systemctl status craftctl@survival
journalctl -u craftctl@survival -n 100 --no-pager
craftctl --logs survival --lines 200
```

Common issues: wrong Java (`UnsupportedClassVersionError`), duplicate ports,
missing RCON credentials after manual file moves, full disks for backups.

## Contributing

See `CONTRIBUTING.md` and `docs/development.md`. Bash only, ShellCheck-clean,
Bats-tested, docs updated with every CLI change.

## Licensing

MIT — see `LICENSE` (copyright `ferranox`, 2026). Functionality is inspired
by [MSCS](https://github.com/MinecraftServerControl/mscs) and the
Pi-Apps modular layout; no MSCS/Pi-Apps code is copied.
