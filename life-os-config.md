# Life OS Config
# Copy this file to ~/.claude/life-os-config.md and fill in your values.
# Skills: /today, /close-today, /weeklyplan, /monthlyplan

## Paths

# Your Obsidian vault root directory (required if SOURCE_OBSIDIAN: enabled)
OBSIDIAN_VAULT: /Users/yourname/Documents/YourVault

# Your Second Brain brain-dump directory (optional, used if SOURCE_SECOND_BRAIN: enabled)
# SECOND_BRAIN: /Users/yourname/Documents/your-second-brain/brain-dump

## Data Sources
# Set each source to "enabled" or "disabled"

SOURCE_CALENDAR: enabled       # Google Calendar via MCP
SOURCE_GITHUB: enabled         # GitHub PRs and notifications (requires `gh` CLI)
SOURCE_OBSIDIAN: enabled       # Daily notes and weekly notes
SOURCE_LINEAR: disabled        # Linear project management (requires Linear MCP)
SOURCE_SLACK: disabled         # Slack DMs, mentions, channels (requires Slack MCP)
SOURCE_TIMING_APP: disabled    # Timing App time tracker (macOS only)
SOURCE_NOTION: disabled        # Notion tasks (requires Notion MCP)
SOURCE_NEWS: enabled           # Curated AI/tech news via RSS + web search
SOURCE_SECOND_BRAIN: disabled  # Local brain-dump markdown files

## Slack Settings (only used if SOURCE_SLACK: enabled)

# Your Slack user ID — find it in Slack: click your avatar → Profile → ⋮ → Copy Member ID
# SLACK_USER_ID: UXXXXXXXXX

# Channels to monitor. Format: "- display-name: CHANNEL_ID"
# SLACK_CHANNELS:
#   - my-team: C0XXXXXXXXX
#   - announcements: C0XXXXXXXXX

## Linear Settings (only used if SOURCE_LINEAR: enabled)

# Your ticket prefix (e.g. for MLS-123, prefix is MLS)
# LINEAR_PREFIX: MLS

## Preferences

# Daily shutdown time (used for capacity planning and circuit breaker)
STOP_TIME: 18:00

# Note file naming formats
DAILY_NOTE_FORMAT: YYYY-MM-DD         # e.g. 2026-03-08.md
WEEKLY_NOTE_FORMAT: YYYY-Wxx          # e.g. 2026-W10.md
MONTHLY_NOTE_FORMAT: YYYY-MM          # e.g. 2026-03.md

# Output language: zh (Chinese) or en (English)
LANGUAGE: en
