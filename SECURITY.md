# Security Policy

## Supported versions

craftctl `main` is the only actively supported line. Security fixes land on
`main` first; there are no long-term maintenance branches.

## Reporting a vulnerability

Do **not** open a public issue for a suspected vulnerability.

Instead, open a
[private security advisory](https://github.com/ferranox/craftctl/security/advisories/new)
or contact the maintainer through the email listed on the GitHub profile.
Include:

- Affected version (`craftctl --version`) and OS/arch
- Steps to reproduce and impact assessment
- Whether RCON credentials, systemd units, or server data are involved

Expect an acknowledgement within 72 hours and a fix or mitigation plan as
soon as one is verified.

## Security model

- Minecraft processes run as the unprivileged `minecraft` user, never root.
- RCON binds to `127.0.0.1` only; per-server 128-bit passwords stored at
  mode `600` under `/etc/craftctl/servers/`.
- No `eval` of user input; server names, usernames, versions, memory and
  ports are validated; atomic writes + `flock` guard config and backups.
- Java runtimes come from distribution packages or official Eclipse Temurin
  builds (Adoptium API) over HTTPS; PaperMC jars are checksum-verified
  against the Fill API (`sha256`).
- Updates and uninstalls never silently delete worlds/backups
  (`--purge` requires double confirmation).
