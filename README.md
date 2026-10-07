# ChromeWalk

**A real, local browser for your AI.** ChromeWalk gives coding and research agents a
private, standalone Chrome-for-Testing instance they drive directly on your own machine —
no cloud relay, no tunnel, never your personal Chrome profile. It fetches WAF-protected
pages, captures full network/screenshot evidence, harvests e-commerce catalogs, runs
performance/accessibility inspection, and drives cross-engine QA (Chromium/WebKit/Firefox),
all exposed to your AI client as standard [MCP](https://modelcontextprotocol.io) tools.

This repository is the **public listing and install package** for ChromeWalk. It does not
contain ChromeWalk's source code, its signing keys, or its server code — those stay in the
private engineering repository. What's here is everything a client (Claude Desktop, Claude
Code, Codex, Cursor, VS Code, Gemini CLI, Windsurf/Cline/Continue, ...) needs to discover and
install the compiled, licensed ChromeWalk build, plus the packaging manifests for each
distribution channel (MCP Registry, Gemini CLI extension, Claude Code plugin, Cursor
directory).

- Website: https://chromewalk.com
- Docs: https://chromewalk.com/docs/
- What is ChromeWalk: https://chromewalk.com/docs/what-is-chromewalk.html
- Privacy: https://chromewalk.com/privacy.html
- Terms: https://chromewalk.com/terms.html
- Support / FAQ: https://chromewalk.com/faq.html
- License terms: [`LICENSE`](./LICENSE) (full EULA also at https://chromewalk.com/LICENSE.txt)

---

## License — a key is required

ChromeWalk is **proprietary software and requires a license**. There is no
anonymous or keyless trial mode in the shipped default build: until a license key is
activated on the machine, every ChromeWalk tool call returns a "license required" error.
(The offline grace window applies only to an already-activated license that temporarily
cannot reach the license server - it is not a keyless trial.)

1. Sign up at **https://chromewalk.com/register.html** with your email - or, from 13.0, just ask your
   AI assistant to sign you up: the `register` tool emails your key, and the `activate` tool activates
   it, both inside the chat and without a license. Your license key is tied to that email address.
2. Install ChromeWalk for your client (below).
3. Activate with either:
   - the `CHROMEWALK_LICENSE` environment variable (read once when the MCP server starts;
     it activates the device if it is not already activated), or
   - `cw activate <key>` from a terminal after install, or
   - pasting the key into the client's own extension/plugin settings UI (Claude Desktop
     `.mcpb` exposes a `license_key` field in `user_config`).

ChromeWalk does **not** collect page content, option values, file paths, or credentials. Its usage
telemetry is required by the license (there is no opt-out); from 12.0 each tool run reports the
target URL (scheme, host and path - credentials, query strings and fragments are removed on your
computer first) and its domain. Your license key itself is never sent in telemetry, only a SHA-256
hash of it (`license_hash`), which links the install to your account. See the
[Privacy Policy](https://chromewalk.com/privacy.html) for exactly what is collected.

---

## What's new in 13.0

**Test the flows, not just the pages.** 35 MCP tools.

- **Sign up and activate from your AI assistant** (`register`, `activate`) - no terminal needed. The
  Claude Desktop extension's license key field is optional.
- **Activate once per computer** - `cw activate <key> --remember` keeps the key in Windows Credential
  Manager and every other install for your user activates itself; `CHROMEWALK_LICENSE_FILE` for CI.
- **Click-flow coverage in smoke** - every scripted step can carry an `expect`, each step is checked for the
  errors and failed requests it caused (a failed step now fails the page), `explore` clicks the safe
  controls you never scripted (never delete / log out / pay / submit), and `coverage` reports what was
  exercised.
- **Logged-in testing on Firefox and WebKit** - `xengine` and `render` take `session`, `auth` and
  `local_storage_file`, and `xengine` runs the suite's steps, checks and Explore on every engine.
- **Microsoft Edge and Brave** - `browser: "edge" | "brave" | "chrome"` (or a path) on smoke, fetch,
  dump, screenshot, compare, design, audit and webinspect, always with ChromeWalk's own private profile.
- **Reliability** - a killed run no longer leaves Chrome behind; large design / audit runs are batched
  and resumable (`resume`); runs report where their time went; `doctor` shows the install folders and
  explains slow starts.
- **More AI tools in one command** - `cw connect` adds GitHub Copilot CLI, Kiro, Amazon Q Developer,
  Kilo Code, Goose and Visual Studio.
- **Skills** for Claude Code, Cursor and Codex in [`skills/`](./skills/) - copy them into your agent's
  skills folder.

---

## What's new in 12.0

- **`audit`** (one-pass UI-uniformity audit bundle) and **`baseline`** (per-page and per-selector visual
  regression with an anti-clobber gate) - 33 MCP tools.
- **Smoke**: slow is not failed, failure artifact bundles, changed-pages-only runs, a base-URL override,
  fewer infrastructure flakes.
- **Pre-capture hooks and browser options** on every capture tool (waits, scripts, attributes, console
  noise filters; language, time zone, throttling, reduced motion, `auth`).
- **Headless by default**, a **self-updating Windows app** (signed, self-tested, automatic rollback), and
  end-to-end recipes in the docs. Full notes: [CHANGELOG.md](./CHANGELOG.md).

---

## What's new in 10.0

**Reliability release.** MCP calls now run concurrently instead of serially, the browser layer
cleans up after itself even when a call is cancelled or hangs, and every tool (CLI and MCP)
shares one error/exit-code contract. Under multi-agent load (same machine, both licensed, 3
reps per scenario):

| Scenario | 9.0 | 10.0 |
|---|---|---|
| 8 pipelined MCP calls | fully serial; p50 24.4s, max 47.4s | up to 6 browsers at once; p50 8.9s, max 15.1s |
| 2 servers x (4 calls + 1 hang) | the hung call blocks every call behind it; up to 3 browser processes left after exit | other calls unaffected; every cancel honoured; 0 browser processes left |
| a forced tool timeout | server wedged, nothing answered within 120s; 10-14 browser processes left | a structured timeout error in 5-10s; 0 browser processes left |
| 12 parallel smoke runs on a saturated CPU | 3 launch failures | 0 launch failures (about 59% slower wall time — launch slots are throttled above 90% CPU) |

- **New `compare` tool** — page-vs-page, directory, or saved-baseline diffs across visual, DOM,
  text, network, structured-data, SEO, links, styles, accessibility, console, headers and
  performance layers, with noise calibration across repeated captures.
- **New diagnostics** — `doctor` (read-only environment health check), `procs` (census of
  ChromeWalk's own browser process trees, with orphan cleanup), `schema` (JSON-LD / microdata /
  RDFa / OpenGraph validation and diff), `cms` (platform fingerprint), and `config` (every
  tuning key, its effective value, and where it came from). The diagnostics work **without** a
  license, so they're how you debug a broken license or install.
- **Tunable configuration** — `chromewalk config show|get|set|unset` covers worker/agent counts,
  launch retries and backoff, connect/command timeouts, the circuit-breaker threshold, profile
  mode, default resource blocking, and more (flag > environment variable > config file > auto).
- **One exit-code contract everywhere:**

  | code | meaning |
  |---|---|
  | 0 | ok |
  | 2 | usage / bad arguments |
  | 3 | checked failure — the tool ran and the target failed a check (a diff, a missing required field, a page that never loaded); still a valid result |
  | 4 | license required |
  | 5 | infrastructure failure — browser launch/connect failed after retries, or a dependency is missing; retry, or run the `doctor` tool |
  | 6 | timed out or cancelled |
  | 1 | unexpected internal error |

- An unreachable page is now reported as a real failure (exit `3`) instead of a false "success"
  against the browser's own error page, and navigation now actually stops at your timeout.

## Tools

ChromeWalk exposes 35 MCP tools once installed and licensed. The sign-up tools (`register`,
`activate`) and the diagnostic tools (`doctor`, `procs`, `config`, `reset`, `recommend`, `calls`, `cancel`)
work without a license — they're how you get a key and check a broken install or license state.

- `register` *(no license required)* — sign up: the license key is emailed to the address you give (never shown in the chat)
- `activate` *(no license required)* — activate this install with your key (the key is never echoed back)

- `fetch` — WAF-bypass fetch through a real browser: status, title, text, validation, optional raw response bodies
- `screenshot` — screenshot or walk one URL or a list of URLs
- `render` — cross-engine render (Chromium/WebKit/Firefox) with pixel diff / A-B / golden baselines
- `webinspect` — console errors, HAR capture, Core Web Vitals, a light accessibility pass
- `pwa` — manifest / service worker / offline reload / HTTPS / installability checks
- `notify` — Web Notifications permission grant/deny plus captured notifications
- `jsonquery` — query/flatten JSON or YMM trees, CSV export (no browser needed)
- `api` — parallel API suite runner with adaptive anti-spam back-off
- `flow` — scripted SPA flow (goto/click/type/wait_for/capture) to reach auth/role-gated views
- `dump` — full-evidence capture: every response, raw bodies, and screenshots, multi-agent, memory-governed
- `drive` — drive a running browser window by CDP: navigate/read/eval/screenshot, or ask a chat app
- `smoke` — concurrent self-verifying smoke gate (page x viewport units, layout detectors, steps/asserts)
- `harvest` — e-commerce harvester: normalized products (sku/price/availability/fitment/metafields)
- `facets` — store-level filter/fitment axes (metafield keys, option values, axis-tree JSON assets)
- `xengine` — the smoke-detector library run across real Chromium/Firefox/WebKit with a divergence report
- `resources` — live RAM/CPU/pagefile plus a memory-aware recommended worker count (no browser)
- `sites_list` — list the configured named sites
- `logs` — read the shared event log: summary / errors / tail
- `doctor` *(no license required)* — read-only environment health check (dependencies, engines, browser launch/CDP, orphans, config)
- `procs` *(no license required)* — census of ChromeWalk's own browser process trees with an orphan verdict; can reap verified orphans
- `schema` — structured data (JSON-LD, microdata, RDFa, OpenGraph) plus Google required/recommended validation and diff
- `cms` — CMS/platform fingerprint with evidence
- `config` *(no license required)* — show the tuning config (value + source per key), or get one key
- `compare` — page vs page / snapshot / baseline diff across visual, DOM, text, network, schema, SEO, and more layers, plus the 11.0 design-drift layers (design tokens, components, CSS rules, UX heuristics, perceptual similarity)
- `design` — site-wide UI/UX design audit: design-token inventory with swatches and type scale, design-system conformance, component variant clusters with element crops, UX heuristic failures (contrast, tap targets, focus visibility, overflow, layout shift, above-the-fold CTA), drift against a saved baseline, and a score per page
- `audit` — one-pass UI-uniformity audit bundle (audit.md + audit.json + shots/): uniformity ranking against a reference page, idiom census, element crops and a component sheet, design-token compliance per selector, responsive deltas, light/dark themes
- `baseline` — stored-baseline visual regression per page and per selector (SSIM), with an anti-clobber gate for the pages a change should not have touched
- `probe` — no-browser preflight of a URL: DNS/TCP/TLS, status, WAF/challenge, login wall, render class and a recommended strategy
- `targets` — what ChromeWalk learned per host (wait strategy, timeouts, verdicts); show or forget
- `reset` *(no license required)* — soft recovery: orphaned browsers, stale locks, the profile pool
- `recommend` *(no license required)* — file a gap / bug report locally (secrets scrubbed)
- `calls` *(no license required)* — this server's in-flight and recent calls, or one call's state/result
- `cancel` *(no license required)* — cancel an in-flight call; its whole browser process tree is killed

Full per-tool reference: https://chromewalk.com/docs/tool-fetch.html (and the sibling
`tool-*.html` pages under https://chromewalk.com/docs/).

---

## Install the ChromeWalk binary (once, per machine)

Every client below talks to one local ChromeWalk install. Install it first:

**Windows** — one-line installer (downloads the release bundle, verifies its SHA-256, and
installs the compiled `chromewalk.exe` — no Python source, no venv to build — plus a `cw.cmd`
terminal shim, to `%LOCALAPPDATA%\Programs\ChromeWalk`, added to your **user** PATH; no admin
required):

```powershell
powershell -c "irm https://chromewalk.com/install.ps1 | iex"
```

or download the installer directly: https://chromewalk.com/download.html
(`ChromeWalk-Setup.exe`, built with Inno Setup). **Note:** the code-signing certificate for
this installer is still pending, so until a signed build is published, Windows SmartScreen
may warn ("Windows protected your PC") the first time you run it — click "More info" ->
"Run anyway" to proceed, or verify the SHA-256 checksum published alongside the download
first if you'd rather not click through the warning.

**macOS** — installs the compiled `chromewalk` binary to `/usr/local/bin`, plus a `cw` shim:

```bash
curl -fsSL https://chromewalk.com/install.sh | bash
```

or download the `.pkg` GUI installer from https://chromewalk.com/download.html once it is
published (see `deploy/installer/macos/` in the engineering repo — **this macOS packaging is
untested**: no CI run or real Mac has confirmed it yet, treat it as best-effort until a
validated release says otherwise).

**Linux** — `install.sh` above also works on Linux, but there is no compiled Linux binary or
packaged installer yet; it falls back to a source install.

After install, run `cw connect` from a terminal — it detects every supported client already
installed on your machine and registers ChromeWalk in each one automatically (idempotent,
config-backed-up). The per-client steps below are the manual equivalent, and what `cw
connect` does under the hood for each one.

---

## Per-client install

> **Read this first.** For every client below, the **recommended** step is to run
> `cw connect` (or `cw connect <client-name>`, e.g. `cw connect codex`) — it detects each
> client already installed on your machine and writes the correct config for you
> (idempotent, backs up the existing config first).
>
> If you'd rather hand-edit a config, the launch command is the same shape on every platform:
> **command `chromewalk`, args `["mcp"]`** — never `cw`, and never through a shell (`cmd /c`
> or otherwise). `cw`/`cw.cmd` is a convenience shim for typing at a terminal; on Windows it
> is a `.cmd` file, which `child_process.spawn`/Rust's `Command` (what MCP clients actually
> use to launch a local server) cannot execute directly without a shell — they fail with
> `ENOENT` even though typing `cw` yourself works fine. `chromewalk` is a real executable
> (`chromewalk.exe` on Windows, `chromewalk` on macOS) with no such limitation, which is why
> every config below uses it, identically on Windows and macOS/Linux.
>
> The installer puts `chromewalk` (and the `cw` shim) on your **user PATH**, so a bare
> `"command": "chromewalk"` resolves correctly once you've opened a new terminal/restarted the
> client after installing. If a client can't find it on PATH (or you want a config that
> doesn't depend on PATH at all), use the absolute install path instead:
> - Windows: `%LOCALAPPDATA%\Programs\ChromeWalk\chromewalk.exe`
> - macOS: `/usr/local/bin/chromewalk`

### Claude Desktop (.mcpb bundle)

1. Download the bundle for your platform from the latest
   [GitHub Release](https://github.com/gator8125/ChromeWalk-MCP/releases). Today that's
   **`chromewalk-13.0.0-win32-x64.mcpb`** (Windows x64) — the only bundle actually built and
   published. macOS (`darwin-arm64` / `darwin-x64`) bundles are planned (see
   `.github/workflows/mcpb-binary.yml`, currently manual-dispatch-only and unvalidated) but
   are **not yet published**; there is no Linux `.mcpb` bundle.
2. Double-click it, or drag it onto Claude Desktop's Settings → Extensions panel.
3. Enter your license key in the extension's settings (`license_key` field, optional), run
   `cw activate <key>` if you also installed the `cw` CLI, or leave it empty and ask Claude to sign
   you up (the `register` / `activate` tools).

Full walkthrough: https://chromewalk.com/docs/install-claude-desktop.html

### Claude Code

Recommended — install the plugin from this repository's marketplace:

```
/plugin marketplace add gator8125/ChromeWalk-MCP
/plugin install chromewalk@chromewalk
```

or let `cw connect claude-code` register it for you — this runs
`claude mcp add chromewalk -- <path-to-chromewalk> mcp` with the absolute installed binary
path, avoiding the `cw.cmd` problem described above. If you register it by hand instead, do
the same:

```
claude mcp add chromewalk -- chromewalk mcp
```

A project-scoped `.mcp.json` you can copy into your own repo is included at
[`/.mcp.json`](./.mcp.json) (uses the `chromewalk mcp` form, same on every platform).

Full walkthrough: https://chromewalk.com/docs/install-claude-code.html

### OpenAI Codex CLI

Recommended: `cw connect codex` (backs up your existing `config.toml` first). To add it by
hand, add to `~/.codex/config.toml`:

```toml
[mcp_servers.chromewalk]
command = 'chromewalk'
args = ['mcp']
startup_timeout_sec = 30
tool_timeout_sec = 1200
```

(Same on Windows and macOS/Linux — no `cmd /c` wrapper needed; use the absolute path,
e.g. `'C:\Users\<you>\AppData\Local\Programs\ChromeWalk\chromewalk.exe'` or
`'/usr/local/bin/chromewalk'`, if `chromewalk` isn't resolving from PATH.)
`startup_timeout_sec = 30` and `tool_timeout_sec = 1200` are intentional, not defaults:
Codex's own defaults (10s startup / 60s per tool call) are too short for ChromeWalk's
real-browser tools, which can legitimately run for several minutes on a heavy fetch/harvest.

Full walkthrough: https://chromewalk.com/docs/install-codex.html

### Cursor

Click **Add to Cursor**, or add the block below to `~/.cursor/mcp.json` / your project's
`.cursor/mcp.json` by hand:

[![Add ChromeWalk to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=chromewalk&config=eyJjb21tYW5kIjoiY2hyb21ld2FsayIsImFyZ3MiOlsibWNwIl19)

(The badge links through `https://cursor.com/install-mcp?...`, Cursor's own HTTPS
install-link redirector, rather than the `cursor://...` deeplink form directly — GitHub
strips non-`http(s)` link targets from rendered Markdown, so a bare `cursor://` href would
be dropped from this page. See `cursor.com/docs/context/mcp/install-links` for the deeplink
format the redirector wraps.)

```json
{
  "mcpServers": {
    "chromewalk": {
      "command": "chromewalk",
      "args": ["mcp"]
    }
  }
}
```

(Same on Windows and macOS/Linux. If `chromewalk` isn't resolving from PATH, use the
absolute install path instead: `%LOCALAPPDATA%\Programs\ChromeWalk\chromewalk.exe` on
Windows, `/usr/local/bin/chromewalk` on macOS.)

Community directory listing text and submission notes: [`docs/cursor-directory-listing.md`](./docs/cursor-directory-listing.md).

Full walkthrough: https://chromewalk.com/docs/install-cursor.html

### VS Code / GitHub Copilot (MCP)

Add to your user or workspace `mcp.json` (`servers` key, not `mcpServers` — VS Code's schema
differs from Claude's):

```json
{
  "servers": {
    "chromewalk": {
      "type": "stdio",
      "command": "chromewalk",
      "args": ["mcp"]
    }
  }
}
```

(Same on Windows and macOS/Linux. If `chromewalk` isn't resolving from PATH, use the
absolute install path instead: `%LOCALAPPDATA%\Programs\ChromeWalk\chromewalk.exe` on
Windows, `/usr/local/bin/chromewalk` on macOS.)

Full walkthrough: https://chromewalk.com/docs/install-vscode.html

### Gemini CLI

Install the extension from this repository:

```
gemini extensions install https://github.com/gator8125/ChromeWalk-MCP
```

This uses [`gemini-extension.json`](./gemini-extension.json) and [`GEMINI.md`](./GEMINI.md)
at the repo root. This repo ships no source or binary, so the extension's `mcpServers` entry
needs `chromewalk` (installed per the steps above) already on `PATH` — it uses the
`command: "chromewalk", args: ["mcp"]` form described above, identically on every platform,
so it launches correctly under Gemini CLI's own no-shell process spawning. Run `cw connect
gemini-cli` after install to have it verified/re-written for your machine, or run
`cw connect` to cover every detected client at once.

Full walkthrough: https://chromewalk.com/docs/install-gemini-cli.html

### Windsurf / Cline / Continue

All three read a `mcpServers` block similar to Claude Desktop's. Add:

```json
{
  "mcpServers": {
    "chromewalk": {
      "command": "chromewalk",
      "args": ["mcp"]
    }
  }
}
```

(Same on Windows and macOS/Linux. If `chromewalk` isn't resolving from PATH, use the
absolute install path instead: `%LOCALAPPDATA%\Programs\ChromeWalk\chromewalk.exe` on
Windows, `/usr/local/bin/chromewalk` on macOS.)

- Windsurf: `~/.codeium/windsurf/mcp_config.json` — https://chromewalk.com/docs/install-windsurf.html
- Cline: VS Code global storage `.../saoudrizwan.claude-dev/settings/cline_mcp_settings.json` — https://chromewalk.com/docs/install-cline.html
- Continue: `~/.continue/config.json` — https://chromewalk.com/docs/install-continue.html

### Any other MCP-capable client

Generic stdio launch command: `chromewalk mcp` (not `cw` — see the note above), identically
on Windows and macOS/Linux, no shell involved. This is exactly what `cw connect` writes for
JSON-config clients. See https://chromewalk.com/docs/install-generic-mcp.html.

---

## What's in this repository

```
README.md                    - this file
LICENSE                      - proprietary license notice (see chromewalk.com/LICENSE.txt for full EULA)
SECURITY.md                  - how to report a vulnerability
CHANGELOG.md                 - pointer to the release changelog
server.json                  - MCP Registry server descriptor
gemini-extension.json        - Gemini CLI extension manifest
GEMINI.md                    - context file loaded by the Gemini CLI extension
.mcp.json                    - example Claude Code project-scope MCP config
.claude-plugin/plugin.json       - Claude Code plugin manifest
.claude-plugin/marketplace.json  - Claude Code marketplace listing (this repo, one plugin)
.github/ISSUE_TEMPLATE/      - bug report / feature request templates
docs/cursor-directory-listing.md - Cursor community-directory submission text
```

No `.py`, no `config.php`, no signing material, and no server code live in this repository.
The compiled `.mcpb` bundle and platform installers are attached to
[GitHub Releases](https://github.com/gator8125/ChromeWalk-MCP/releases); the license/activation
server is not part of this repo at all.

## Support

- Docs: https://chromewalk.com/docs/
- FAQ: https://chromewalk.com/faq.html
- Issues: use this repository's [issue tracker](https://github.com/gator8125/ChromeWalk-MCP/issues)
  for install/packaging problems. For license/account issues, use the contact info in the
  Privacy Policy / Terms of Use at https://chromewalk.com.
