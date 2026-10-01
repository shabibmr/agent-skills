# agent-skills

Shared agent skills for Grok / Claude-style `SKILL.md` packs.

## Skills

| Skill | Slash | What it does |
| --- | --- | --- |
| [concise](./concise/) | `/concise` | Ultra-short, plain replies |
| [kh-api-logs-check](./kh-api-logs-check/) | `/kh-api-logs-check` | Check live kh-api PM2 logs for errors |

## Install

Copy a skill folder into:

- Project: `<repo>/.grok/skills/<name>/`
- User: `~/.grok/skills/<name>/`

Or clone this repo and link the skill you want.

## Layout

Each skill is a directory with a `SKILL.md` (YAML frontmatter + instructions).
