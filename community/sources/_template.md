# Source: [Tool Name]

**Description**: What data this source pulls and why it's useful in a daily/weekly plan.

**Requirements**: What the user needs installed or configured (e.g., a specific MCP server, CLI tool, or API key).

**Config keys**: Add these to `~/.claude/life-os-config.md` to use this source:

```
SOURCE_TOOLNAME: disabled  # Short description of what this source pulls
# TOOLNAME_SETTING: value  # Any additional settings needed
```

## Skill snippet

Add this block to `today/SKILL.md` and/or `weeklyplan/SKILL.md` under the appropriate step:

### Source N — [Tool Name]
> Skip this entire source if `SOURCE_TOOLNAME: disabled` in config.

[Describe what to fetch, what tool/API to call, what to extract, and how to format the output.]

## Output format

[Describe what section this adds to the plan output and what it looks like.]
