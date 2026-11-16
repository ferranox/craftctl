# Development

## Layout

- `craftctl`: central dispatcher (parses `--command`, sources `commands/*`).
- `api`: Pi-Apps-inspired loader that sources every `lib/*`.
- `commands/*`: one file per user-facing group (`setup`, `create`, ...).
- `lib/*`: reusable functions (`common`, `validation`, `config`,
  `dependencies`, `java`, `papermc`, `server`, `systemd`, `rcon`, `logging`).
- `tests/*`: Bats test suites with mocked system commands.

## Prerequisites

```bash
# Debian/Ubuntu
sudo apt-get install -y shellcheck bats jq
```

`shellcheck` and `bats` are used by CI (`.github/workflows/ci.yml`).
Tests never touch the real system: they override `CRAFTCTL_*` paths and stub
`systemctl`, `curl`, `need_root`, etc.

## Running checks

```bash
shellcheck craftctl api install uninstall lib/* commands/* tests/*
bats tests/
# or individual suites:
bats tests/test-commands
```

## Conventions

- Bash only. No Go/Python/Node/Rust in the application itself
  (a stdlib-only `python3` RCON fallback is an *external utility*, like `curl`).
- `set -uo pipefail` in executables; libraries must be safe when sourced
  (no `set -e` in `lib/*`).
- Never `eval` user input. Quote all expansions. Validate server names,
  usernames, versions, memory and ports before use.
- Atomic writes for config (`conf_set`, `atomic_write`).
- Per-server locking via `flock` (`with_server_lock`).
- No colours when stdout is not a TTY (`NO_COLOR` respected).
- Functions: `cmd_*` for commands, `*_for`/`ensure_*`/`find_*` in libs.
- Keep `README.md` and `docs/commands.md` in sync with the CLI.

## Adding a command

1. Add `cmd_foo` to `commands/foo`.
2. Wire `--foo` in the `case` block in `craftctl`.
3. Document in `README.md` + `docs/commands.md` + `completions/craftctl`.
4. Add Bats coverage in `tests/`.

## Releasing

1. Bump `CRAFTCTL_VERSION` in `craftctl` and `lib/common`.
2. Update `CHANGELOG.md`.
3. Tag: `git tag vX.Y.Z && git push --tags`.
