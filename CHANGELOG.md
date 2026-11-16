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
