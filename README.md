# Life OS — Planning Skills for Claude Code

> Config-driven daily, weekly, and monthly planning skills for [Claude Code](https://claude.ai/claude-code).

English | [中文](./README.zh.md)

## What this is

A set of Claude Code skills that generate your daily plan, weekly sprint, and monthly retro — pulling from your calendar, GitHub, Slack, Linear, and other tools you already use. Works with whatever tools you have; everything is optional.

## Skills

| Skill | Trigger | What it does |
|-------|---------|--------------|
| `today` | `/today` | Pulls 7+ data sources, AI-sorts by urgency, outputs today's plan to terminal + Obsidian daily note |
| `close-today` | `/close-today` | End-of-day review: auto-detects completed items, generates stats and coaching feedback |
| `weeklyplan` | `/weeklyplan` | Weekly retro + RPM planning session (brain dump → outcome → 2 projects → non-negotiables) |
| `monthlyplan` | `/monthlyplan` | Monthly retro + RPM cycle with time tracking and deprecation review |

## Requirements

- [Claude Code](https://docs.anthropic.com/claude-code) (Claude Code CLI)
- `gh` CLI (for GitHub source)
- Optional MCPs depending on which sources you enable (see Data Sources below)

## Install

1. Clone this repo:
   ```bash
   git clone https://github.com/MIAJIA/Lifeos.git
   cd Lifeos
   ```

2. Copy the config template:
   ```bash
   cp life-os-config.md ~/.claude/life-os-config.md
   ```

3. Edit `~/.claude/life-os-config.md` — set your Obsidian vault path and enable the sources you use.

4. Copy skills to your Claude skills directory:
   ```bash
   cp -r today close-today weeklyplan monthlyplan ~/.claude/skills/
   ```

5. Restart Claude Code (or open a new session).

## Configure

Open `~/.claude/life-os-config.md` and set:

- `OBSIDIAN_VAULT` — path to your Obsidian vault (for daily/weekly/monthly notes)
- `SOURCE_*` flags — `enabled` or `disabled` for each data source
- `STOP_TIME` — your daily shutdown time for capacity planning
- `LANGUAGE` — `en` for English output, `zh` for Chinese

Example minimal config (GitHub + Obsidian only):
```
OBSIDIAN_VAULT: /Users/yourname/Documents/MyVault
SOURCE_CALENDAR: disabled
SOURCE_GITHUB: enabled
SOURCE_OBSIDIAN: enabled
SOURCE_SLACK: disabled
SOURCE_LINEAR: disabled
SOURCE_TIMING_APP: disabled
SOURCE_NEWS: disabled
STOP_TIME: 18:00
LANGUAGE: en
```

## Data Sources

| Source | Requires | Config flag |
|--------|----------|-------------|
| Google Calendar | Google Drive MCP | `SOURCE_CALENDAR` |
| GitHub | `gh` CLI | `SOURCE_GITHUB` |
| Obsidian | Local vault path | `SOURCE_OBSIDIAN` |
| Slack | Slack MCP | `SOURCE_SLACK` |
| Linear | Linear MCP | `SOURCE_LINEAR` |
| Timing App | macOS + Timing App | `SOURCE_TIMING_APP` |
| Notion | Notion MCP | `SOURCE_NOTION` |
| News | WebSearch / WebFetch | `SOURCE_NEWS` |
| Second Brain | Local markdown files | `SOURCE_SECOND_BRAIN` |

## Contribute

Using a different tool? Add a data source module in `community/sources/` and share it.
See [CONTRIBUTING.md](./CONTRIBUTING.md) for how.
