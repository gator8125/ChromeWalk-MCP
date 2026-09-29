# ChromeWalk (Gemini CLI extension context)

ChromeWalk gives you a real, local browser you can drive to fetch pages (including
WAF/JS-challenge-protected ones), capture full network + screenshot evidence, harvest
e-commerce catalogs into structured data, inspect performance/accessibility, and run
cross-engine (Chromium/WebKit/Firefox) QA comparisons — all executed on the user's own
machine, never in the cloud.

## Requirements

- ChromeWalk must be installed separately and licensed — this extension does not bundle
  the binary or any source. See https://chromewalk.com/download.html and
  https://chromewalk.com/register.html.
- The `cw` command must be on `PATH`. The Windows/macOS/Linux one-line installers
  (`install.ps1` / `install.sh`) add it automatically; if you installed a different way,
  make sure `cw help` works in a fresh shell before using this extension (there is no
  `cw --version` command; `cw help` is a good no-args sanity check, or use
  `cw telemetry --json` and read its `"release"` field for the installed version number).
- A valid `CHROMEWALK_LICENSE` (env var, read when the MCP server starts) or a prior
  `cw activate <key>` on this machine. Without an activated license every tool call
  returns a "license required" error.

## Known packaging tradeoff

This GitHub repository intentionally ships **no ChromeWalk source code or binary** —
only listing/install metadata (see the repo root `README.md`). That means
`gemini-extension.json`'s `mcpServers.chromewalk` entry launches via the user's `PATH`
rather than an absolute path into a bundled venv or compiled binary the way a
source-carrying extension normally would. The command is `chromewalk` with `args: ["mcp"]`
— **not** `cw` — on every platform: `chromewalk` is the real compiled binary the installer
puts on PATH, while `cw`/`cw.cmd` is a terminal-only convenience shim. On Windows in
particular, a bare `cw` resolves only to `cw.cmd`, and Gemini CLI (like most Node-based MCP
clients) launches local servers with `child_process.spawn` *without* a shell, which cannot
execute a `.cmd`/`.bat` file directly and fails with `ENOENT` even though typing `cw`
yourself in a terminal works fine; `chromewalk.exe` is a real executable, so this is a
non-issue there, and the same `command: "chromewalk", args: ["mcp"]` form works unchanged
on macOS/Linux. The remaining tradeoff:

- **Pro:** the extension works across Windows/macOS/Linux and across every ChromeWalk
  install location, with no per-machine absolute path baked into this manifest.
- **Con:** if `cw` is not on `PATH` when Gemini CLI starts, the extension will fail to
  launch the MCP server with a "command not found"-style error rather than a
  ChromeWalk-specific error message, and there is no way for this manifest alone to
  detect or explain that to the user ahead of time.

The more robust fix is to run `cw connect gemini-cli` (supported by `scripts/connect.py`)
after installing ChromeWalk instead of relying on this extension's static manifest — it
writes an absolute, verified path for your specific install. `cw connect` (no argument)
does the same for every supported client in one pass.

## Tools

ChromeWalk exposes 26 MCP tools once running (fetch, screenshot, render, webinspect,
pwa, notify, jsonquery, api, flow, dump, drive, smoke, harvest, facets, xengine,
resources, sites_list, logs, doctor, procs, schema, cms, config, compare, calls,
cancel). The diagnostic tools (`doctor`, `procs`, `config`, `calls`, `cancel`) work
without a license. Full reference: https://chromewalk.com/docs/tool-fetch.html
(and the sibling `tool-*.html` pages under https://chromewalk.com/docs/).
