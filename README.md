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
- Remote tool delegation for:
  - `read`
  - `write`
  - `edit`
  - `bash`
- Persistent remote shell session for bash commands
  - environment persists across commands (for example `export TEST=123`)
  - Ctrl-C interrupts the current remote command but keeps the SSH session alive
- Remote execution for user `!` commands
- Status indicator in the pi UI when SSH mode is active
- System prompt cwd rewrite to reflect remote cwd

## Requirements

- SSH client installed locally
- Passwordless SSH auth recommended (keys/agent)
- Remote host with:
  - `bash`
  - `cat`, `test`, `mkdir`, `pwd`
  - optional: `file` (for image mime detection)

## Install

### Option A: project-local extension

```bash
mkdir -p .pi/extensions
cp /path/to/pi-ssh/index.ts .pi/extensions/pi-ssh.ts
```

Then start pi in your project and pass `--ssh`.

### Option B: global extension

```bash
mkdir -p ~/.pi/agent/extensions
cp /path/to/pi-ssh/index.ts ~/.pi/agent/extensions/pi-ssh.ts
```

## Usage

### Use remote host default cwd

```bash
pi --ssh user@my-vm
```

### Use explicit remote workspace path

```bash
pi --ssh user@my-vm:/home/user/chromium/src
```

You should see a status line similar to:

```text
SSH user@my-vm:/home/user/chromium/src
```

## Typical workflow

1. Start pi locally with `--ssh ...`
2. Ask pi to inspect/edit files as usual
3. All tool operations run remotely
4. Keep local model switching, auth, and limits as usual

## Notes

- Absolute paths are strongly recommended for the remote path.
- If `--ssh` is not set, extension falls back to local tool behavior.
- Current version focuses on core coding tools (`read/write/edit/bash`).

## Troubleshooting

### "pi-ssh failed to connect"

Check:

```bash
ssh user@host
ssh user@host 'bash -lc "pwd"'
```

### Commands work locally but not remotely

Verify remote shell tools exist:

```bash
ssh user@host 'bash -lc "which bash cat test mkdir pwd"'
```

### Slow tool calls

Each operation opens an SSH process in this MVP. This is simple and robust, but adds overhead. Connection reuse can be added in a future version.

## Development

- Spec: `extension-spec.md`
- Extension entry: `index.ts`

## License

MIT
