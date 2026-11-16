# Changelog

All notable changes to this project are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [Unreleased]

### Fixed

- `craftctl --create` no longer fails with `Permission denied` on
  `/var/lock/craftctl/...` on fresh installs. The installer and `--setup`
  now set the lock directory to mode `1777`, and locking falls back to a
  per-user tmp directory when the system directory is not writable, with a
  clear error instead of a misleading "Another operation is already running".
- Progress logs (`log_info`) now go to stderr, so captured values such as
  the Java path can no longer be polluted (previously `Using Java:` could
  print embedded `[info]` lines).
- Full privilege model for non-root operation: reads fall back to `sudo`
  (`priv_cat`), writes escalate atomically (`priv_write`/`priv_install`),
  tarballs run as the `minecraft` user when direct access fails (`tar_data`),
  and `server_exists`/`list_servers`/port detection work on minecraft-owned
  trees. Previously almost every command after `--create` (`info`, `list`,
  `backup`, `restore`, `update-server`, offline whitelist, `logs`) broke
  with misleading errors when run as a normal user. Non-secret files are now
  `644` in traversable (`2755`) server dirs so routine reads need no sudo;
  `server.properties`, backup contents and RCON credentials stay restricted
  (`640`/`600`). `--setup` migrates older installs best-effort.
- `cmd_send` exit-status capture simplified (removed dead `|| rc=$?` chain).
- `find_free_port` no longer silently ignores bound ports when `ss` is
  missing (falls back to a `/dev/tcp` probe).
- Fill API and jar downloads now retry transient failures (`curl --retry`).
- `papermc_download_jar` uses a unique `mktemp` name instead of a
  predictable `.download.$$` file.
- `load_global_config` sanitizes `BACKUP_RETENTION`/`DEFAULT_PORT` so a
  hand-edited global config cannot break pruning or port allocation.

## [1.0.0] - 2026-11-16

### Added

- Initial release of craftctl (`https://github.com/ferranox/craftctl`).
- Bash-only PaperMC manager: `--setup`, `--create`, `--delete`, `--list`,
  `--versions`, `--info`, `--start/--stop/--restart/--status` (incl. `--all`),
  `--backup`, `--restore`, `--update-server`, `--logs`, `--send`,
  `--whitelist add/remove/list/on/off/status`, `--update`, `--uninstall`.
- Fill API (v3) integration with `User-Agent` identification, STABLE-channel
  resolution, and `sha256` verification.
- Automatic Java selection (8/11/16/17/21/25) with APT + Eclipse Temurin
  fallback; per-server `JAVA_BIN`.
- systemd `craftctl@.service` template (SIGTERM graceful shutdown,
  `on-failure` restart, `minecraft` user, hardening).
- Per-server RCON on `127.0.0.1` with unique 600-mode credentials; offline
  whitelist.json fallback.
- Timestamped, verified, prunable backups with pre-restore safety copies.
- Pipe-friendly installer, idempotent updates, safe-by-default uninstaller.
- Bash completion, ShellCheck + Bats suites, GitHub Actions CI, full docs.

[1.0.0]: https://github.com/ferranox/craftctl/releases/tag/v1.0.0
