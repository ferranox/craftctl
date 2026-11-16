# Configuration

## Global config: `/etc/craftctl/craftctl.conf`

```ini
DEFAULT_VERSION=
DEFAULT_MEMORY=2G
DEFAULT_PORT=25565
BACKUP_RETENTION=7
JAVA_DIR=
```

- `DEFAULT_VERSION`: default PaperMC version for `--create` when `--version`
  is omitted. Empty = latest stable (resolved via Fill API at create time).
- `DEFAULT_MEMORY`: default heap (e.g. `2G`, `4096M`).
- `DEFAULT_PORT`: first port tried for new servers.
- `BACKUP_RETENTION`: backups kept per server.
- `JAVA_DIR`: optional override directory searched first for Java binaries.

The installer and `--setup` never overwrite this file once created.
Edit with any text editor, then create new servers to pick up defaults.

## Per-server config: `/var/lib/craftctl/servers/<name>/config.conf`

```ini
SERVER_NAME=survival
MC_VERSION=1.21.11
PAPER_BUILD=132
MEMORY=2G
SERVER_PORT=25565
RCON_PORT=25575
JAVA_BIN=/usr/lib/jvm/java-21-openjdk-arm64/bin/java
JVM_FLAGS=-XX:+UseG1GC ...
CREATED=2026-01-01T00:00:00Z
```

Managed by `craftctl --create` / `--update-server`. You may tune `MEMORY` or
`JVM_FLAGS` manually then `craftctl --restart <name>`. The file contains no
secrets and is world-readable so `list`/`info` work without sudo.

## RCON credentials: `/etc/craftctl/servers/<name>.rcon`

- Unique 32-hex-char password per server, generated at creation.
- Mode `600`, owned by the admin user who created the server. Reads fall
  back to `sudo cat`, so other admins keep working (with a password prompt).
- Bound to `127.0.0.1` only (`rcon.ip=127.0.0.1` in `server.properties`).
- Never printed to logs or stdout. `rcon_send` scrubs it from output.

## Permissions and sudo

craftctl runs as your normal user; Minecraft processes run as `minecraft`.
The ownership model keeps routine reads password-free while secrets stay
restricted:

| Path | Owner | Mode | Notes |
|---|---|---|---|
| `/var/lib/craftctl`, `/var/lib/craftctl/servers`, `/var/backups/craftctl` | `root:root` | `755` | traversable, no sudo needed to list |
| `/var/lib/craftctl/servers/<name>/` | `minecraft:minecraft` | `2755` | traversable; files govern access |
| `config.conf`, `whitelist.json`, `eula.txt` | `minecraft:minecraft` | `644` | no secrets; readable without sudo |
| `server.properties`, backup archives | `minecraft:minecraft` | `640` | contain the RCON password |
| `/etc/craftctl/` | `root:minecraft` | `2750` | global config `644`, RCON creds `600` |
| `/var/lock/craftctl` | any | `1777` | sticky, like `/tmp` |

All writes to system locations escalate via `sudo` automatically
(`priv_write`/`priv_install`/`tar_data` helpers); reads fall back to
`sudo cat` only for the restricted files above. If `sudo` is unavailable
and files are not directly accessible, commands fail with a clear error
instead of corrupt output. `--setup` migrates pre-existing installs to
this model (best effort).

## server.properties

Generated at creation with enforced keys:

- `server-port`, `enable-rcon=true`, `rcon.port`, `rcon.password`,
  `rcon.ip=127.0.0.1`.

Other keys are preserved across updates (a `.bak.<timestamp>` copy is left
next to the file when managed keys are rewritten).

## JVM flags

Defaults are Aikar-inspired G1 flags, conservative for Raspberry Pi
(see `lib/server`: `default_jvm_flags`). Per-server override via `JVM_FLAGS`.

## Data layout

```text
/var/lib/craftctl/servers/<name>/
  server.jar  server.properties  eula.txt  config.conf
  whitelist.json  world/  logs/
```

## Environment overrides (testing/development)

All root paths can be redirected without code changes:

```bash
export CRAFTCTL_ROOT=/tmp/fake-opt
export CRAFTCTL_ETC=/tmp/fake-etc
export CRAFTCTL_DATA=/tmp/fake-data
export CRAFTCTL_BACKUP_ROOT=/tmp/fake-backups
export CRAFTCTL_SYSTEMD_DIR=/tmp/fake-systemd
export CRAFTCTL_LOCK_DIR=/tmp/fake-lock
```
