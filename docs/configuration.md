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
`JVM_FLAGS` manually then `craftctl --restart <name>`.

## RCON credentials: `/etc/craftctl/servers/<name>.rcon`

- Unique 32-hex-char password per server, generated at creation.
- Mode `600`, owned by `minecraft`/`root` where possible.
- Bound to `127.0.0.1` only (`rcon.ip=127.0.0.1` in `server.properties`).
- Never printed to logs or stdout. `rcon_send` scrubs it from output.

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
