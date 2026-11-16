# Troubleshooting

## Installation

| Symptom | Cause / fix |
|---|---|
| `Do not run the installer as root` | Run as your normal user (with `sudo` access), not `sudo bash`. |
| `Unsupported distribution` | Only Debian / Ubuntu Server / Raspberry Pi OS (64-bit) are supported. Derivatives that merely have `apt` are rejected on purpose. |
| `Unsupported architecture` | 32-bit ARM cannot run modern PaperMC/Java. Reinstall a 64-bit OS. |
| `systemd does not appear to be running` | You are in Docker/WSL1/chroot without systemd. craftctl requires systemd. |
| `apt-get update` fails | Check DNS/proxy and outbound HTTPS. |
| Download from `github.com` fails | Check firewall allow-list for `github.com`, `fill.papermc.io`, `fill-data.papermc.io`, `api.adoptium.net`. |

## Server creation

- `PaperMC version 'X' does not exist`: run `craftctl --versions` for the
  exact list. Pre-releases (`-rc`, `-pre`) are hidden from `--versions` but
  can be requested explicitly if Fill provides builds.
- `Permission denied` on `/var/lock/craftctl/...` (fixed in unreleased
  versions after 1.0.0): re-run the installer or
  `sudo mkdir -p /var/lock/craftctl && sudo chmod 1777 /var/lock/craftctl`.
- `No STABLE build ... using latest build`: the version has no stable build
  yet (very new or very old). The newest build of any channel is used with a
  warning.
- `Checksum mismatch`: delete any partial download and retry; check disk
  space.
- `Failed to download Temurin Java`: Adoptium may not publish that major for
  your arch. Java 8/16 on newer ARM boards are the usual cases; consider a
  newer Minecraft version instead.
- `Port N is already used`: each server needs unique game + RCON ports.
  Pick explicit `--port`/`--rcon-port`.

## Runtime

```bash
systemctl status craftctl@survival
journalctl -u craftctl@survival -n 100 --no-pager
craftctl --logs survival --lines 200
```

- `FAILED state`: the JVM exited. Common causes: wrong Java major
  (`UnsupportedClassVersionError` in logs), out of memory, corrupt jar.
- `server.jar missing`: the jar was deleted; run
  `craftctl --update-server survival` to re-download.
- Out of memory on Pi: lower `--memory` (e.g. `1G`) and reduce
  `view-distance` in `server.properties`.

## Permissions and "does not exist" errors

New installs keep server data owned by `minecraft`, so commands escalate
with `sudo` where needed. If you see `Permission denied`, wrong
`Server 'X' does not exist`, `?` values in `list`/`info`, or false
`Backup file not found` / `is corrupt` messages:

1. Make sure your user has `sudo` access and re-run `craftctl --setup`
   (it repairs ownership and modes, best effort).
2. On versions before this model (≤1.0.0 lock fix): re-run the installer —
   it is idempotent and preserves worlds/backups.
3. Manual repair:
   ```bash
   sudo mkdir -p /var/lock/craftctl && sudo chmod 1777 /var/lock/craftctl
   sudo chown -R minecraft:minecraft /var/lib/craftctl/servers/<name>
   sudo chmod 2755 /var/lib/craftctl/servers/<name>
   sudo chmod 644 /var/lib/craftctl/servers/<name>/config.conf
   ```

## Whitelist / RCON

- `No RCON credentials found`: `/etc/craftctl/servers/<name>.rcon` is
  missing. It is created at `--create`; restoring a server directory by hand
  without it breaks RCON.
- `RCON authentication failed`: password mismatch between the `.rcon` file
  and `server.properties`. Re-create the server or sync both files.
- `No RCON client available`: install `mcrcon` or `python3`.
- Offline whitelist edits apply at next startup; online edits apply
  immediately via the running server.

## Backups

- `Backup file is corrupt`: the archive failed `tar -tzf` verification and
  was deleted. Check free disk space (`df -h /var/backups/craftctl`).
- Restores require the server to be stopped and confirmed interactively
  (or `--yes`).

## Getting help

Open an issue at <https://github.com/ferranox/craftctl/issues> with:

- `craftctl --version`
- OS/arch: `cat /etc/os-release; uname -m`
- Relevant `journalctl -u craftctl@<name>` excerpt (redact nothing except
  RCON passwords — which are never logged anyway)
