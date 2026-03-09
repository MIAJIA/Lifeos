---
name: weeklyplan
description: "Use when the user says 'weekly plan', 'weeklyplan', '本周计划', 'plan my week', '周计划', or it's Monday morning planning time, or the start of a new week. Also triggers on '--debug' for mid-week check-in (Wednesday debug mode). Generates a weekly plan through brain dump conversation + RPM blocks, pulling data from Linear, GitHub, Calendar, Slack, Timing App, and Obsidian notes."
---

# /weeklyplan

Generate a weekly plan through structured conversation (brain dump + RPM blocks), pulling data from 6+ sources, and outputting to terminal + Obsidian weekly note.

**Interaction budget**: ~15 min conversation. Do NOT exceed this.
**Tone**: Focused, sprint-like. Brief. "What's the one deliverable?"
**Voice**: NO Tony Robbins hype. The voice is: "I'm training a model: myself." <!-- Customize this tone to suit your style -->

## Arguments

- (none) = full weekly plan (Phase 0–4)
- `--debug` = mid-week check-in (abbreviated, Wednesday mode) — skip to [Mid-Week Debug Mode](#mid-week-debug-mode)

## Full Weekly Plan Flow

```
Phase 0: Load config
Phase 1: Retro — Last week reality check
Phase 2: Capture — Brain dump open windows
Phase 3: Chunk + RPM — Deliverable-level blocks
Phase 4: Auto data pull — MAP material
Phase 5: Match + Output
```

---

## Step 0: Load config

Read `~/.claude/life-os-config.md`. Parse all settings into variables.

For each `SOURCE_*` set to `disabled`, skip that source entirely.

If the config file does not exist, print:
  ❌ Config not found. Copy life-os-config.md from the repo to ~/.claude/ and fill in your values.
Then stop.

Variable reference:
- `{{OBSIDIAN_VAULT}}` — vault root path
- `{{SECOND_BRAIN}}` — brain-dump directory
- `{{SLACK_USER_ID}}` — Slack member ID
- `{{LINEAR_PREFIX}}` — ticket prefix
- `{{STOP_TIME}}` — daily shutdown time
- `{{LANGUAGE}}` — output language (zh or en)

---

## Phase 0: Weekly Retro (automated + 1 question)

Collect data from all sources below. **Run independent sources in parallel.**

### Determine date context

Calculate:

- Current ISO week number and year (e.g., `2026-W10`)
- Last week's ISO week number (e.g., `2026-W09`)
- Monday and Friday dates for this week and last week
- Current month file reference (e.g., `2026-03`)

### Data sources (parallel)

| Source                       | Tool / Method                                                                                                                                                                                                                                                        | What to extract                                                                                                                                                      |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Last week's weekly note      | `Read` file: `{{OBSIDIAN_VAULT}}/YYYY-Wxx.md` (last week)                                                                                                                                                                                                           | Results set vs. achieved, carry-forward items, non-negotiables status, open window count                                                                             |
| Timing App (last 7 days)     | Guard with `SOURCE_TIMING_APP`. If disabled: skip silently and note `⚠️ SOURCE_TIMING_APP disabled — skipping time tracking`. If enabled: `Bash`: `python3 ~/.claude/skills/timingapp-timeline-loader/generate_timeline.py --summary` for each of the 5 weekdays last week (Mon–Fri dates) | Actual hours by project/app, top categories. Aggregate into: Deep work (VSCode/Cursor), Slack, Meetings, Browser, Other                                              |
| Linear (completed last week) | `mcp__plugin_linear_linear__list_issues` with `assignee: "me"`, filter for issues completed/closed last week. **Note: MCP tool names may vary — check your Linear MCP config.**                                                                                     | Closed ticket count, identifiers, titles                                                                                                                             |
| GitHub (merged last week)    | `Bash`: `gh pr list --state merged --author @me --search "merged:>YYYY-MM-DD"` (Monday of last week)                                                                                                                                                                 | Merged PR count, numbers, titles                                                                                                                                     |
| Slack (signal check)         | Guard with `SOURCE_SLACK`. If disabled: skip this source. If enabled: `mcp__claude_ai_Slack__slack_search_public_and_private` with `query: "from:<@{{SLACK_USER_ID}}> after:YYYY-MM-DD"` (Monday of last week), `sort: "timestamp"`, `sort_dir: "desc"`, `limit: 30`. **Note: MCP tool names may vary — check your Slack MCP config.** | Top 3 channels/people by reply volume — am I a "human router"?                                                                                                       |
| Last week's daily notes      | `Read` each daily note `{{OBSIDIAN_VAULT}}/YYYY-MM-DD.md` (Mon–Fri last week)                                                                                                                                                                                       | Unchecked `- [ ]` items = carry-forward candidates                                                                                                                   |
| Brain Dump (Second Brain)    | Guard with `SOURCE_SECOND_BRAIN`. If disabled: skip this source. If enabled: `Bash`: `find {{SECOND_BRAIN}} -name "*.md" ! -name "_index.md"` then `Read` each file                                                                                                 | All unchecked `- [ ]` items with priority (`P1`/`P2`/`P3`) and type tag (`#todo`/`#idea`/`#question`/`#reminder`). P1 items are candidates for this week's projects. |

### Generate mini-retro

Present to user:

```
Last Week — W[xx]
  Outcome set: "[from last weekly note]"
  Status: ✅ Achieved / 🟡 Partial / 🔴 Missed

  Timing reality:
    Deep work (VSCode): XXh
    Slack: XXh
    Meetings: XXh

  Closed: X Linear tickets, X PRs merged
  Unclosed carry-forward: X items
  Brain Dump pending: X items (P1: X, P2: X, P3: X) — oldest: X days

  Slack signal: Top reply targets — #channel1 (Xmsg), @person (Xmsg)
    → [Assessment: normal / potential human-router pattern]
```

If last week's weekly note doesn't exist, skip the outcome comparison and just show data.

### Ask ONE retro question

If `{{LANGUAGE}}` is `zh`:
> "上周回顾 — 如果可以改一件事，你会改什么？"

If `{{LANGUAGE}}` is `en`:
> "Last week retro — if you could change one thing, what would it be?"

Wait for user response. Record their answer as `lesson`.

---

## Phase 1: Capture (brain dump, ~5 min)

### Pre-load: Second Brain pending items

If `SOURCE_SECOND_BRAIN` is enabled: before prompting the user, silently load pending brain dump items (already fetched in Phase 0 data sources). Present them as a reminder:

```
📥 Second Brain pending ({{count}} items):
  🔴 P1: [item description] (from YYYY-MM-DD, {{age}} days ago)
  🟡 P2: [item description] (from YYYY-MM-DD)
  🟢 P3: [item description] (from YYYY-MM-DD)
```

- Flag P1 items older than 7 days as stale: `⚠️ {{age}} days unprocessed`
- If zero pending items, skip this section

Then prompt the user (in `{{LANGUAGE}}`):

If `{{LANGUAGE}}` is `zh`:
> "以上是 Second Brain 里还没处理的 items。加上这些，快速 brain dump：这周脑子里还装着什么？工作、生活都行，倒出来就好。"
> (If no pending items: "快速 brain dump：这周脑子里装着什么？工作、生活都行，倒出来就好。")

If `{{LANGUAGE}}` is `en`:
> "Above are unprocessed items from your Second Brain. Add to those: quick brain dump — what's on your mind this week? Work, life, anything — just get it out."
> (If no pending items: "Quick brain dump — what's on your mind this week? Work, life, anything — just get it out.")

User dumps freely. AI listens, then:

1. Lists all items back to the user (**including** pre-loaded Second Brain items merged with user's new dump)
2. Counts open windows
3. Groups into natural clusters (not forced categories)

Compare window count to last week's count if available.

---

## Phase 2: RPM Blocks (~10 min)

### Frame A: 1 Outcome (must be verifiable)

Show the user:

- Monthly main line (read from `{{OBSIDIAN_VAULT}}/YYYY-MM.md` — the `### This Month` section or `**Main line**` field)
- Brain dump clusters from Phase 1
- Linear tickets due this week (from Phase 4 data, pre-fetched if possible)

Ask (in `{{LANGUAGE}}`):

If `zh`:
> "这周的 ONE outcome 是什么？（必须可验证）"
> "怎么判断做完了？"（verification criteria）
> "这个 outcome 怎么服务你这个月的 main line？"（Purpose — brief, one sentence）

If `en`:
> "What's your ONE outcome this week? (must be verifiable)"
> "How will you know it's done?" (verification criteria)
> "How does this outcome serve your monthly main line?" (Purpose — brief, one sentence)

### Frame B: 2 Projects (that serve the Outcome)

AI suggests project candidates from Linear/GitHub data (tickets in progress, upcoming due dates).

Ask (in `{{LANGUAGE}}`):

If `zh`: > "这两个 project 对吗？要调整吗？"
If `en`: > "Are these the right 2 projects? Any adjustments?"

**Hard rule: Max 2 active projects.** If user tries to add more, push back:

If `zh`: > "最多 2 个 active projects。哪个可以推到下周？"
If `en`: > "Max 2 active projects. Which one can you defer to next week?"

### Friction Check (after Frame B)

Ask (in `{{LANGUAGE}}`):

If `zh`: > "这周所有事情里，哪一件你最想'等准备好了再做'？"
If `en`: > "Which item this week are you most tempted to say 'I'll do it when I'm ready'?"

That action is almost always the highest-leverage surgery point.

Respond (in `{{LANGUAGE}}`):

If `zh`: > "那大概就是应该周一早上第一件做的事。Block 45 min。"
If `en`: > "That's probably the first thing to do Monday morning. Block 45 min."

### Frame C: 3 Non-negotiables (minimum effective dose)

Read defaults from last week's weekly note (the `### 🛡️ Non-negotiables` section). Pre-fill and present:

| Non-negotiable | Template                     | Last week's value     |
| -------------- | ---------------------------- | --------------------- |
| Body           | [frequency] × [activity]     | [from last week]      |
| Relationship   | [frequency] × [quality time] | [from last week]      |
| Retro          | Mid-week + weekend           | Wed debug + Sun retro |

Ask (in `{{LANGUAGE}}`):

If `zh`: > "Non-negotiables 沿用上周的吗？要调整哪个？"
If `en`: > "Same non-negotiables as last week? Any adjustments?"

Wait for confirmation or adjustment.

---

## Phase 3: Auto Data Pull (silent, parallel)

After Phase 2 conversation completes, pull additional MAP material silently:

| Source             | Tool / Method                                                                                                                                                   | Range                                   | Purpose |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------- | ------- |
| Work Calendar      | `mcp__google-drive__calendar_events_list` — this week Mon–Fri                                                                                                   | Meeting load, time blocks               |
| Linear (this week) | `mcp__plugin_linear_linear__list_issues` with `assignee: "me"` — filter for due this week or status `In Progress` / `In Review`. **Note: MCP tool names may vary — check your Linear MCP config.** | MAP material for Projects               |
| GitHub (open)      | `Bash`: `gh pr list --search "review-requested:@me" --json number,title,url` and `gh pr list --author @me --state open --json number,title,url`                 | Blocking items, open PRs                |
| Monthly note       | `Read` file: `{{OBSIDIAN_VAULT}}/YYYY-MM.md`                                                                                                                    | Main line reference for alignment check |

---

## Phase 4: Match + Output

### AI Matching Logic

Match the user's Result/Projects (from Phase 2) against Linear tickets and GitHub PRs:

| Match                               | Treatment                                                                                                              |
| ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Ticket supports this week's Outcome | Listed under the relevant Project in output                                                                            |
| Ticket unrelated to Outcome         | Listed under "Not This Week" (backlog)                                                                                 |
| Outcome has no supporting tickets   | Flag (zh): "这个 outcome 还没有对应的 ticket — 要建一个吗？" / (en): "This outcome has no linked ticket — want to create one?" |
| PR review requests                  | Listed under relevant Project if related, otherwise under Project actions                                              |

Ticket identifiers use `{{LINEAR_PREFIX}}-\d+` pattern (e.g., `MLS-751` if prefix is `MLS`).

### Generate Output

Generate both terminal and Obsidian output following the templates in `references/output-template.md`.

**Terminal**: Print the formatted plan directly.
**Obsidian**: Write to `{{OBSIDIAN_VAULT}}/YYYY-Wxx.md` (e.g., `2026-W10.md`).

### Obsidian Write Behavior

| Condition                                 | Action                                                                                                     |
| ----------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| File does not exist                       | Create new file with full weekly plan template                                                             |
| File exists, no `## Weekly Plan` section  | Append `## Weekly Plan` section at end of file                                                             |
| File exists, has `## Weekly Plan` section | Replace the existing `## Weekly Plan` section (from `## Weekly Plan` to next `## ` heading or end of file) |

---

## Mid-Week Debug Mode

Triggered by `--debug` flag. Wednesday quick check. **No brain dump, no RPM.** Just 3 questions.

### Data Pull (silent, parallel)

| Source                            | Tool / Method                                                                                                            | Purpose                       |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ----------------------------- |
| This week's weekly note           | `Read` file: `{{OBSIDIAN_VAULT}}/YYYY-Wxx.md`                                                                           | Original plan reference       |
| Linear                            | `mcp__plugin_linear_linear__list_issues` with `assignee: "me"`, status `In Progress` or `Done` this week. **Note: MCP tool names may vary — check your Linear MCP config.** | Progress check                |
| GitHub                            | `Bash`: `gh pr list --author @me --state all --json number,title,state,mergedAt`                                         | PR progress                   |
| This week's daily notes (Mon–Wed) | `Read` each daily note `{{OBSIDIAN_VAULT}}/YYYY-MM-DD.md`                                                               | Completed vs. unchecked items |

### Show Progress Dashboard

```
📊 Mid-Week Debug — W[xx] (Wednesday)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

🎯 Outcome: [from weekly note]
  Progress: [X/Y tickets done] | [X PRs merged]
  Status: 🟢 On track / 🟡 At risk / 🔴 Off track

📋 Project 1: [name]
  ✅ Done: [completed items]
  ⏳ In progress: [items]
  ☐ Not started: [items]

📋 Project 2: [name]
  [same breakdown]
```

### Ask 3 Debug Questions (in `{{LANGUAGE}}`)

If `zh`:
1. > "这周的 Outcome 还 on track 吗？"
   > (AI shows Linear/GitHub progress before asking)
2. > "有没有在开新 window 而不是在关旧的？"
   > (AI compares current open items against Monday's plan — flag any new items not in original plan)
3. > "有没有卡超过 2 小时的东西？需要 pull stop hook 吗？"

If `en`:
1. > "Is this week's Outcome still on track?"
   > (AI shows Linear/GitHub progress before asking)
2. > "Are you opening new windows instead of closing existing ones?"
   > (AI compares current open items against Monday's plan — flag any new items not in original plan)
3. > "Anything stuck for more than 2 hours? Do you need a pull-stop?"

### Output

Append `### Mid-Week Debug` section to the existing weekly note. Do NOT replace the weekly plan — append after it.

---

## Life OS Hard Rules (enforced by this skill)

| Rule                               | Enforcement                                                                                                            |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Max 2 active projects per week     | Refuse to accept 3+. Ask user to defer one.                                                                            |
| Closing > starting                 | In retro, count closings vs. new starts. Flag if ratio is wrong.                                                       |
| No "learning without delivering"   | If brain dump has "research X" items, ask (zh): "closing 是什么？" / (en): "What's the closing deliverable?"          |
| Background windows must trend down | Compare this week's count to last week. Warn if trending up.                                                           |
| Friction = signal                  | The thing you're avoiding is probably the surgery point. Surface it in Friction Check.                                 |
| Slack is a signal, not a task      | If `SOURCE_SLACK` enabled: retro shows top 3 reply targets — flag human-router pattern if Slack > 20% of time.         |

---

## Error Handling

- If any single data source fails (MCP timeout, API error, file not found), log the error briefly and continue with remaining sources. Never let one source failure stop the entire plan.
- If Timing App script fails, skip the time tracking section and note: `⚠️ Timing App data unavailable — skipping time tracking.`
- If last week's weekly note doesn't exist, skip retro comparison and start from brain dump.
- If monthly note doesn't exist, skip main line alignment check and note it.
- If ALL sources fail, print an error message and suggest checking MCP configuration.

---

## Language Rules

- **Output language**: Respect `{{LANGUAGE}}` setting. Use `zh` for Chinese with English technical terms (ticket IDs, PR numbers, product names, technical concepts). Use `en` for full English output.
- **Day names**: If `zh`: `周一` through `周日`. If `en`: Monday through Sunday.
- **Week format**: `W10` (ISO week number)
