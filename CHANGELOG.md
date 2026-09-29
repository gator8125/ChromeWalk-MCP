# Changelog

Release notes for ChromeWalk live with each [GitHub Release](https://github.com/gator8125/ChromeWalk-MCP/releases) —
every tagged release (`vX.Y.Z`) has its own notes describing what changed, alongside the
downloadable per-platform `.mcpb` bundle(s) (e.g. `chromewalk-11.0.1-win32-x64.mcpb` for
Windows x64) and platform installers.

This repository is a listing/install package, not the ChromeWalk source tree, so it does not
carry the full internal development changelog (which tracks build tooling, internal
refactors, and packaging work across many pre-release forks). What you get here is:

- The current `server.json` / `gemini-extension.json` / plugin manifests, versioned to match
  the latest release.
- The **user-facing** highlights for that release, copied into each GitHub Release's notes.

## 11.0.1

**Hotfix, recommended for all 11.0.0 users.**
- Fixes pages that never load: smoke reporting "timeout: document never became interactive", and
  fetch ending on about:blank. This was most visible when the ChromeWalk data folder is on a slower
  (HDD) disk. Navigations Chrome cancels while a fresh profile is still starting are now retried.
- Removes a redundant cookie-clear step that stalled each browser launch by up to 30 s.
- `fetch` no longer crashes when printing page text that the Windows console can't display.
- The documented opt-out `CW_TELEMETRY=off` now actually turns telemetry off. Before, only
  `CHROMEWALK_NO_TELEMETRY` / `DO_NOT_TRACK` worked.
- Telemetry and `update --check` report the full version (for example 11.0.1), so an installed
  hotfix is recognised as up to date.

## 11.0.0

**Design-drift release.** Answers "did the UI/UX drift?" across pages, templates, sites and
time, even when the CSS itself is different.

- New tool: `design` - a site-wide UI/UX audit over a URL list or a sitemap: a design-token
  inventory (palette with near-duplicate colour clusters, type scale, spacing grid, radii,
  shadows, breakpoints, CSS custom properties that differ between pages), conformance against
  your own design-token file (W3C Design Tokens / Style Dictionary), component variant clusters
  with a gallery of element crops, UX heuristic failures (WCAG 2.2 contrast incl. dark mode, tap
  targets, visible focus, overflow, layout shift, above-the-fold call to action, alt text,
  heading order), drift against a saved baseline, and a score per page. HTML and JUnit reports;
  `fail_on` gates the exit code (error, warn, drift or none).
- `compare` gains opt-in design layers - tokens, components, css, ux and perceptual (SSIM
  heatmap) - with `layers=design` as the preset, plus the capture options `css`, `coverage`,
  `states` (forced hover/focus/active/disabled) and `color_schemes` (dark mode).
- Smoke suites gain a page check `{"design": {"baseline": NAME, "fail_on": "error"}}`.
- Snapshots are schema version 2 (additive: version 1 snapshots and baselines keep working).

## 10.0.0

**Reliability release.** MCP calls now run concurrently instead of serially; a browser-layer
overhaul (pooled profiles, launch slots, an orphan reaper) plus one error/exit-code contract
for every tool close out the field reports about failed launches, orphaned browser processes,
and hung calls wedging the server.

- MCP calls run concurrently: a hung or cancelled call no longer blocks the calls behind it, and
  its whole browser process tree is always cleaned up. 8 pipelined calls that used to run fully
  serial (p50 24.4s, max 47.4s) now overlap (p50 8.9s, max 15.1s); a forced tool timeout that
  used to wedge the server with 10-14 browser processes left now returns a clean error in 5-10s
  with none left behind.
- New tools: `compare` (page/baseline diff across visual, DOM, text, network, schema, SEO and
  more layers), `doctor` (read-only environment health check), `procs` (browser-process census
  with orphan cleanup), `schema` (structured-data validation/diff), `cms` (platform fingerprint),
  and `config` (view every tuning setting and its source). The diagnostics work without a
  license.
- One exit-code contract for every tool, CLI and MCP: `0` ok, `2` usage error, `3` checked
  failure (still a valid result), `4` license required, `5` infrastructure failure, `6` timed
  out or cancelled.
- An unreachable page is now reported as a real failure (`3`, not a false success against the
  browser's own error page), and navigation now actually honors your timeout.
- Security hardening: closed a license-bypass path in the compiled build's `help` command;
  compiled-build endpoint overrides now only take effect on loopback addresses; the signature
  verifier runs a startup self-test before it is trusted.

For what's new in the current release, see the latest entry at
https://github.com/gator8125/ChromeWalk-MCP/releases/latest.
