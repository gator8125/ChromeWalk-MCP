# Security Policy

ChromeWalk runs entirely on your own machine as a local MCP server: it never opens a
listening port to the network, never relays your browser traffic through a cloud service,
and the license/activation calls it makes are limited to chromewalk.com. See the
[Privacy Policy](https://chromewalk.com/privacy.html) for exactly what is (and is not)
sent.

## Reporting a vulnerability

Please **do not** open a public GitHub issue for a security vulnerability.

Instead, report it privately using GitHub's built-in advisory flow:
[github.com/gator8125/ChromeWalk-MCP/security/advisories/new](https://github.com/gator8125/ChromeWalk-MCP/security/advisories/new)

If that isn't available, use the contact channel listed on https://chromewalk.com
(the site's contact/support surface links from https://chromewalk.com/faq.html).

Please include:

- ChromeWalk version (`cw telemetry --json`, `"release"` field — there is no `cw --version`)
  and OS/platform
- Which client you were using it from (Claude Desktop, Claude Code, Codex, etc.)
- Steps to reproduce, and what you expected vs. observed
- Whether the issue requires a malicious/untrusted page or site to trigger, since
  ChromeWalk's primary job is fetching pages you don't control

## Scope

This repository is a **listing and install package only** — no ChromeWalk source code,
server code, or signing material lives here, so most application-level vulnerabilities
(if any) live in the compiled build distributed via GitHub Releases and the `.mcpb`
bundle, not in this repository's contents. In-scope for reports against this repository
specifically:

- A malicious or misleading `server.json` / `gemini-extension.json` / plugin manifest
  entry that could trick a client into running something other than the genuine
  ChromeWalk binary
- A supply-chain issue in how a release artifact referenced from this repo is fetched
  or verified (e.g. a broken or missing SHA-256 check)
- Anything in the install instructions that would lead a user to disable a safety
  mechanism (SmartScreen, Gatekeeper, hash verification) without good reason

Out of scope: findings against third-party sites ChromeWalk fetches, or vulnerabilities
requiring physical access to an already-compromised machine.

## Response

We aim to acknowledge reports promptly and to ship a fix (and, where relevant, a
CHANGELOG entry) before any public disclosure. There is currently no paid bug-bounty
program.
