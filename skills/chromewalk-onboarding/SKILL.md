---
name: chromewalk-onboarding
description: Get a user from "license required" to a working ChromeWalk without leaving the chat - sign them up with chromewalk_register (key is emailed, never shown), activate with chromewalk_activate, optionally remember the key for every install, and connect more AI clients. Use when a ChromeWalk tool returns license_required or exit 4, when the user has no key, or when setting ChromeWalk up for the first time.
---

# ChromeWalk onboarding (sign-up and activation)

Until an install is activated, every browser tool answers `license_required` (exit 4,
`error.class: license`). Two tools work WITHOUT a license so you can fix that inside the conversation:

| Tool | Argument | What it does |
|---|---|---|
| `chromewalk_register` | `email` (required) | emails the user their license key; the key is never returned to you or the chat |
| `chromewalk_activate` | `key` (required) | activates this install; never echoes the key back |

CLI equivalents: `chromewalk register you@example.com`, `chromewalk activate <key>`.

## The conversation

1. A tool reports `license_required` (or exit 4). Do not retry in a loop.
2. Ask: "ChromeWalk needs a license first. Do you already have a key? If not, I can sign you up - what
   email address should it go to?"
3. User gives an email: call `chromewalk_register` with it. Tell them to check their inbox (and spam).
   Do not ask anything else about the account; do not invent or guess a key.
4. The user pastes the key (36 characters, `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`): call
   `chromewalk_activate` with it.
5. On success, retry the tool that failed.

Privacy rules for you:
- The email and key travel to ChromeWalk in a private args file, never on a command line.
- Never repeat the key back, log it, write it into a file, a config, a commit or a document.
- If the user would rather not paste the key in chat, they can put it in the Claude Desktop extension's
  License Key setting, or run `chromewalk activate` in their own terminal.
- Register only with an email the user gave you in this conversation.

## Checking the state
`chromewalk license` (add `--json`) shows ACTIVE or why not. `chromewalk license status` is the same.
Never license-gated.

## Remember the key once per computer (Windows)
```
chromewalk activate <key> --remember
chromewalk license              # ACTIVE
chromewalk license forget-key   # remove the remembered key
```
`--remember` also stores the key in Windows Credential Manager for this Windows user, so every other
ChromeWalk install of that user (another folder, the Claude Desktop extension) that has no key activates
itself the first time it needs to. Only a key the license server accepted is remembered. Activations
are still tied to the device and the license's device limit still applies.

Where an install looks for a key, in order:
1. env `CHROMEWALK_LICENSE` (what the extension's License Key field sets)
2. env `CHROMEWALK_LICENSE_FILE` - a path to a file holding the key (CI and headless machines; keep the
   file readable by that user only - ChromeWalk warns, does not refuse, if others can read it)
3. the key remembered in Credential Manager

macOS / Linux: use `CHROMEWALK_LICENSE_FILE` or `chromewalk activate`.

## Errors you may meet
- `license_required` exit 4: no license, expired, revoked, another machine, or a state that cannot be
  evaluated. `chromewalk license --json` tells which.
- Activation refused: the key was mistyped or is not valid for this email/device. Ask the user to
  re-copy it from the email; do not retry in a loop (a failed automatic activation is not retried for an
  hour).
- No network: activation talks to chromewalk.com; say so rather than retrying.

## First-run setup (after activation)
```
chromewalk setup        # Chrome for Testing + Playwright engines (Firefox, WebKit, Chromium), then doctor
chromewalk doctor       # read-only health check; exit 5 only if a REQUIRED check fails
```
`setup` is idempotent. `CW_SETUP_NOPAUSE=1` skips the final "press any key" in automation.

## Connect more AI tools
```
chromewalk connect --list           # detected clients and status
chromewalk connect                  # register ChromeWalk in every detected client (idempotent)
chromewalk connect codex cursor     # only these
chromewalk connect --remove         # undo
```
Detected clients: Claude Desktop, Claude Code, Codex CLI, Cursor, Windsurf, Cline, Gemini CLI, VS Code
(Copilot), Zed, Continue, GitHub Copilot CLI (`copilot-cli`), Kiro (`kiro`), Amazon Q Developer
(`amazon-q`), Kilo Code (`kilo-code`), Goose (`goose`) and Visual Studio (`visual-studio`, Windows).
Each config is backed up to `<file>.cw.bak` before the first change, only the ChromeWalk entry is touched,
and `connect` never writes a license key into any client config. Restart the client afterwards; some
clients show a one-time "trust this MCP server" prompt, which is the client's own gate.
Cloud chat apps (ChatGPT web, Gemini web, Grok) cannot reach a local process and are not supported.

## Never
- Do not create accounts or enter passwords on the user's behalf anywhere.
- Do not try to bypass licensing, edit license state files or fabricate a token.
