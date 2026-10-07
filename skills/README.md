# ChromeWalk skills

Four small skills (plain `SKILL.md` files with `name` and `description` frontmatter) that teach an AI
coding agent to use ChromeWalk well. They are local files: nothing is uploaded, and they only describe
ChromeWalk's own commands and MCP tools. They assume ChromeWalk is installed and connected (CLI on the
PATH, or the MCP server registered with `chromewalk connect`).

| Skill | Teaches the agent to |
|---|---|
| [`chromewalk-release-gate`](chromewalk-release-gate/SKILL.md) | run the smoke suite as a release gate: `--self-test`, `--changed` / `--git-diff`, step `expect`, `--explore`, `--coverage`, artifacts, and what exit 0 / 3 / 5 / 4 / 2 mean |
| [`chromewalk-cross-engine`](chromewalk-cross-engine/SKILL.md) | test in Chromium, Firefox and WebKit with `xengine`, logged in via `--session` (saved with `warm`), `--auth` or `--local-storage-file`; read divergences |
| [`chromewalk-onboarding`](chromewalk-onboarding/SKILL.md) | get from `license_required` to a working install inside the chat (`chromewalk_register`, `chromewalk_activate`, `--remember`), then `setup` and `connect` |
| [`chromewalk-troubleshooting`](chromewalk-troubleshooting/SKILL.md) | diagnose exit 5 / launch problems with `doctor`, clean up with `procs --reap`, read the run job files, `calls`, `logs`, `reset`, and file gaps with `recommend` |

## Install

Where to get them: the `skills/` folder of the public ChromeWalk-MCP repository
(https://github.com/gator8125/ChromeWalk-MCP/tree/main/skills) - download or clone it; `<chromewalk>` below
means that folder's parent. They are not part of the Windows installer or the `.mcpb` bundle.

Copy the four folders (each contains one `SKILL.md`) into the agent's skills directory. Restart or
reload the agent afterwards. The folder name must stay equal to the skill `name`.

### Claude Code
- Personal (all projects): `~/.claude/skills/<skill-name>/SKILL.md`
- One project (commit to share with the team): `<project>/.claude/skills/<skill-name>/SKILL.md`

PowerShell:
```powershell
Copy-Item -Recurse "<chromewalk>\skills\chromewalk-*" "$env:USERPROFILE\.claude\skills\"
```
bash:
```bash
mkdir -p ~/.claude/skills && cp -r <chromewalk>/skills/chromewalk-* ~/.claude/skills/
```
Claude loads a skill when your request matches its description, or you can type `/chromewalk-release-gate`.

### Cursor
Cursor loads skills from `.cursor/skills/` (project) and `~/.cursor/skills/` (user), `.agents/skills/` /
`~/.agents/skills/`, and also from Claude's and Codex's directories, so the Claude Code install above
already works in Cursor. To install for Cursor only, copy the folders to `~/.cursor/skills/`. Invoke
one manually by typing `/` in Agent chat and searching for its name.

### Codex (CLI and IDE)
Codex loads skills from `$CWD/.agents/skills`, `$REPO_ROOT/.agents/skills`, `$HOME/.agents/skills` and
`/etc/codex/skills`. Copy the folders to `~/.agents/skills/` (all projects) or `<repo>/.agents/skills/`
(one repository) and restart Codex. A skill can be disabled without deleting it with a
`[[skills.config]]` entry (`path = ".../SKILL.md"`, `enabled = false`) in `~/.codex/config.toml`.

(Directory locations as documented by Anthropic, Cursor and OpenAI on 2026-10-06; check the vendor's
current docs if a skill does not appear.)

## Connect ChromeWalk itself first

The skills call ChromeWalk's tools; they do not install it. Register the MCP server in your agent with
one command (idempotent, backs up each config first, never writes a license key):

```
chromewalk connect --list
chromewalk connect claude-code codex cursor
```

Without MCP the agent can still use the CLI (`chromewalk smoke ...`) from a terminal.

## Keeping them accurate

The skills describe the commands as of ChromeWalk 13.0. `tests/test_v13_skills.py` checks that every
`chromewalk <command> --flag` in a skill exists in the generated `docs/tools/*.md`, that every
`chromewalk_*` MCP tool named is real, and that the frontmatter is valid, so a renamed flag fails the
test instead of silently misleading an agent.
