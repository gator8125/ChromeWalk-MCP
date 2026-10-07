---
name: chromewalk-release-gate
description: Use ChromeWalk as a release gate - run the smoke suite (chromewalk_smoke / chromewalk smoke) against staging, only on the pages a change touched (--changed / --git-diff), click-explore safe controls (--explore) and report click coverage (--coverage), then act on the exit code (0 ship, 3 stop, 5 retry, 4 license). Use when asked to test a site or web app before release, check a deploy, run smoke tests, find UI breakage, or gate a pull request.
---

# ChromeWalk release gate

ChromeWalk runs locally and drives real browsers. `smoke` is the gate: every (page x viewport) unit
runs in its own isolated headless browser, ten detectors fire (console errors, uncaught exceptions,
unhandled rejections, "undefined" text, `input=undefined`, handler undefined, horizontal overflow,
top-right overflow, modal overlap, mobile overlay on desktop), plus your steps, asserts, compare and
design hooks. Use the MCP tool `chromewalk_smoke` when it is connected, otherwise the CLI
(`chromewalk smoke ...`, or `cw smoke ...` from a source checkout). Always ask for JSON (`--json`).

## 1. Prove the gate itself works (once per machine or after an upgrade)

```
chromewalk smoke --self-test --json
```
It runs every detector against a deliberately buggy fixture; exit 3 means a detector is broken (a green
gate would then be false). MCP: `{"self_test": true}`.

## 2. Run the gate

Minimal, no suite file:
```
chromewalk smoke --url https://staging.example.com/ --url https://staging.example.com/pricing --viewport 1440x900 --viewport 390x844@3 --json
```
With a suite file (pages, viewports, steps, asserts; schema in `docs/SCENARIOS.md` and
`docs/tools/smoke.md`):
```
chromewalk smoke suite.json --base-url https://staging.example.com --json
```
- `--base-url` overrides the suite's `base_url` (flag > env `CW_SMOKE_BASE` > env `SMOKE_BASE` > suite).
  An unreachable suite `base_url` is only a warning, so always pass the host you mean.
- Smoke is headless by default; `--headful` only for debugging.
- `--viewport` is repeatable: `WxH` or `WxH@dpr`. A bad value is exit 2.
- Runs are concurrent and RAM-governed (`--agents N` caps them).

### Only the pages a change affects: `--changed` / `--git-diff`
The suite carries `route_map` ({file glob or `dir/`: [routes]}) and `always` (core routes that run on
every change run):
```
chromewalk smoke suite.json --changed src/pricing/plan.css,src/nav.js --base-url https://staging.example.com --json
chromewalk smoke suite.json --git-diff=origin/main --repo . --base-url https://staging.example.com --json
```
- A changed file that no `route_map` entry covers runs the FULL suite (safe default); the summary's
  `changed.reason` says which file did it. Do not treat that as a bug - add the missing route_map entry.
- `--git-diff` with no REV compares the working tree to HEAD (incl. untracked files); write
  `--git-diff=REV` with `=` when a positional suite follows.
- Inline `--url` runs have no route_map: `--changed` there is exit 2 `bad_changed`.
- `no_matching_pages` (exit 2): every changed file mapped to routes that match no page and there is no
  `always` core - add `"always": ["/"]`.

### Steps with expectations
Steps in a suite page may carry `expect`; a failing step is a finding that names the step:
```json
{"path": "/settings", "steps": [
  {"do": "click", "selector": "#tab-billing", "expect": {"selector": "#billing-panel"}},
  {"do": "click", "selector": "text=Save", "expect": {"text": "Saved", "within": ".toast", "timeout": 5}}
]}
```
`expect` keys: `selector`, `absent`, `text` (+ `within`), `url_contains`, `url_changed`, `timeout` (s, default 5).
After every step ChromeWalk also checks new uncaught exceptions, console errors and failed same-origin
requests (HTTP 5xx or network failure fails the step; 4xx is a warning). Step `do` values: click, tap,
type/fill (`text`), wait_for, wait (`seconds`), eval (`js`).

### Explore the controls nobody scripted: `--explore N`
```
chromewalk smoke suite.json --explore 25 --coverage --base-url https://staging.example.com --json
```
After the checks ChromeWalk clicks up to N SAFE controls per page (tabs, buttons, menus, accordions,
same-site links), checks each click (exceptions, console errors, failed same-origin requests) and restores
the page. It NEVER clicks destructive controls (delete, remove, log out, pay, purchase, send, submit,
unsubscribe, reset, publish and similar), form submit buttons, downloads, new-tab links or other sites.
Use it on staging or a test account. `--explore-time S` caps seconds per page (default 20);
`--explore-allow REGEX` lets controls with a matching label through even if they look destructive - only
use that when the user explicitly asks. A suite page can set `"explore": N`.

### Coverage: `--coverage`
Per page: `COVERAGE <uid> controls=47 scripted=12 explored=25 skipped_destructive=5 skipped_offsite=3
skipped_cap=2 errors=1` (also in each result's `coverage`). `controls` = interactive controls found,
`scripted` = covered by steps, `explored` = clicked by explore, `skipped_*` = not clicked and why,
`errors` = explored clicks that errored. Report coverage honestly: explore does not fill forms or finish
multi-page journeys.

### Useful switches
`--session NAME` (saved warm login), `--auth "key=value;k2=v2"` or `--local-storage-file FILE` (localStorage
token apps), `--profile NAME` (re-run only the matching unit(s)), `--ignore-console REGEX` (known benign
noise), `--skip-check NAME` (leave a detector out, listed in the summary), `--wait-for-selector CSS`,
`--wait-for-network-idle=800`, `--set-attr html:data-theme=dark`, `--slow-budget SEC`, `--fail-on-slow`,
`--unit-retries N`, `--circuit-breaker N`, `--no-artifacts`.

## 3. Read the result and act on the exit code

| Exit | Meaning | What to do |
|---|---|---|
| 0 | every unit passed (slow units still count as passed) | ship |
| 3 | one or more units really FAILED | stop; report each finding |
| 5 | no real failure, but units failed for INFRA reasons (browser launch, CDP, engine missing, circuit breaker opened) | retry; if it repeats, see the `chromewalk-troubleshooting` skill |
| 4 | license blocked | see the `chromewalk-onboarding` skill |
| 2 | usage error (bad flag, suite, viewport, hook) | fix the input named in `error.message` |
| 6 | MCP only: the call timed out or was cancelled | narrow the call (fewer pages) or re-issue |
| 1 | internal error | file it: `chromewalk recommend --type bug --tool smoke --title "..."` |

Never report a 5 as a failing site, and never report a 3 as flaky without looking at the findings.

The JSON summary (`--json`, also `<out>/summary.json`): counts `passed`, `failed`, `infra_failed`,
`not_run`, `slow`; `failures[]` and `unit_results[]` with `findings [{check, detail, selector}]`;
`changed` (why those pages ran); `warnings[]` (e.g. `base_url_unreachable`); `findings_by_check`.
Every unit also streams to `<out>/results.jsonl` as it finishes (`outcome`: passed | failed | slow |
infra_failed | not_run).

- Every FAIL or slow unit gets `artifacts/<uid>/` under the run folder: `screenshot` (full page),
  `console.json`, `network.json` (failed / blocked / HTTP >= 400), `dom.html`, `timing.json`, `nav.json`.
  Open the screenshot and console.json before explaining a failure.
- `slow` = every check clean but page time over budget (`--slow-budget`, else the p95 of that page's own
  recent clean runs, else 30 s). It is reported, not a failure, unless `--fail-on-slow`.
- stderr prints one `PASS` / `FAIL` / `INFRA` / `SLOW` line per unit and a `SUMMARY ... exit=N` line.

## 4. A good gate report

State: the host tested, pages x viewports, pass/fail/infra counts, each finding with its page, check and
selector, the artifact path for failures, coverage numbers when `--coverage` ran, and the exit code. If
`changed.reason` shows a full-suite fallback, say so.

## Rules
- Run against staging or a test account; explore clicks real buttons.
- Never put tokens on a command line you will paste into chat; use `--auth "tp_key=${TP_KEY}"` (the
  value is expanded from the environment and never logged) or `--local-storage-file`.
- Do not solve CAPTCHAs or bot challenges; a blocked host is reported as a preflight finding, not worked
  around.
- Do not invent flags: `chromewalk help smoke` and `docs/tools/smoke.md` are the authority.
