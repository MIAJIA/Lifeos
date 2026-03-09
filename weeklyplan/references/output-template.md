# Output Template Reference

This file defines the output formats for the `/weeklyplan` skill:
1. **Terminal output** — displayed directly in the CLI
2. **Obsidian weekly note** — written to `{{OBSIDIAN_VAULT}}/YYYY-Wxx.md`
3. **Mid-week debug output** — appended to existing weekly note

---

## 1. Terminal Output Template

```
📋 Weekly Plan — W{{XX}} ({{MM.DD}} - {{MM.DD}})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 Last Week
  Outcome: ✅ Achieved / 🟡 Partial / 🔴 Missed
  "{{outcome description from last week}}"
  Deep work: {{XX}}h | Slack: {{XX}}h | Meetings: {{XX}}h
  Closed: {{X}} tickets, {{X}} PRs merged
  Lesson: {{user's retro answer}}

🎯 This Week's Outcome
  {{outcome description}}
  Verification: {{how you'll know it's done}}
  Serves monthly main line: {{link to monthly main line}}

📋 Project 1: {{name}}
  ☐ {{LINEAR_PREFIX-xxx}} — {{ticket title}}
  ☐ {{LINEAR_PREFIX-xxx}} — {{ticket title}}
  ☐ {{action item}}

📋 Project 2: {{name}}
  ☐ {{LINEAR_PREFIX-xxx}} — {{ticket title}}
  ☐ PR #{{xxx}} review
  ☐ {{action item}}

🔥 Friction Point
  {{the thing to do FIRST on Monday morning}}
  → Block 45 min

🛡️ Non-negotiables
  Body: {{spec}}
  Relationship: {{spec}}
  Retro: Wed debug + Sun retro

⚪ Backlog (not this week)
  {{LINEAR_PREFIX-xxx}}, {{LINEAR_PREFIX-xxx}} — {{deferred reason}}

🧠 Open Windows: {{count}} ({{trend: down from X / up from X / same as last week}})
  {{parked item 1}}
  {{parked item 2}}
  {{parked item 3}}

─────────────────────────────
📅 Meetings this week: {{count}} (~{{hours}}h)
🧩 Action items: {{count}}
```

### Rendering Rules

- **Sections are omitted** if they contain zero items (don't show empty headers).
- **Day names**: If `{{LANGUAGE}}` is `zh`: `周一` through `周日`. If `en`: Monday through Sunday.
- **Week format**: `W10` (ISO week number, no leading zero for weeks 1-9).
- **Date range**: `MM.DD - MM.DD` format (Monday to Sunday of the week).
- **Retro section**: Only shown if last week's weekly note exists.
- **Backlog section**: Only shown if there are deferred items.
- **Timing section** (Deep work / Slack / Meetings): Only shown if `SOURCE_TIMING_APP` is enabled.
- **Open Windows**: Always show count and trend direction compared to last week.
- **Language**: Respect `{{LANGUAGE}}` setting.

---

## 2. Obsidian Weekly Note Template

### Target Path

```
{{OBSIDIAN_VAULT}}/YYYY-Wxx.md
```

Example: `{{OBSIDIAN_VAULT}}/2026-W10.md`

### Write Behavior

| Condition | Action |
|-----------|--------|
| File does not exist | Create new file with full weekly plan content |
| File exists, no `## Weekly Plan` section | Append `## Weekly Plan` section at the end of the file |
| File exists, has `## Weekly Plan` section | Replace the existing `## Weekly Plan` section (from `## Weekly Plan` to the next `## ` heading or end of file) |

### Format

```markdown
## Weekly Plan

### Retro: W{{xx-1}}
- Outcome: {{status emoji}} {{outcome description}}
- Time: Deep work {{x}}h | Slack {{x}}h | Meetings {{x}}h
- Closed: {{x}} tickets, {{x}} PRs
- Lesson: {{one line}}

### 🎯 Outcome
{{description}}
- Verification: {{criteria}}
- Monthly main line: [[YYYY-MM]]

### 📋 Projects
#### Project 1: {{name}}
- [ ] {{LINEAR_PREFIX-xxx}} — {{ticket title}}
- [ ] {{action item}}

#### Project 2: {{name}}
- [ ] {{LINEAR_PREFIX-xxx}} — {{ticket title}}
- [ ] {{action item}}

### 🔥 Friction Point
- [ ] {{the thing to do FIRST on Monday}} (Block 45 min)

### 🛡️ Non-negotiables
- [ ] Body: {{spec}}
- [ ] Relationship: {{spec}}
- [ ] Retro: Wed debug + Sun retro

### 🧠 Open Windows ({{count}})
- {{parked item 1}}
- {{parked item 2}}

### ⚪ Not This Week
- {{LINEAR_PREFIX-xxx}} — {{deferred item}}

> Monthly main line: [[YYYY-MM]] | Open windows: {{count}} ({{trend}}) | Meetings: {{count}} (~{{hours}}h)
```

### Formatting Rules

- **Subsection headers** use `###` (one level below `## Weekly Plan`).
- **Project items** use checkbox format (`- [ ]`).
- **Non-negotiables** use checkbox format (`- [ ]`).
- **Open Windows** use plain bullet list (`-`) — these are informational, not tasks.
- **Not This Week** uses plain bullet list (`-`) — backlog items.
- **Retro section** uses plain bullet list (`-`) — retrospective data.
- **Omit sections** with zero items.
- **Retro time row**: Omit the `Time:` row if `SOURCE_TIMING_APP` is disabled.
- **Summary line** as blockquote at the bottom with monthly main line reference, window count with trend, and meeting stats.
- **Wikilinks**: Use `[[YYYY-MM]]` for monthly note reference.

---

## 3. Mid-Week Debug Output Template

### Terminal Output

```
📊 Mid-Week Debug — W{{XX}} (Wednesday)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 Outcome: {{from weekly note}}
  Progress: {{X/Y tickets done}} | {{X PRs merged}}
  Status: 🟢 On track / 🟡 At risk / 🔴 Off track

📋 Project 1: {{name}}
  ✅ Done: {{completed items}}
  ⏳ In progress: {{items}}
  ☐ Not started: {{items}}

📋 Project 2: {{name}}
  ✅ Done: {{completed items}}
  ⏳ In progress: {{items}}
  ☐ Not started: {{items}}

🆕 New windows opened since Monday:
  {{list of items not in original plan, or "None"}}

─────────────────────────────
Days remaining: {{X}} | Tickets remaining: {{X}}
```

### Obsidian Append Format

Append (do NOT replace the weekly plan) the following section to the existing weekly note:

```markdown
### Mid-Week Debug ({{YYYY-MM-DD}})
- Outcome status: {{🟢/🟡/🔴}} {{assessment}}
- Project 1 progress: {{X/Y done}}
- Project 2 progress: {{X/Y done}}
- New windows: {{count}} — {{list or "None"}}
- Stuck point: {{user's answer or "None"}}
- Adjustment: {{any mid-week course correction}}
```

### Write Behavior for Debug

| Condition | Action |
|-----------|--------|
| Weekly note exists | Append `### Mid-Week Debug` section at the end of the file (after all existing sections) |
| Weekly note does not exist | Print warning (zh): "⚠️ 没有找到本周的 weekly note (YYYY-Wxx.md)。先跑一次完整的 /weeklyplan 吧。" / (en): "⚠️ No weekly note found for this week (YYYY-Wxx.md). Run a full /weeklyplan first." and stop |

---

## General Formatting Rules (all outputs)

- **Language**: Respect `{{LANGUAGE}}` setting. `zh` = Chinese with English technical terms (ticket IDs like `{{LINEAR_PREFIX}}-751`, PR numbers like `#2601`, product names, app names like VSCode/Cursor/Slack). `en` = full English output.
- **Day names**: If `zh`: `周一` through `周日`. If `en`: Monday through Sunday.
- **Week format**: `W10` (ISO week number).
- **Status emojis**: `✅` achieved, `🟡` partial, `🔴` missed, `🟢` on track.
- **Trend indicators**: "down from X" / "up from X" / "same as last week" for open window counts.
- **No Tony Robbins hype**: Keep tone focused and sprint-like. Data-driven, not motivational.
