---
name: close-today
description: End-of-day review. Reads today's plan, pulls the latest status from up to 7 data sources, auto-checks completed items, generates retrospective stats and AI coaching feedback, and writes output to the terminal and Obsidian daily note. Run before shutting down each day.
---

# /close-today

End-of-day review. Reads today's plan, pulls the latest status from enabled data sources, auto-determines completion status, generates retrospective statistics and AI coaching feedback, and writes output to the terminal and Obsidian daily note.

## Step 0: Load config

Read `~/.claude/life-os-config.md`. Parse all settings into variables.

For each `SOURCE_*` set to `disabled`, skip that source entirely.

If the config file does not exist, print:
  ❌ Config not found. Copy life-os-config.md from the repo to ~/.claude/ and fill in your values.
Then stop.

Variable reference:
- `{{OBSIDIAN_VAULT}}` — vault root path
- `{{SLACK_USER_ID}}` — Slack member ID
- `{{LINEAR_PREFIX}}` — ticket prefix
- `{{LANGUAGE}}` — output language (zh or en)
- `{{STOP_TIME}}` — daily shutdown time (default 19:00)
- `{{SOURCE_CALENDAR}}` — enabled/disabled
- `{{SOURCE_GITHUB}}` — enabled/disabled
- `{{SOURCE_LINEAR}}` — enabled/disabled
- `{{SOURCE_SLACK}}` — enabled/disabled
- `{{SOURCE_TIMING_APP}}` — enabled/disabled
- `{{SOURCE_NOTION}}` — enabled/disabled

## Step 1: Read today's plan

Determine today's date in `YYYY-MM-DD` format.

Read the Obsidian daily note at:

```
{{OBSIDIAN_VAULT}}/{{YYYY-MM-DD}}.md
```

Parse the `## 今日计划` section (from `## 今日计划` to the next `## ` heading or end of file).

Extract every item with:
- **Checkbox state**: `[ ]` (unchecked) or `[x]` (checked)
- **Full item text**
- **🍅 pomodoro estimate** (the number before 🍅)
- **Section category** (📅 固定时间 / 🔥 紧急 / 📋 专注工作 / 📬 待处理 / 🔄 遗留)

If `## 今日计划` section does not exist, print `❌ No plan found for today. Please run /today first.` (or the equivalent in `{{LANGUAGE}}`) and **stop**.

Identify item types by pattern matching:

| Pattern | Item type |
|---------|-----------|
| `{{LINEAR_PREFIX}}-\d+` in text | Linear issue |
| `PR #\d+` or `#\d+` in text | GitHub PR |
| Line starts with time range `HH:MM-HH:MM` | Calendar event |
| Everything else | Generic item |

(If `SOURCE_LINEAR` is disabled, treat any ticket-like pattern as a generic item.)

## Step 2: Fetch latest status from data sources

**Run all enabled sources in parallel** (use parallel tool calls where possible).

### Source 1 — Google Calendar (if `SOURCE_CALENDAR: enabled`)

Use `mcp__google-drive__calendar_events_list` to fetch today's events.

- Parameters: set the date range to today only (start of day to end of day)
- Extract: **start time**, **end time**, **title**

### Source 2 — Linear (if `SOURCE_LINEAR: enabled`)

Use `mcp__plugin_linear_linear__list_issues` (or the available Linear MCP tool) to fetch issues assigned to me.

- Make **two** calls:
  1. `assignee: "me"`, `state: "started"` — captures In Progress and In Review
  2. `assignee: "me"`, `state: "unstarted"` — captures Todo, Ready for Work, Backlog
- Extract: **identifier** (e.g. `{{LINEAR_PREFIX}}-751`), **title**, **status**, **priority**

### Source 3 — GitHub (if `SOURCE_GITHUB: enabled`)

Run these Bash commands from the current directory (or project root if known):

```bash
gh pr list --search "review-requested:@me" --json number,title,url,state,mergedAt
```

```bash
gh api notifications --jq '.[] | select(.reason=="review_requested" or .reason=="mention")'
```

- Also check specific PRs mentioned in the plan: for any `PR #NNN` or `#NNN` found in Step 1, run:
  ```bash
  gh pr view NNN --json state,mergedAt
  ```
- Extract: **PR number**, **title**, **state** (open/merged/closed)

### Source 4 — Slack (if `SOURCE_SLACK: enabled`)

Use `mcp__slack__conversations_history` to check channels.
(Note: MCP tool names may vary depending on your Slack MCP installation.)

- Focus on **@mentions** and **DMs** from the past 48 hours
- Use the `oldest` parameter set to 48 hours ago (Unix timestamp)
- Filter for messages mentioning `{{SLACK_USER_ID}}`
- Extract: **channel name**, **sender**, **whether user has replied in thread**

### Source 5 — Gmail

Check if Gmail MCP tools are available (tools with prefix `mcp__gmail`).

- **If NOT available**: print exactly:
  ```
  ⚠️ Gmail MCP not configured, skipping email.
  ```
  (Use the appropriate language variant based on `{{LANGUAGE}}`.)
- **If available**: fetch unread emails

### Source 6 — Notion (if `SOURCE_NOTION: enabled`)

Use `mcp__notion__notion-search` (or available Notion MCP tool) to find tasks assigned to the user.

- Extract: **title**, **status**
- If Notion fails or token expired, log briefly and continue

### Source 7 — Timing App (if `SOURCE_TIMING_APP: enabled`)

Fetch today's time data from local Timing SQLite database:

```bash
python3 ~/.claude/skills/timingapp-timeline-loader/generate_timeline.py --date {{YYYY-MM-DD}} --summary --output -
```

- If the script fails (Timing not installed, DB not found), print a warning and continue
- Parse JSON output: `total_hours`, `by_project` (map of project→hours), `work_sessions`

## Step 3: Match items and determine completion status

For each **unchecked** `- [ ]` item in the plan, match against data source results:

| Item type | High confidence → auto-check | Medium confidence → ask user |
|-----------|------------------------------|------------------------------|
| Calendar event | Current time > event end time | — |
| Linear `{{LINEAR_PREFIX}}-xxx` | Issue status is Done or Cancelled | Status still In Progress but has activity today |
| GitHub `PR #xxx` | PR is merged, or I submitted a review | PR open with new comments |
| Slack @mention | I replied in the thread after the mention | Thread has new messages, I haven't replied |
| Gmail | — | Cannot determine (always ask) |
| Notion task | Status is Done/Complete | — |
| Generic (no identifier) | Already `[x]` in daily note | Cannot auto-determine (always ask) |

Items already checked `[x]` in the daily note → mark as manually completed, no matching needed.

Classify each item into one of three categories:
- **✅ completed** (high confidence auto-check OR already `[x]`)
- **❓ needs confirmation** (medium confidence)
- **❌ not completed** (no evidence of completion)

## Step 4: Auto-check high confidence items in daily note

For each high-confidence completed item that is currently `- [ ]` in the daily note:
- Use the Edit tool to change `- [ ] {{exact item text}}` → `- [x] {{exact item text}}`
- Track which items were auto-checked (tagged `[auto]` in the report)

Calendar events that don't have checkboxes (plain `-` bullets) — just note them as completed in the report, no edit needed.

## Step 5: Terminal output — completion status + confirmation

Print to terminal in this order:

1. **Completed** section — all completed items:
   - `✓ [auto] {{item}} (→ {{reason}})` for auto-checked items
   - `✓ [manual] {{item}}` for items already checked by user
   - `✓ {{item}} (past)` for past calendar events

2. **Needs confirmation** section — medium-confidence items:
   - List each item with reason for uncertainty
   - Use `AskUserQuestion` tool to let user confirm/deny each item
   - After user responds, update daily note accordingly (`[x]` if confirmed, keep `[ ]` if not)

3. **Not completed → carry over tomorrow** section — clearly not completed items

(If `{{LANGUAGE}}` is `zh`, use Chinese section headers and labels instead.)

## Step 6: Generate retrospective statistics

Calculate:

- **Completion rate**: (completed items / total items) as percentage
- **Pomodoro consumption**: sum of 🍅 estimates for completed items / sum of all 🍅 estimates
- **Incomplete reason classification** — for each uncompleted item, classify reason:
  - `⏰ Underestimated` — item was started but not finished (e.g., Linear status still In Progress)
  - `🚧 Blocked` — item was blocked by external dependency
  - `📥 New insertion` — any new items that appeared during the day but weren't in original plan
- **Time analysis** (from Timing data, if `SOURCE_TIMING_APP: enabled`):
  - Group `by_project` into meaningful categories
  - Calculate percentage of total hours for each category
  - Display as progress bar: `█` (filled) and `░` (empty), 10 chars wide

## Step 6.5: One Thing + Weekly Outcome check

### One Thing check

Read today's plan from the daily note. Find the **One Thing** (the `🎯 **One Thing**` line or the `🎯 One Thing:` line in the plan).

- If the One Thing was completed → note as a closing for the week
- If not completed → surface it prominently in the "not completed" section and flag it for tomorrow

### Weekly Outcome progress

Read the current week's weekly note:

```
{{OBSIDIAN_VAULT}}/{{YYYY}}-W{{WW}}.md
```

(Use the format from `{{WEEKLY_NOTE_FORMAT}}` in config.)

Compare today's completed items against the weekly Outcome and Projects. Generate a brief progress line:
- "Weekly Outcome: [description] — Today's progress: [what was done today toward it]"
- If no progress toward the weekly outcome today, flag it.

(If `{{LANGUAGE}}` is `zh`, generate in Chinese.)

### Stop Time check

Check whether the user is running `/close-today` before or after their Stop Time (`{{STOP_TIME}}`).
- If before → "Good, you're wrapping up on time."
- If after → "Note: past Stop Time ({{STOP_TIME}}). Try to wrap up on time tomorrow."

(Adapt phrasing to `{{LANGUAGE}}`.)

## Step 6.8: Agent Log (3 questions)

After the statistics and before AI coaching, ask the user 3 short questions. Keep it to 5 min max.

**Q1** (auto-filled + confirm):
> "What did you deliver today?"

Pre-fill from completed items. Show the list and ask user to confirm or add anything.

**Q2** (user fills):
> "Did you fall into any old patterns today? What was the trigger?"

Wait for user response. If user says "no" or equivalent, record "none" and move on.

**Q3** (user fills):
> "What is one small adjustment for tomorrow?"

Wait for user response. Record their answer.

These 3 answers go into the output as the `### 📝 Agent Log` section.

(If `{{LANGUAGE}}` is `zh`, ask the questions in Chinese.)

## Step 7: Generate AI coaching feedback

Based on ALL collected data (completion stats, time analysis, what was done, what wasn't), generate feedback from 3 perspectives. Each perspective gives 2-4 sentences based on the actual data from today.

### 👔 Career Coach

- Focus on: skill growth, impact of completed work, career goal alignment
- Reference specific tickets/PRs completed and their significance
- If something was blocked, suggest escalation strategies

### ⚡ Efficiency Coach

- Focus on: time allocation, pomodoro completion rate, work rhythm
- Compare planned vs actual
- Give specific suggestion for tomorrow's scheduling

### 🌱 Positive Intelligence

- Focus on: psychological state, identify saboteur patterns (Judge, Achiever, Controller, Hyper-Achiever, etc.)
- Acknowledge progress with empathy
- Suggest a PQ rep (mental fitness exercise)
- Use warm, supportive tone

(If `{{LANGUAGE}}` is `zh`, generate all coaching feedback in Chinese.)

## Step 8: Terminal output — full retrospective

Print the complete retrospective following the **Terminal Output Template** in `references/output-template.md` exactly (if that file exists; otherwise use a structured format with the sections above).

## Step 9: Write retrospective to Obsidian

Append a `## Today's Review` section (or `## 今日回顾` if `{{LANGUAGE}}` is `zh`) to the daily note following the **Obsidian Daily Note Template** in `references/output-template.md` (if it exists).

Write behavior:

| Condition | Action |
|-----------|--------|
| Review section does not exist | Append review section at end of file |
| Review section already exists | Replace from the heading to next `## ` heading or end of file |

## Error Handling

- If any single data source fails (MCP timeout, API error), log the error briefly and continue with remaining sources. Never let one source failure stop the entire review.
- If ALL sources fail, print an error message and suggest checking MCP configuration.
- Gmail is expected to be unconfigured initially — this is not an error.
- Timing App not installed → skip time analysis section entirely.
- Notion token expired → log briefly, skip.
- If the today's plan section is not found → stop with an appropriate error message prompting the user to run `/today` first.
- If `SOURCE_LINEAR: disabled`, skip all Linear matching logic; treat any ticket-like patterns as generic items.
- If `SOURCE_SLACK: disabled`, skip Slack matching logic.
