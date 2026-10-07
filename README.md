# pi-ssh

Run pi locally, work on files remotely over SSH.

`pi-ssh` is a pi extension that gives you a Cursor-like remote SSH workflow:

- pi runs on your local machine
- model access, API keys, and billing stay local
- `read`, `write`, `edit`, and `bash` run on a remote host via SSH

## Why

This is useful when:

- your code/checkouts live on a VM
- your model/tooling access is easier locally
- you want one local account to drive many remote workspaces

## Features

- `--ssh user@host` or `--ssh user@host:/remote/path`
- optional port: `--ssh-port 2222` (alias: `-p 2222`); if not set, the port from `~/.ssh/config` is used (else 22)
- Remote context files: `AGENTS.md` / `CLAUDE.md` from the remote working directory are loaded into the system prompt
- Remote tool delegation for:
  - `read`
  - `write`
  - `edit`
  - `bash`
- SSH connection multiplexing (`ControlMaster`/`ControlPersist`) for faster repeated tool calls
- Persistent remote shell session for bash commands
  - uses your remote account's configured login shell (for example zsh)
  - environment persists across commands (for example `export TEST=123`)
  - Ctrl-C interrupts the current remote command but keeps the SSH session alive
- Remote execution for user `!` commands
- Status indicator in the pi UI when SSH mode is active
- System prompt cwd rewrite to reflect remote cwd

## Requirements

- SSH client installed locally
- Passwordless SSH auth recommended (keys/agent)
- Remote host with:
  - a standard login shell (for example `zsh` or `bash`)
  - `cat`, `test`, `mkdir`, `pwd`
  - optional: `file` (for image mime detection)

## Install

### Option A: pi package (recommended)

```bash
pi install npm:pi-ssh
```

### Option B: project-local extension

```bash
mkdir -p .pi/extensions
cp /path/to/pi-ssh/index.ts .pi/extensions/pi-ssh.ts
```

Then start pi in your project and pass `--ssh`.

### Option C: global extension

```bash
mkdir -p ~/.pi/agent/extensions
cp /path/to/pi-ssh/index.ts ~/.pi/agent/extensions/pi-ssh.ts
```

## Usage

### Use remote host default cwd

```bash
pi --ssh user@my-vm
# same, explicit default
pi --ssh user@my-vm --ssh-port 22
```

If the host has a `Port` entry in `~/.ssh/config` and you don't pass
`--ssh-port`, that port is used.

### Use explicit remote workspace path

```bash
pi --ssh user@my-vm:/home/user/chromium/src
# custom port
pi --ssh user@my-vm:/home/user/chromium/src -p 2222
```

You should see a status line similar to:

```text
SSH user@my-vm:/home/user/chromium/src (port 22)
```

## Typical workflow

1. Start pi locally with `--ssh ...`
2. Ask pi to inspect/edit files as usual
3. All tool operations run remotely
4. Keep local model switching, auth, and limits as usual

## Notes

- Absolute paths are strongly recommended for the remote path.
- Paths under local `$HOME` are mapped to remote `$HOME` in SSH mode (for example `~/.config/...`).
- If `--ssh` is not set, extension falls back to local tool behavior.
- `AGENTS.md` (preferred) or `CLAUDE.md` in the remote cwd is appended to the
  system prompt as remote project context, after any local context.
- Current version focuses on core coding tools (`read/write/edit/bash`).

## Troubleshooting

### "pi-ssh failed to connect"

Check:

```bash
ssh user@host
ssh user@host 'pwd'
```

### Commands work locally but not remotely

Verify remote shell tools exist:

```bash
ssh user@host 'which cat test mkdir pwd'
```

### Slow tool calls

`bash`/`!` and most `read`/`write`/`edit` operations use a shared persistent SSH session.
Very large writes fall back to one-shot SSH streaming for reliability.

## Development

- Spec: `extension-spec.md`
- Extension entry: `index.ts`

## License

MIT
