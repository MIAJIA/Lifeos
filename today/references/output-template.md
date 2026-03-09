# Output Template Reference

This file defines the two output formats for the `/today` skill:
1. **Terminal output** — displayed directly in the CLI
2. **Obsidian daily note** — written to the user's vault

Output language is controlled by `{{LANGUAGE}}` in `~/.claude/life-os-config.md`.
- `en`: English output (section names, labels, summaries all in English)
- `zh`: Chinese output with English proper nouns retained

The templates below show the `zh` variant. When `{{LANGUAGE}}: en`, translate section headers and labels accordingly (e.g., `📅 固定时间` → `📅 Fixed Time`, `🔄 遗留` → `🔄 Carry-forward`, etc.).

---

## 1. Terminal Output Template

```
☀️ Today's Plan — {{DATE}} ({{DAY}})
━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 One Thing: {{the closing slice for today}}
📎 Support: {{task 2}}, {{task 3}}
🛑 Stop Time: {{STOP_TIME}}
⏸️ Circuit Breaker: {{auto-generated interrupt suggestion}}

📅 固定时间
  {{HH:MM}}-{{HH:MM}}  {{event title}}                 🍅 {{estimate}}

🔥 紧急（别人在等）
  ☐ {{item description}}                                🍅 {{estimate}}

📋 专注工作
  ☐ {{item description}}                                🍅 {{estimate}}

📬 待处理
  ☐ {{item description}}                                🍅 {{estimate}}

🔄 遗留 (carry-forward)
  ☐ [昨日] {{item description}}                         🍅 {{estimate}}

💬 频道动态
  • **#channel-name**: [2-3 sentence summary]

🔍 关注话题
  • [topic name]: [2-4 sentence summary of recent activity]

📰 今日资讯
  • {{title}}  — {{one-line summary}}  ({{source}})

─────────────────────────────
合计: ~{{total}} 🍅 ({{hours}}h)  |  可用时间: ~{{available}}h (减去会议)
{{overload warning if total > available}}
```

### Rendering Rules

- **Sections are omitted** if they contain zero items (don't show empty headers).
- **Pomodoro estimates** are right-aligned with 🍅 emoji.
- **Overload warning** appears only when total pomodoros exceed available time.
  - `en`: `⚠️  Overloaded! Planned 12🍅 but only ~4h available — consider deferring low-priority tasks`
  - `zh`: `⚠️  超载！计划 12🍅 但只有 ~4h 可用，考虑推迟低优先级任务`
- **Day names**: `en` — Mon/Tue/Wed/Thu/Fri/Sat/Sun; `zh` — 周一/周二/周三/周四/周五/周六/周日
- **One Thing block** always appears at the top, before any section. Shows the single most important closing slice.
- **Weekly reference** (blockquote line) only appears if a weekly note exists for the current week. Omit if no weekly note found.
- **Circuit Breaker** is auto-generated based on One Thing complexity. Stop Time comes from `{{STOP_TIME}}` in config.

---

## 2. Obsidian Daily Note Template

### Target Path

```
{{OBSIDIAN_VAULT}}/{{YYYY-MM-DD}}.md
```

### Write Behavior

| Condition | Action |
|-----------|--------|
| File does not exist | Create new file with `## 今日计划` (or `## Today's Plan`) as first section |
| File exists, no plan section | Append plan section at the end |
| File exists, has plan section | Replace the existing plan section |

### Format

```markdown
> 📋 本周重点: [[{{YYYY}}-W{{WW}}]] — 🎯 {{weekly outcome}} | 🛡️ {{non-negotiable summary}}

## 今日计划

🎯 **One Thing**: {{closing slice for today}}
📎 **Support**: {{task 2}}, {{task 3}}
🛑 **Stop Time**: {{STOP_TIME}} | ⏸️ **Circuit Breaker**: {{interrupt suggestion}}

### 📅 固定时间
- {{HH:MM}}-{{HH:MM}} {{event title}} 🍅 {{estimate}}

### 🔥 紧急（别人在等）
- [ ] {{item description}} 🍅 {{estimate}}

### 📋 专注工作
- [ ] {{item description}} 🍅 {{estimate}}

### 📬 待处理
- [ ] {{item description}} 🍅 {{estimate}}

### 🔄 遗留
- [ ] [昨日] {{item description}} 🍅 {{estimate}}

### 💬 频道动态
- **#channel-name**: [2-3 sentence summary]

### 🔍 关注话题
- **[topic name]**: [2-4 sentence summary of recent activity]

### 📰 今日资讯
- {{title}} — {{one-line summary}} ([{{source}}]({{link}}))

> 合计: ~{{total}} 🍅 ({{hours}}h) | 可用时间: ~{{available}}h
```

### Formatting Rules

- **Fixed-time events** use plain bullet list (`-`) — meetings don't need checking off.
- **All other items** use checkbox format (`- [ ]`).
- **Each item** is annotated with a 🍅 estimate.
- **Carry-forward items** are prefixed with `[昨日]` (or `[Yesterday]` if `{{LANGUAGE}}: en`).
- **News items** use plain bullet list (`-`) with linked source — no checkbox, no 🍅.
- **Sections are omitted** if they contain zero items.
- **Subsection headers** use `###` (one level below `## 今日计划`).
