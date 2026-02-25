# pi-ssh extension spec

## Summary

`pi-ssh` makes local pi behave like "remote SSH mode" in editors:

- pi process runs on your local machine
- model/API keys stay local
- file and shell tools operate on a remote machine over SSH

This gives you the same main benefit Yoav described:

- remote code/workspace access
- local model/account usage (for example employer-funded local tooling)

## Problem statement

Current setup patterns are usually one of these:

1. Run pi on the VM
   - Pro: direct filesystem access
   - Con: model/API access on VM is harder (network policy, credentials, billing, SSO)

2. Run pi locally
   - Pro: easiest model/API access
   - Con: tools only see local filesystem

We want a third mode:

3. Run pi locally, target remote workspace over SSH
   - Pro: best of both worlds

## Goals

- Local model execution path (no reverse model tunnel in MVP)
- Remote workspace operations via SSH for core coding tools
- Minimal setup and low operational complexity
- Keep behavior close to built-in tools

## Non-goals (MVP)

- Full connection pooling and retry policy beyond one persistent shell
- Agent-side bidirectional tunnel protocol
- Remote model proxying

## Architecture options

### Option A: Tool-level SSH delegation (chosen for MVP)

Implement a pi extension that overrides built-in tools and delegates operations over SSH:

- `read`
- `write`
- `edit`
- `bash`

Optional support included in this project for read-only tools:

- `ls`
- `find`
- `grep`

How it works:

- Register a `--ssh` flag (`user@host` or `user@host:/remote/path`)
- Resolve remote cwd on startup (`pwd` if path omitted)
- Override tools by re-registering tool names
- Map local absolute paths to remote absolute paths
- Execute remote commands using `ssh ... bash -lc ...`
- Reuse built-in tool factories with custom operations

Pros:

- Leverages official extension APIs
- Small and understandable
- Works with existing pi tool loop and rendering

Cons:

- SSH process startup overhead per tool call
- Requires command-line tooling on remote host (bash, rg/fd/file/base64 depending on operation)

### Option B: Reverse tunnel for model requests

Run pi/tooling on VM and send model requests through a local gateway tunnel.

Pros:

- Keeps remote workspace execution native
- Can avoid per-command SSH startup

Cons:

- More moving parts (proxy/gateway, auth, reconnect, secrets)
- Harder to share and maintain
- Better handled as provider/proxy infra than first extension MVP

### Option C: Full custom provider + remote tool bridge

Blend provider override and remote operations in one extension.

Pros:

- Maximum flexibility

Cons:

- Highest complexity
- Not needed for first release

## Selected design

Use Option A as the first publishable OSS cut.

### Runtime model

- pi starts locally
- if `--ssh` is set, extension enters remote mode
- `bash` and user `!` commands run through one persistent remote shell session
- `read`/`write`/`edit` delegate to remote ops over SSH
- if `--ssh` is not set, local tools are used

### UX behavior

- status line shows active SSH target
- startup notification confirms mode
- `!` user bash commands also execute remotely
- bash environment persists during session (for example exported vars)
- Ctrl-C interrupts current remote command while keeping SSH shell alive
- system prompt cwd line is rewritten to remote cwd

### Safety

- no credential forwarding by default beyond SSH auth
- user explicitly enables mode with `--ssh`
- remote path mapping is deterministic

## CLI contract

### Flag

- `--ssh user@host`
- `--ssh user@host:/remote/path`

If path is omitted, remote cwd is detected with `pwd`.

## Tool behavior details

### read

- remote file read and image mime check (`file --mime-type` if present)

### write

- remote mkdir support
- remote file write via SSH stdin streaming

### edit

- composed from remote read + write operations

### bash

- remote command execution with output streaming
- supports abort and timeout behavior expected by built-in bash tool

### ls/find/grep

- optional read-only helpers delegated to remote shell tooling

## Error handling

- explicit errors on SSH failure
- fallback to local mode only when `--ssh` is not configured
- invalid SSH target causes startup error notification

## Packaging and publishing

Repository:

- `hjanuschka/pi-ssh`

Contents:

- `index.ts` extension entry
- `README.md` usage and troubleshooting
- `extension-spec.md` (this file)
- minimal `package.json`

## Future roadmap

1. Connection reuse (control master / mux)
2. Optional persistent remote shell channel
3. Better remote capability detection
4. Optional remote git metadata widget
5. Optional provider proxy mode for reverse model tunnel workflows
