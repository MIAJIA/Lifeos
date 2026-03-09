---
name: today
description: Generate today's plan. Aggregate Calendar, Linear, GitHub, Slack, Gmail, Notion, and News data sources, AI-smart-sort, output to terminal + Obsidian daily note. Run once each morning.
---

# /today

Generate today's plan. Fetch from up to 9 data sources in parallel, AI-smart-sort, output to terminal and write to Obsidian daily note.

## Step 0: Load config

Read `~/.claude/life-os-config.md`. Parse all settings into variables.

For each `SOURCE_*` set to `disabled`, **skip that source entirely** — do not call its APIs, do not show its section in output.

If the config file does not exist, print:

```
❌ Config not found. Copy life-os-config.md from the repo to ~/.claude/ and fill in your values.
```

Then stop.

Variable reference:
- `{{OBSIDIAN_VAULT}}` — vault root path
- `{{SECOND_BRAIN}}` — brain-dump directory (only if SOURCE_SECOND_BRAIN: enabled)
- `{{SLACK_USER_ID}}` — your Slack member ID
- `{{SLACK_CHANNELS}}` — list of channels to monitor
- `{{LINEAR_PREFIX}}` — ticket prefix (e.g. MLS)
- `{{STOP_TIME}}` — daily shutdown time
- `{{LANGUAGE}}` — output language (`zh` or `en`)

## Step 1: Collect data from 9 sources

Determine today's date and yesterday's date in `YYYY-MM-DD` format. Then collect data from all enabled sources below. **Run independent sources in parallel** (use parallel tool calls where possible).

### Source 1 — Google Calendar

> Skip this entire source if `SOURCE_CALENDAR: disabled` in config.

Use `mcp__google-drive__calendar_events_list` to fetch today's events.

- Parameters: set the date range to today only (start of day to end of day)
- Extract from each event: **start time**, **end time**, **title**, **attendees count**
- These go into the `📅 Fixed Time` (or `📅 固定时间` if `{{LANGUAGE}}: zh`) section

### Source 2 — Linear

> Skip this entire source if `SOURCE_LINEAR: disabled` in config.

> Note: MCP tool names may vary — check your Linear MCP server config. The tool names below use the pattern `mcp__plugin_linear_linear__*`.

Use the Linear MCP tool to fetch issues assigned to me.

- Make **two** calls to the list-issues tool:
  1. `assignee: "me"`, `state: "started"` — captures In Progress and In Review
  2. `assignee: "me"`, `state: "unstarted"` — captures Todo, Ready for Work, Backlog
- From the combined results, keep issues where:
  - Status is `In Progress` or `In Review` (regardless of due date), OR
  - Status is `Ready for Work` or `Todo` with due date within 7 days or priority ≤ 2 (High/Urgent), OR
  - Due date is today or overdue (any status)
- Extract: **identifier** (e.g. `{{LINEAR_PREFIX}}-751`), **title**, **priority** (1=Urgent, 2=High, 3=Medium, 4=Low), **due date**, **status**

### Source 3 — GitHub

> Skip this entire source if `SOURCE_GITHUB: disabled` in config.

Run these two Bash commands:

```bash
gh pr list --search "review-requested:@me" --json number,title,url,reviewRequests
```

```bash
gh api notifications --jq '.[] | select(.reason=="review_requested" or .reason=="mention")'
```

- Extract: **PR number**, **title**, **comment count**, **repo name**
- Deduplicate: if a PR appears in both results, merge into one item

### Source 4 — Slack

> Skip this entire source if `SOURCE_SLACK: disabled` in config.

> Note: MCP tool names may vary — check your Slack MCP server config. The tool names below use the pattern `mcp__claude_ai_Slack__*`; your server may use different names.

Fetch recent DMs, @mentions, and monitored channel summaries from the past 48 hours using the Slack MCP tools.

My Slack user ID is `{{SLACK_USER_ID}}`.

**Step 4a: DMs and @mentions (actionable items)**

Run **two** parallel searches using the Slack search tool:

1. DMs to me (last 48h):
   - `query`: `on:{{today}} OR on:{{yesterday}}`
   - `channel_types`: `im,mpim`
   - `sort`: `timestamp`, `sort_dir`: `desc`, `limit`: 20
   - `include_context`: true

2. @mentions in channels:
   - `query`: `<@{{SLACK_USER_ID}}> after:{{yesterday}}`
   - `sort`: `timestamp`, `sort_dir`: `desc`, `limit`: 10

From the DM results:

- Keep channels where **the other person sent something** (skip channels where only I sent the last message)
- Flag unanswered DMs (other person's message is newer than my last reply)

From the @mention results:

- Keep mentions that seem to **require a response** (questions, review requests, action items)
- Deprioritize FYI-only mentions

**Step 4b: Monitored channels (digest summaries)**

Summarize recent activity from the channels listed in `{{SLACK_CHANNELS}}` (configured in `~/.claude/life-os-config.md`). Add your own channels in `life-os-config.md`.

For each channel, run a search for messages in the last 48h:

- `query`: `in:{{channel-name}} after:{{yesterday}}`
- `sort`: `timestamp`, `sort_dir`: `desc`, `limit`: 10
- `include_context`: false (save tokens)

Run searches **in parallel** (batch up to 5 concurrent calls).

Channels should be configured with a priority tier (P0/P1/P2) and a description of what to extract. Group results by tier:

- **P0**: Always include — team decisions, blockers, asks directed at me
- **P1**: Include if noteworthy — design/architecture discussions, release updates
- **P2**: Include only if something stands out — engineering updates, tooling

**Channel ID lookup**: If a channel ID is not known, use the channel-search tool to resolve the name to an ID. Cache discovered IDs for future runs.

**Step 4c: Extract and deduplicate**

- Extract: **channel name**, **sender**, **message preview** (first 80 chars), **timestamp**
- Group multiple messages in the same thread into one item
- If a thread has replies, use the thread-read tool to get context
- Deduplicate across DMs and channel mentions

**Step 4d: Categorize for output**

| Destination | What goes there |
| --- | --- |
| 🔥 Urgent (others waiting) | Unanswered DMs, @mentions requiring response |
| 📬 Pending | FYI threads, informational channel activity |
| 📋 Focused Work | Channel discussions that need my follow-up action |
| 💬 Channel Digest | 2-3 sentence digest per monitored channel with notable activity. P0 always included; P1/P2 only if noteworthy. Informational only. |

**Output for 💬 Channel Digest section:**

For each channel with notable activity, output:

```
- **#channel-name**: [2-3 sentence summary of key discussions/decisions]
```

Skip channels with no notable activity in the last 48h. Order by tier (P0 first, then P1, then P2).

### Source 5 — Gmail

> Skip this entire source if `SOURCE_GMAIL: disabled` in config.

Check if Gmail MCP tools are available (e.g., tools with prefix `mcp__gmail` or similar).

- **If available**: fetch unread emails from today and yesterday
  - Extract: **sender**, **subject**, **date**
  - Group bulk/notification emails (e.g., multiple from same sender) into one summary item
- **If NOT available**: print exactly:
  ```
  ⚠️ Gmail MCP not configured, skipping email.
  ```
  (Use Chinese if `{{LANGUAGE}}: zh`: `⚠️ Gmail MCP 未配置，跳过邮件。`)

### Source 6 — Notion

> Skip this entire source if `SOURCE_NOTION: disabled` in config.

Use `mcp__notion__notion-search` to find tasks assigned to the user.

- Query for tasks or to-do items
- Extract: **title**, **status**, **due date**
- If no relevant tasks found, **skip silently** (do not show an empty section or warning)

### Source 7 — News

> Skip this entire source if `SOURCE_NEWS: disabled` in config.

Fetch top AI/tech/product news. Source list is in `references/news-sources.md`.

**Strategy A — RSS Feeds (primary, low-token)**

Read all RSS URLs from `references/news-sources.md` (the "RSS Feeds" tables).

Use `WebFetch` to fetch all feeds **in parallel**, batched up to 5 concurrent calls:

- Batch 1: HN, Karpathy, DeepMind, Google Research, Ethan Mollick
- Batch 2: LangChain, Stratechery, Lenny, Paul Graham, Astral Codex Ten
- Batch 3: Joel on Software, Sebastian Raschka, fast.ai, Distill.pub, arXiv cs.AI
- Batch 4: Sam Altman, Dwarkesh Patel, Amjad Masad

For each feed:

- Prompt WebFetch to extract **titles and dates only** from the last 48 hours
- Do NOT fetch full article content (saves tokens)
- If a feed fails, skip silently and continue

**Strategy B — Twitter/X via WebSearch (high-signal accounts)**

Twitter has no public RSS. Use `WebSearch` to catch viral/high-engagement tweets from key accounts listed in `references/news-sources.md` (the "Twitter/X Accounts" tables).

Run **two** parallel `WebSearch` calls:

1. Scientists + Engineers:

```
query: "site:x.com (@_akhaliq OR @DrJimFan OR @polynoamial OR @karpathy_out OR @yoheinakajima) today"
```

2. Builders + Visionaries:

```
query: "site:x.com (@VitalikButerin OR @balajis OR @linus_lee) today"
```

- Only keep tweets with **substance** (insights, announcements, paper links) — skip replies, memes, quote dunks
- People who already have RSS (Karpathy, Raschka, Howard, etc.) are covered by Strategy A; only include their tweets if they share something **not on their blog**

**Strategy C — WebSearch (non-RSS, non-Twitter supplement)**

Run **one** `WebSearch` call for sources without RSS or active Twitter:

```
query: "Ilya Sutskever Safe Superintelligence news today"
```

- Extract: **title**, **source**, **URL**

**Curation**

From combined results (Strategy A + B), use AI judgment to select the **top 3–5 items** most relevant to the user. Apply these rules:

Priority order:

1. AI / LLM breakthroughs (new models, major research, tool releases)
2. Product & startup news (launches, pivots, funding)
3. Deep thinking pieces (strategy, industry analysis)
4. Developer tools & DevEx

Selection rules:

- **Daily quota**: 3–5 items total, never more
- **Dedup**: same story across multiple sources = one item, pick best source
- **Recency**: prefer last 24h, allow up to 48h for low-frequency blogs
- **arXiv**: only surface papers with unusually high engagement or from well-known labs
- **HN**: favor posts with high points-to-time ratio (trending)

For each selected item, generate:

- **Title**: headline (keep proper nouns in English; translate to Chinese if `{{LANGUAGE}}: zh`)
- **One-line summary**: why it matters (in `{{LANGUAGE}}`)
- **source**: origin (HN / Karpathy / DeepMind / etc.)
- **link**: original URL

These go into the `📰 News` section. This section is **informational only** — no pomodoro estimate, no checkbox.

### Source 8 — Tracked Topics (topic memory)

> Skip this entire source if `SOURCE_TRACKED_TOPICS: disabled` in config.

Read the tracked topics file:

```
references/tracked-topics.md
```

This file contains topics the user is actively following. For each active topic, run a Slack search to surface recent discussions.

**For each topic:**

1. Run a Slack search with:
   - `query`: the topic's `search_query` value
   - `sort`: `timestamp`, `sort_dir`: `desc`
   - `limit`: 10
   - `include_context`: false
   - `response_format`: `concise`

2. From results, extract:
   - New messages since last `/today` run (use `since` field as cutoff, default: 48h)
   - Key updates, decisions, status changes
   - Any messages that @mention me or request my action

3. Generate a **2-4 sentence summary** of what's new for each topic

**Output:**

These go into a new `🔍 Tracked Topics` section in the plan. Format:

```
🔍 Tracked Topics
  • [topic name]: [2-4 sentence summary of recent activity]
```

- This section is **informational only** — no 🍅, no checkbox
- If a topic has no new activity, show: `[topic name]: No new activity` (or `无新动态` if `{{LANGUAGE}}: zh`)
- If a topic surfaces actionable items for me, also add them to 🔥 or 📋 as appropriate
- Order topics by priority (high first)

**Topic lifecycle:**

- Topics with `status: active` are searched every run
- Topics with `status: paused` are skipped
- Topics with `expires` date in the past are automatically skipped (mark as expired in output)

### Source 9 — Brain Dump (Second Brain todos)

> Skip this entire source if `SOURCE_SECOND_BRAIN: disabled` in config.

Read all pending brain dump items from:

```
{{SECOND_BRAIN}}/
```

**Steps:**

1. Use `Bash` to find all `.md` files (excluding `_index.md`):

   ```bash
   find {{SECOND_BRAIN}} -name "*.md" ! -name "_index.md" | sort
   ```

2. `Read` each file found

3. Extract all unchecked items: lines matching `- [ ]`

4. Parse each item's metadata:
   - **Priority**: `P1` (high), `P2` (medium), `P3` (low)
   - **Type tag**: `#todo`, `#idea`, `#question`, `#reminder`
   - **Source date**: from the filename (`YYYY-MM-DD.md`)

5. Calculate age: days since source date. Flag items older than 7 days as stale.

**Routing to today's plan:**

| Priority | Treatment |
| --- | --- |
| `P1` | Add to `🧠 Brain Dump` section with 🍅 estimate. Also consider for `📋 Focused Work` if it's a concrete task. |
| `P2` | Add to `🧠 Brain Dump` section. Informational — no auto-promotion to work sections. |
| `P3` | Add to `🧠 Brain Dump` section only if < 3 items total. Otherwise skip (stays in brain dump file). |

- `#todo` and `#reminder` items get 🍅 estimates
- `#idea` and `#question` items are informational (no 🍅)
- If zero pending items, skip the section silently

## Step 2: Carry-forward from yesterday

Read yesterday's Obsidian daily note:

```
{{OBSIDIAN_VAULT}}/{{yesterday_YYYY-MM-DD}}.md
```

- Find all lines matching the pattern `- [ ]` (unchecked checkboxes)
- Strip the leading `- [ ] ` and any existing section prefixes like `[Yesterday]` / `[昨日]`
- These items become the `🔄 Carry-forward` section
- If the file does not exist, skip carry-forward silently (no error, no warning)

## Step 2.5: Load weekly context

Read the current week's weekly note to align today's plan with the weekly Outcome:

```
{{OBSIDIAN_VAULT}}/{{YYYY}}-W{{WW}}.md
```

Where `{{WW}}` is the ISO week number (e.g., `W10`).

- Extract: **Outcome** (from `### 🎯 Outcome`), **Projects** (from `### 📋 Projects`), **Non-negotiables** (from `### 🛡️ Non-negotiables`), **Open Windows count**
- If the weekly note does not exist, skip silently — do not show a warning
- This context is used in Step 3 to determine the **One Thing** and to add the weekly reference line

## Step 3: AI Smart Sort

After collecting ALL data, organize items into categories and sort using these dimensions:

### Sorting dimensions

1. **Time urgency**
   - Calendar events go to `📅 Fixed Time`, sorted by start time (chronological)
   - Items with due date today or overdue get higher priority within their category
   - Overdue items should be flagged

2. **Blocking others**
   - Items where **others are waiting on me** go to `🔥 Urgent (others waiting)`:
     - PR review requests
     - Slack @mentions requiring a response
     - Linear issues blocking others
   - Solo/independent work goes to `📋 Focused Work`

3. **Context-switching cost**
   - Group related items together (e.g., multiple comments on the same PR = one item)
   - Adjacent Linear tickets in the same epic = group together
   - Low-priority informational items go to `📬 Pending`:
     - Slack threads that are FYI-only
     - Bulk/notification emails
     - Notion tasks with no due date

### Category assignment

| Category | What goes here |
| --- | --- |
| 📅 Fixed Time | Calendar events (sorted by start time) |
| 🔥 Urgent (others waiting) | PR reviews, Slack @mentions needing response, blocking issues |
| 📋 Focused Work | Linear tickets (In Progress first, then by priority), focused coding tasks |
| 📬 Pending | Informational threads, bulk emails, low-priority Notion tasks |
| 🔄 Carry-forward | Unchecked items from yesterday's daily note |
| 🧠 Brain Dump | Pending P1/P2 items from Second Brain (P1 with 🍅, P2 informational) |
| 💬 Channel Digest | Digest of monitored Slack channels (informational, no 🍅, no checkbox) |
| 🔍 Tracked Topics | Tracked topic updates from Slack (informational, no 🍅, no checkbox) |
| 📰 News | Curated news items (informational, no 🍅, no checkbox) |

If `{{LANGUAGE}}: zh`, use the Chinese section names instead:
📅 固定时间 / 🔥 紧急（别人在等） / 📋 专注工作 / 📬 待处理 / 🔄 遗留 / 🧠 Brain Dump / 💬 频道动态 / 🔍 关注话题 / 📰 今日资讯

### Pomodoro estimates

Assign a 🍅 pomodoro estimate to every item:

| Estimate | Duration | Use for |
| --- | --- | --- |
| 0.5 | ~12 min | Quick reply, skim email, short review |
| 1 | ~25 min | Standard PR review, respond to thread, small task |
| 2 | ~50 min | Medium coding task, design review prep |
| 3 | ~75 min | Deep focus work, complex implementation |

### Capacity calculation

Calculate:

- **Total pomodoros**: sum of all 🍅 estimates
- **Meeting hours**: sum of calendar event durations
- **Available hours**: `8h - meeting hours`
- **Available pomodoros**: `available hours * 60 / 25`
- If total pomodoros > available pomodoros, show an overload warning in the output

### One Thing + Stop Time + Circuit Breaker

After sorting all items, determine these three elements (shown at the TOP of the output, before all sections):

**One Thing**: The single task that most advances this week's Outcome (from weekly note). If no weekly note exists, pick the highest-priority item from 🔥 Urgent or 📋 Focused Work.

**Stop Time**: Use `{{STOP_TIME}}` from config. User can override via weekly note Non-negotiables.

**Circuit Breaker**: Auto-generate an interrupt suggestion based on the One Thing's complexity:

- Estimate complexity from Linear ticket size, PR diff, or task nature
- Circuit breaker time = Stop Time minus 3 hours
- For deep debug/design tasks: "If still stuck on [One Thing] at {{circuit_breaker_time}}, force stop → 10 min physical activity → reassess path"
- For communication tasks: "If draft not done by {{circuit_breaker_time}}, send the 80% version"
- (Use Chinese phrasing if `{{LANGUAGE}}: zh`)

## Step 4: Output to terminal

Print the formatted plan to the terminal. Follow the **Terminal Output Template** in `references/output-template.md` exactly.

Key formatting rules:

- **Language**: Respect `{{LANGUAGE}}` setting — if `en`, output in English; if `zh`, output in Chinese with English proper nouns (technical terms, product names, ticket IDs like `{{LINEAR_PREFIX}}-751`, PR numbers like `#2601`)
- **Day names**: Use locale-appropriate format (`Mon`, `Tue`, etc. for `en`; `周一`, `周二`, etc. for `zh`)
- **Omit empty sections**: if a category has zero items, do not show its header
- **Pomodoro estimates**: right-aligned with 🍅 emoji
- **Overload warning**: only show when total pomodoros exceed available capacity
  - Format (`en`): `⚠️  Overloaded! Planned {{total}}🍅 but only ~{{available}}h available — consider deferring low-priority tasks`
  - Format (`zh`): `⚠️  超载！计划 {{total}}🍅 但只有 ~{{available}}h 可用，考虑推迟低优先级任务`

## Step 5: Write Obsidian daily note

> Skip this step if `SOURCE_OBSIDIAN: disabled` in config.

Write the plan to the Obsidian vault daily note:

```
{{OBSIDIAN_VAULT}}/{{YYYY-MM-DD}}.md
```

### Write behavior

| Condition | Action |
| --- | --- |
| File does not exist | Create new file with `## Today's Plan` (or `## 今日计划` if `zh`) as the first section |
| File exists, no plan section | Append the plan section at the end of the file |
| File exists, has plan section | Replace the existing plan section (from the header up to the next `## ` heading or end of file) |

### Obsidian format

Follow the **Obsidian Daily Note Template** in `references/output-template.md` exactly.

Key formatting rules:

- Subsection headers use `###` (one level below `## Today's Plan` / `## 今日计划`)
- **Fixed-time events**: plain bullet list (`-`) — meetings don't need checking off
- **All other items**: checkbox format (`- [ ]`)
- Every item annotated with 🍅 estimate
- Carry-forward items prefixed with `[Yesterday]` (or `[昨日]` if `zh`)
- Omit sections with zero items
- Summary line as blockquote: `> Total: ~{{total}} 🍅 ({{hours}}h) | Available: ~{{available}}h` (or Chinese equivalent if `zh`)

## Error Handling

- If any single data source fails (MCP timeout, API error), log the error briefly and continue with remaining sources. Never let one source failure stop the entire plan.
- If ALL sources fail, print an error message and suggest checking MCP configuration.
- Gmail is expected to be unconfigured initially — this is not an error.
