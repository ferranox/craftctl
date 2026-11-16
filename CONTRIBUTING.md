# Contributing to craftctl

Thanks for helping. craftctl is intentionally small, Bash-only, and aimed at
headless Raspberry Pi / Debian / Ubuntu servers.

## How to contribute

1. Fork `https://github.com/ferranox/craftctl` and create a branch.
2. Follow `docs/development.md` (ShellCheck + Bats must pass).
3. Keep changes Bash-only; do not add Go/Python/Node/Rust to the application.
4. Update `README.md` / `docs/` / `completions/craftctl` when you change the CLI.
5. Add or update Bats tests under `tests/`.
6. Open a pull request describing behaviour, testing, and docs updates.

## Code style

- `#!/usr/bin/env bash` + `# shellcheck shell=bash` headers.
- `set -uo pipefail` in executables; no `set -e` in sourced `lib/*`.
- Quote everything, validate all user input (see `lib/validation`).
- Never `eval` user input; use `conf_set`/`atomic_write` for files.
- Restraint with colour; respect `NO_COLOR` and non-TTY output.

## Tests

```bash
shellcheck craftctl api install uninstall lib/* commands/* tests/*
bats tests/test-commands tests/test-config tests/test-whitelist tests/test-install
```

Tests must not require root, network, or destructive operations: override
`CRAFTCTL_*` paths and stub `systemctl`/`curl` as the existing suites do.

## Reporting issues

Use the GitHub templates and include `craftctl --version`,
`cat /etc/os-release; uname -m`, and relevant
`journalctl -u craftctl@<name>` excerpts.
