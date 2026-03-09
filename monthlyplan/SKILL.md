---
name: monthlyplan
description: "Use when the user says 'monthly plan', 'monthlyplan', '月度计划', '月度回顾', 'monthly retro', or when it is the last day of the month or the first day of a new month. Also trigger when the user asks to review the past month, set monthly direction, or do a monthly retrospective."
---

# /monthlyplan

Monthly planning skill. Runs the full RPM cycle at monthly scale: Retro → Capture → RPM Blocks → Output. Produces a monthly note in Obsidian and a formatted terminal dashboard.

**Interaction budget**: ~30 min conversation (brain dump + 4 questions + retro reflection). Do NOT rush — this is the deepest planning session.

**Tone**: Reflective, coaching. Like a research lead reviewing training logs. "I'm training a model: myself." NOT Tony Robbins hype. Calm, data-driven.

**Language**: Respect `{{LANGUAGE}}` from config. Default: output in the configured language with English technical terms (ticket IDs, product names, tool names). Skill instructions are always in English.

---

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
- `{{MONTHLY_NOTE_FORMAT}}` — monthly note filename format (e.g. `YYYY-MM`)
- `{{WEEKLY_NOTE_FORMAT}}` — weekly note filename format (e.g. `YYYY-Wxx`)

---

## Phase 0: Monthly Retro (automated, ~5 min for user to review)

Determine the current date. Compute the previous month (e.g., if today is 2026-03-01, previous month = 2026-02). Compute the current month for the new plan.

### Data Collection (run all sources in parallel)

#### Source A — Previous Month's Weekly Notes

If `SOURCE_OBSIDIAN` is disabled, skip this source.

Read all weekly notes from the previous month from the Obsidian vault:

```
{{OBSIDIAN_VAULT}}/{{WEEKLY_NOTE_FORMAT}}.md
```

Calculate which ISO weeks fell in the previous month (typically 4-5 weeks). Read each file using `Read`. Extract:
- Weekly outcome (set vs. achieved)
- Carry-forward items (unchecked `- [ ]`)
- Open window counts

If a weekly note doesn't exist, skip silently.

#### Source B — Timing App

If `SOURCE_TIMING_APP` is disabled, skip this source entirely. Do not display any time tracking section in the retro dashboard.

If enabled, run the Timing App timeline generator for each week of the previous month:

```bash
python3 ~/.claude/skills/timingapp-timeline-loader/generate_timeline.py --summary
```

Run with appropriate date ranges covering each week of the previous month. Extract:
- Total hours by project/app
- Top categories (e.g., VSCode, Slack, Meetings, Browser)
- Percentage breakdown

If the script fails or is not available, log the error and continue without Timing data.

#### Source C — Linear

If `SOURCE_LINEAR` is disabled, skip this source.

If enabled, use the Linear MCP tool to fetch issues completed during the previous month. Note: MCP tool names may vary by installation (e.g., `mcp__plugin_linear_linear__list_issues` or similar). Use whichever Linear MCP tool is available:
- `assignee: "me"`, `state: "completed"`
- Filter to issues completed within the previous month's date range

Extract: identifier, title, story points (if available), completion date.

#### Source D — GitHub

If `SOURCE_GITHUB` is disabled, skip this source.

If enabled, run via Bash:

```bash
gh pr list --state merged --author @me --search "merged:>YYYY-MM-DD merged:<YYYY-MM-DD" --json number,title,url,repository
```

Use the first and last day of the previous month as the date range. Extract: PR count, repo names, PR titles.

### Generate Retro Dashboard

Combine all data into a retro dashboard. Display it in the terminal:

```
Last Month Retro — YYYY-MM
━━━━━━━━━━━━━━━━━━━━━━━━━━

Time allocation (Timing App):        ← omit this block if SOURCE_TIMING_APP is disabled
  [App1]:    XXh (XX%)
  [App2]:    XXh (XX%)  ← [flag if concerning, e.g., "is this too high?"]
  [App3]:    XXh (XX%)
  ...

Closings delivered: [count]
  - [closing 1]
  - [closing 2]
  - [closing 3]

Weekly Results hit rate: [x/y] weeks achieved primary outcome

Last month's "bug to fix": [from previous monthly note]
  → Status: [achieved / partial / not addressed]

Last month's "window to close": [from previous monthly note]
  → Status: [closed / still open]
```

To populate "Last month's bug to fix" and "window to close", read the previous month's monthly note (if `SOURCE_OBSIDIAN` is enabled):

```
{{OBSIDIAN_VAULT}}/{{MONTHLY_NOTE_FORMAT}}.md
```

(Substitute the previous month's date values into the format — e.g., `YYYY-MM` → `2026-02`.)

Extract the `**Bug to fix**` and `**Window to close**` entries. If the file doesn't exist, note "No previous monthly note found."

### Retro Reflection

After displaying the dashboard, ask the user ONE question:

> "Looking at this — what's the one pattern you notice?"

Wait for the user's response. Record their observation for inclusion in the output.

---

## Phase 1: Capture (brain dump, ~10 min)

After the retro reflection, prompt the user:

> "Before we plan, let's clear the buffer. Dump everything that's taking up space in your head right now — work, life, worries, ideas, half-finished things. No order, no filter. I'll organize it after."

Wait for the user to dump freely. They may send multiple messages. When they indicate they're done (or after a natural pause), process the dump:

1. **List all items back** to the user in a numbered list
2. **Count**: "You have **N background windows** open right now."
3. **Group** into natural clusters (let the clusters emerge from the content — do NOT force into predefined categories)

---

## Phase 2: RPM Blocks (4 questions + Deprecation Review, ~30 min)

Walk the user through exactly 4 questions, one at a time. Do not skip ahead.

### Q1: "What's the main line this month?" (only 1)

Show the user:
- Their brain dump clusters (from Phase 1)
- The annual plan's current quarter focus (if available from vault — check for a yearly note like `YYYY.md` or `YYYY-Q1.md` in `{{OBSIDIAN_VAULT}}`)
- Last month's trajectory (from the retro)

Ask: **"Based on all this — what's the ONE main line this month?"**

The user must pick exactly ONE. If they try to pick 2+, push back:
> "I can only accept one main line. Which one matters MORE right now? The other goes to Not Doing List."

After they pick one, ask: **"Why does this matter RIGHT NOW?"** (Purpose)

Record: main line + purpose.

### Q2: "What's the one bug to fix?"

Show the user:
- Patterns from the retro (e.g., "Slack took 16% of your time", "only 2/4 weekly outcomes hit")
- Recurring un-closed items from weekly notes
- Last month's bug status

Ask: **"What's the ONE behavioral bug to fix this month?"**

Help the user write a **bug card**:
- **Trigger**: What situation activates the bug?
- **Reaction**: What do you currently do?
- **Cost**: What does it cost you?
- **Alternative action**: What's the new behavior?

Record: bug card (all 4 fields).

### Q3: "What's one closing to deliver?"

Ask: **"What's ONE reusable artifact you'll deliver this month?"**

Check: "Can future-you or someone else USE this artifact?" If the answer is vague ("work on X", "learn about Y"), push back:
> "That sounds like a process, not a closing. What's the artifact? A doc, a template, a shipped feature, a data point, a SOP?"

Record: closing description.

### Q4: "What's one window to close?"

Show the user:
- Brain dump items that have been lingering (appeared in previous months)
- Items from the Not Doing List (if available from previous monthly note)

Ask: **"What's ONE window to close — a project to kill, an input source to cut, a decision to stop deferring?"**

Record: window to close.

### Deprecation Review (after Q4)

Cross-reference: windows opened last month (from weekly notes and brain dump) that did NOT enter the main line this month.

For each zombie window, force a decision:
- **Kill**: delete repo, close tab, unsubscribe, cancel subscription. Concrete action.
- **Defer with expiry**: must have a specific date, max 1 month out.

**"Parking" without a kill date is not allowed** — that's memory leak, not memory management.

If the user tries to "just keep it around":
> "That's indefinite parking. Give it a kill date (max 1 month) or kill it now."

Output: a concrete Deprecation List with actions taken (not intentions).

---

## Phase 3: Output

### Terminal Output

Print the formatted monthly plan to the terminal. Follow the template in `references/output-template.md` — the "Terminal Output Template" section.

### Obsidian Monthly Note

If `SOURCE_OBSIDIAN` is disabled, skip this step.

If enabled, write the plan to:

```
{{OBSIDIAN_VAULT}}/{{MONTHLY_NOTE_FORMAT}}.md
```

(Substitute the current month's date values into the format — e.g., `YYYY-MM` → `2026-03`.)

Follow the template in `references/output-template.md` — the "Obsidian Monthly Note Template" section.

#### Write Behavior

| Condition | Action |
|-----------|--------|
| File does not exist | Create new file with full monthly note content |
| File exists, no `## Monthly Plan` section | Append the monthly plan sections at the end of the file |
| File exists, has `## Monthly Plan` section | Replace the existing `## Monthly Plan` section (from `## Monthly Plan` up to the next `## ` heading or end of file) |

---

## Life OS Hard Rules (enforced during all phases)

| Rule | Enforcement |
|------|-------------|
| Max 1 main line per month | Refuse to accept 2+. Force the user to pick ONE. |
| closing > starting | Retro counts closings vs. new starts. Flag if starts > closings. |
| No "learning without delivering" | If brain dump has "research X" / "learn Y", ask: "what's the closing?" |
| Zombie windows must die | Deprecation Review: kill or set expiry, no indefinite parking. |
| Background windows must trend down | Compare this month's window count to last month's. Flag if trending up. |
| Evidence-based, not "I feel like I know" | Retro uses actual data (Timing, Linear, GitHub), not vibes. |

---

## Error Handling

- If any single data source fails (Timing App script error, MCP timeout, file not found), log the error briefly and continue with remaining sources. Never let one source failure stop the entire plan.
- If `SOURCE_TIMING_APP` is enabled but the script at `~/.claude/skills/timingapp-timeline-loader/generate_timeline.py` is not found, skip Timing data and note: `⚠️ Timing App script not found, skipping time analysis.`
- If `SOURCE_LINEAR` is enabled but Linear MCP is unavailable, skip Linear data and note: `⚠️ Linear MCP not connected, skipping ticket data.`
- If `SOURCE_GITHUB` is enabled but GitHub CLI is not authenticated, skip GitHub data and note: `⚠️ GitHub CLI not authenticated, skipping PR data.`
- If the previous monthly note doesn't exist, skip "last month's bug/window" references and note it.
- If ALL sources fail, still proceed with the brain dump + RPM blocks (the human conversation is the most valuable part).
