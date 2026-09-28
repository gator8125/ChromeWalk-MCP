# Cursor community directory — listing text

This is the submission text for Cursor's community MCP directory (`cursor.directory`) and
for a README "Add to Cursor" badge, kept as a standalone doc so it can be copy-pasted into
either surface without pulling in the rest of this repo's README.

**Verification note:** Cursor's own docs (`cursor.com/docs/context/mcp/install-links`)
confirm the `cursor://anysphere.cursor-deeplink/mcp/install?name=...&config=...` deeplink
protocol and its base64-encoded-JSON `config` parameter. GitHub strips non-`http(s)` hrefs
from rendered Markdown, so a README badge cannot link to that `cursor://` URL directly;
`https://cursor.com/install-mcp?name=...&config=...` is Cursor's own HTTPS redirector that
accepts the same `name`/`config` parameters and forwards into the install flow (confirmed
live: fetching it returns an "Installing MCP Server... Redirecting to Cursor..." page) — use
that form for any clickable link on GitHub or elsewhere HTTPS is required. The "Add to
Cursor" SVG badge asset URLs (`cursor.com/deeplink/mcp-install-{dark,light}.svg`) were not
shown inline on Cursor's docs page as fetched; render the badge and click through once
before publishing to confirm the asset still resolves. Community directory submission is
done by opening a PR against Cursor's directory repo/site, not via an API in these docs;
check cursor.directory's own contribution instructions at submission time in case the
process has changed.

## Listing text

**Name:** ChromeWalk

**One-line description:** Real local browser automation - WAF-safe fetch, full-evidence
capture, e-commerce harvesting, and cross-engine QA, all running on your own machine.

**Category:** Browser automation / Web scraping / Testing

**Longer description:**

> ChromeWalk gives your AI agent a private, standalone Chrome-for-Testing browser it drives
> locally - no cloud, no tunnel, and it never touches your personal Chrome profile. It fetches
> pages behind WAFs and JS challenges, captures full network + screenshot evidence, harvests
> e-commerce catalogs into structured data, inspects performance/accessibility, and runs
> cross-engine (Chromium/WebKit/Firefox) QA comparisons. Requires a license key from
> chromewalk.com (no keyless trial mode); the key is tied to your email.

**Repository:** https://github.com/gator8125/ChromeWalk-MCP
**Homepage:** https://chromewalk.com
**Docs:** https://chromewalk.com/docs/

## Deeplink (protocol form, for a local shell / browser address bar)

```
cursor://anysphere.cursor-deeplink/mcp/install?name=chromewalk&config=eyJjb21tYW5kIjoiY2hyb21ld2FsayIsImFyZ3MiOlsibWNwIl19
```

The `config` value is the base64 encoding of the compact JSON
`{"command":"chromewalk","args":["mcp"]}` (no spaces — `JSON.stringify`-style, not
`json.dumps`'s default spacing), matching the format documented at
`cursor.com/docs/context/mcp/install-links`. `chromewalk` (the compiled binary the installer
puts on PATH) is a real executable on every platform, so this same config works unchanged on
Windows and macOS/Linux - unlike `cw`/`cw.cmd` (a terminal convenience shim), which
Cursor's no-shell process launcher cannot execute on Windows.

## README badge markdown (HTTPS install-link form, not the `cursor://` deeplink)

```markdown
[![Add ChromeWalk to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=chromewalk&config=eyJjb21tYW5kIjoiY2hyb21ld2FsayIsImFyZ3MiOlsibWNwIl19)
```

GitHub strips non-`http(s)` hrefs from rendered Markdown, so a `cursor://...` link target
would silently be dropped from the badge on the README page — use the `https://cursor.com/
install-mcp?...` redirector instead (same `name`/`config` query parameters, confirmed live
to forward into Cursor's install flow). Swap `mcp-install-dark.svg` for
`mcp-install-light.svg` for a light-background README. The `config` value here decodes to
`{"command":"chromewalk","args":["mcp"]}` (the same launch form on every platform); render
the badge and click through once before publishing to confirm the asset still resolves and
the deeplink opens Cursor's install dialog with that command filled in.

## Manual `mcp.json` entry (fallback / non-deeplink)

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
