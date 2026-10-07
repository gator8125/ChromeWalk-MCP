---
name: chromewalk-cross-engine
description: Test a web app in Chromium, Firefox and WebKit (Safari's engine) with ChromeWalk's xengine, including the screens behind a login (--session saved with warm, --auth, --local-storage-file). Use when asked whether a site works in Safari or Firefox, to find browser-specific bugs, to compare engines, or to test a logged-in app across browsers.
---

# ChromeWalk cross-engine testing (logged in)

`xengine` runs the SAME suite file that `smoke` uses in real Chromium, Firefox and WebKit (via
Playwright) and reports per page and engine pass/fail plus a `divergences[]` list: which engine
differed on which check. Since 13.0 it can log in on every engine, so the screens behind a login are
covered too. MCP tool: `chromewalk_xengine`; CLI: `chromewalk xengine`.

Honest limit: WebKit here is Safari's engine, not the Safari app. It finds layout, CSS and JavaScript
differences, not Safari-app, ITP or iPhone-specific behaviour.

## Prerequisites
- A license (see `chromewalk-onboarding`); exit 4 otherwise.
- The engines installed: `chromewalk setup` (installs Chromium, Firefox, WebKit and runs `doctor`).
  Exit 5 `engine_missing` means one is not installed.

## Public pages
```
chromewalk xengine suite.json --json
chromewalk xengine --url https://app.example.com/ --engines chromium,webkit --json
```
- `--engines chromium,firefox,webkit` (default: all three). Three engines take about three times as long
  as one; run cross-engine on changed pages or nightly, not on every commit.
- `--render` also runs a render gate: each engine's screenshot plus layout-landmark geometry, flagging
  STRUCTURAL divergence (an element in one engine but not another, geometry deltas over
  `--geo-threshold` px, default 4). Pixel diff is informational. It also writes `report.junit.xml`.
- `--fail-on-divergence` makes any functional divergence between engines exit 3. Without it,
  divergences are reported but only real failures gate.
- `--self-test` proves every detector fires in every engine.

## Logged-in pages: pick the login input
| Input | Use when |
|---|---|
| `--session NAME` | cookie-based apps: a login saved once with `chromewalk warm <login-url> --save-session NAME` |
| `--auth "k=v;k2=v2"` | apps that log in from a token in localStorage; `${ENV_VAR}` is expanded |
| `--local-storage-file FILE` | same, from a JSON `{key: value}` file (keeps tokens out of the suite) |
| suite `"local_storage": {...}` | same, per suite or per page |

localStorage seeds are written before any page script runs and ONLY on the page's own origin, on every
engine. A session applies its cookies to every engine.

### Saving a session (a human step)
`warm` opens a visible window; the user signs in (solving any MFA or challenge themselves), then it saves
the login:
```
chromewalk warm https://staging.example.com/login --save-session acct
chromewalk xengine suite.json --session acct --json
```
Do not type the user's password for them and never try to defeat a CAPTCHA or MFA prompt. If the
session file is missing the run exits 2 `missing_session`; unreadable is `bad_session`. Tell the user to
re-run `warm`. `--session-domain REGEX` limits which cookie domains are saved.

### Token apps
```
chromewalk xengine suite.json --auth "tp_key=${TP_KEY}" --json
```
Values are never logged, never written to results (the result lists only the seeded key NAMES and whether
a session was used), and via MCP they travel in a private args file, never on a command line.

## Steps, explore and coverage run in every engine
The suite's `steps` (with `expect`), `assert` blocks, and:
```
chromewalk xengine suite.json --session acct --explore 15 --coverage --json
```
`--explore N` clicks up to N safe controls per page in each engine (never delete, log out, pay, send,
submit, form submits, downloads, new-tab or other-site links; `--explore-allow REGEX` overrides labels
only on explicit request). `--coverage` reports controls found / scripted / explored per page AND engine.
Use staging or a test account.

## Reading the result
- One result per page x engine; engine versions are recorded.
- `divergences[]`: a check that passed in one engine and failed in another. A unit failing in every
  engine is a real site bug, not an engine difference.
- Console / page errors from a THIRD-PARTY frame (an iframe on another site) are warnings with code
  `third_party_frame`, not findings.
- Pre-capture hooks work as in smoke: `--wait-for-selector`, `--wait-for-network-idle=800`, `--eval`,
  `--set-attr html:data-theme=dark`, `--ignore-console REGEX`, `--config FILE`.

| Exit | Meaning |
|---|---|
| 0 | success |
| 3 | the gate failed: functional failures, render gate, or divergence with `--fail-on-divergence` |
| 5 | an engine could not be launched and nothing failed for real: retry, then `chromewalk-troubleshooting` |
| 4 | license |
| 2 | usage (`bad_args`, `bad_suite`, `missing_session`, `bad_session`, `bad_local_storage_file`, `bad_prehook`) |
| 6 | MCP only: timed out / cancelled (default budget 900 s) |

## Typical sequence for "does it work in Safari?"
1. `chromewalk smoke suite.json --base-url <staging> --json` - the Chromium baseline.
2. `chromewalk xengine suite.json --engines chromium,webkit,firefox --json` (add `--session` / `--auth`
   if the pages are behind a login).
3. Report divergences by engine and check; add `--render` when the symptom is layout.
