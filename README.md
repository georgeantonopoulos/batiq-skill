# BATIQ skill for AI agents

An [Agent Skill](https://agentskills.io) that teaches Claude Code, Codex,
Antigravity and other agents to drive [BATIQ](https://batiq.app), the free
native node compositor, through its `batiq-headless` command line: build and
edit comps, wire a 3D Set, track, key, animate and render.

BATIQ changes fast, so the skill holds the stable shape of the tool and sends
the agent to the live docs (https://batiq.app/llms.txt) and to the binary's
own error messages for everything else.

## Install

Paste this to your agent:

> Install the BATIQ skill: read
> https://raw.githubusercontent.com/georgeantonopoulos/batiq-skill/main/INSTALL.md
> and follow it.

Or by hand, for Claude Code:

```sh
git clone --depth 1 https://github.com/georgeantonopoulos/batiq-skill.git ~/.claude/skills/batiq
```

Other agents' folders are listed in [INSTALL.md](INSTALL.md).
Update with `git -C ~/.claude/skills/batiq pull`.

## What's inside

- [`SKILL.md`](SKILL.md): finding the executable, the JSON-RPC protocol,
  how to discover actions and parameters, rules that avoid lost work.
- [`references/recipes.md`](references/recipes.md): tested request files
  for common jobs.

## Licence

MIT. BATIQ itself is free to download from https://batiq.app.
