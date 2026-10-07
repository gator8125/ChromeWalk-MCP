---
name: chromewalk-troubleshooting
description: Diagnose ChromeWalk problems - exit 5 infra errors, browser launch failures, engine_missing, leftover Chrome processes, runs that went quiet or were killed - using doctor, procs, logs, reset and the run job files. Use when a ChromeWalk tool fails with launch_failed, cdp_timeout, engine_missing, profile_locked, circuit_open, timeout_dispatch, or when a long audit/smoke run seems stuck or vanished.
---

# ChromeWalk troubleshooting

Read the exit code first. ChromeWalk's contract: 0 ok, 2 usage (fix your input), 3 real failure (the
site or the check failed), 4 license (see `chromewalk-onboarding`), 5 infra / retryable (the browser or
an engine, not the site), 6 MCP only (timed out or cancelled), 1 internal error. Every JSON failure has
`error.class`, `error.code`, `error.message`, `error.hint`, plus `run_id` and `corr_id`. The hint is
usually the fix; read it before guessing.

## Step 1: doctor (read-only, never license-gated)
```
chromewalk doctor --json              # or MCP chromewalk_doctor {}
chromewalk doctor --launch --json     # adds a real headless launch + CDP connect (throwaway profile)
chromewalk doctor --fix-hints         # only the failing checks, each with its fix
chromewalk doctor --strict            # advisories also fail (exit 5)
```
Each check is `{name, ok, detail, hint}`. Doctor exits 5 only when a REQUIRED check fails; advisories
(Defender coverage, Dev Drive, orphans, profile pool) are warnings. It reports where everything is
installed (interpreter, dependency versions, nodriver patch, Playwright engines, the Chrome for Testing
binary, config sources, unread recommendations) and flags slow starts with advice.

| Symptom | Likely code | Fix |
|---|---|---|
| a browser tool exits 5 | `engine_missing` | `chromewalk setup`, or point `CHROMEWALK_BROWSER=<path to chrome.exe>` at an existing binary |
| exit 5 | `launch_failed` / `cdp_timeout` | `doctor --launch`; lower `launch_slots` or `--agents` on a busy machine; `chromewalk config set cdp_timeout 60`; `chromewalk resources` for load; reap orphans |
| exit 5 | `profile_locked` | another ChromeWalk run owns that profile: wait, use another `--profile`, or `procs --reap` if its owner is dead |
| exit 5 | `circuit_open` | consecutive units failed for infra reasons so the rest were not run: fix the cause (doctor), re-run, or raise `circuit_breaker` |
| exit 5 | `cdp_no_page_target`, `unit_exception` | transient; smoke already retries (`unit_retries`); if it repeats, `chromewalk recommend --type bug ...` |
| exit 5 | `dependency_missing` / `dependency_broken` | `chromewalk setup`; `chromewalk reset --hard --yes` repairs the nodriver patch; then `doctor` |
| exit 6 | `timeout_dispatch` / `cancelled` | the per-tool budget passed (smoke / xengine 900 s) or the call was cancelled: narrow the call, or raise config `tool_timeout` |
| host wrongly "blocked" | probe finding | `chromewalk probe <url>`; `chromewalk targets forget <host>` or `--probe on` |

## Step 2: leftover browsers (`procs`)
```
chromewalk procs --json                 # ChromeWalk's own Chrome trees with an orphan verdict + recent jobs
chromewalk procs --reap --dry-run --json  # what a reap would kill
chromewalk procs --reap --json          # tree-kill VERIFIED orphans, drop stale locks
```
`procs` only ever lists and kills ChromeWalk's OWN browser trees (launcher dead, no CDP client, lock
owner dead, user-data-dir under ChromeWalk's profiles). It never touches the user's own Chrome. Do not
kill browsers or processes by name yourself; use `procs --reap`. Runs also clean up their browsers when
killed, and a CLI run sweeps dead launchers before launching.

## Step 3: a run that went quiet, vanished or was killed (lifecycle job files)
Every tool run writes a job file `<data_dir>/run/jobs/<job_id>.json` (schema `chromewalk.job/1`),
replaced atomically:
- `state`: `running` -> `complete` | `failed` | `cancelled` | `abandoned`, plus `phase`, units done /
  total, `last_ok`, `last_heartbeat_at`, `exit_code`, `reason`.
- Heartbeat: the file is refreshed every 15 s, and on the CLI a line goes to stderr once a run is older
  than one interval: `[cw:audit job=3f9c1a2b7d4e] running 45s phase=capture 17/60 last_ok=p017 (job file: ...)`.
  stdout is untouched, so JSON output stays pure.
- Caller gone: if the wrapper that started the run is killed, the run is cancelled cleanly
  (`state: cancelled`, `reason: cancelled:caller_gone`). A caller that merely stops waiting does not cancel
  anything. `CW_JOB_DETACH=1` turns the watch off for deliberately detached runs.
- Abandoned: a run killed outright (no chance to write its end state) is marked `abandoned` the next
  time ChromeWalk starts.
- `chromewalk procs --json` lists the recent jobs and their states under `"jobs"`. Use it to answer "did
  my 280-state audit finish?": `complete` + `exit_code`, or `failed` / `cancelled` / `abandoned` and why.
- Very large runs print a size warning above 160 states with a hint to split them.

For MCP calls: `chromewalk_calls` lists in-flight and recent calls (queued / running / done / failed /
timeout / cancelled, with `corr_id`) and returns a finished call's result by id even after your client timed
out (`wait_s` long-polls up to 25 s). `chromewalk_cancel {"id": "<corr_id>"}` stops one and kills its
whole process tree. Every tool result carries `_meta.corr_id`.

## Step 4: logs
```
chromewalk logs --failures --json         # recent failed calls (CLI and MCP)
chromewalk logs --corr <corr_id> --json   # one call
chromewalk logs --timeline <corr_id> --json
chromewalk logs --gaps --json
```
MCP: `chromewalk_logs` (`{"mode": "errors", "n": 20}`).

## Step 5: reset (plan first)
```
chromewalk reset --dry-run --json     # the plan
chromewalk reset --json               # soft: sweep, reap, locks, stale results, pool rebuild
chromewalk reset --hard --yes --json  # also target memory, tuning.json, nodriver patch, engines check
```
Always show the user the `--dry-run` plan before `--hard`.

## Step 6: file the gap
If a failure is a ChromeWalk bug or a docs hole, file it so it reaches the maintainers (works without a
license):
```
chromewalk recommend --type bug --severity P2 --tool smoke --title "..." --evidence "..." --repro "..."
chromewalk recommend --list --json
```
Types: bug, flakiness, performance, refinement, enhancement, new_tool, docs. Include the `run_id` /
`corr_id`, the exit code and what `doctor --json` said. Never include license keys, tokens or passwords.

## Never
- Never kill processes by name or touch the user's own browser.
- Never edit files in the data directory by hand to "fix" license or job state.
- Do not retry a 3 (real failure) as if it were infra, and do not report a 5 as a site bug.
