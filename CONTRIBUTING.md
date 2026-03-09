# Contributing to Life OS

Thanks for using Life OS! Here are the ways you can contribute.

## Ways to Contribute

- **Add a data source** — Use Jira, Toggl, Bear, Todoist, Apple Reminders, or another tool? Add a source module so others can use it too.
- **Share a config example** — Share your `life-os-config.md` setup as a reference for others with similar tool stacks.
- **Add a new skill** — Built a `yearlyplan`, `quarterlyplan`, or something else? Add it to the suite.
- **Translate** — Add translated output templates or bilingual prompts for your language.

## Adding a Data Source

1. Create a new file at `community/sources/your-tool-name.md` using the template in `community/sources/_template.md`.
2. Fill in the description, requirements, config keys, and the skill snippet.
3. Submit a PR with a short description of what the source pulls and why it's useful.

**Tips:**
- Keep the skill snippet self-contained — it should work when copy-pasted into today/SKILL.md or weeklyplan/SKILL.md.
- Add any new `SOURCE_*` keys to the "Config keys" section so users know what to add to `life-os-config.md`.
- Test your source before submitting.

## Sharing a Config Example

1. Copy your `~/.claude/life-os-config.md` to `community/configs/your-setup-name.md` (e.g., `minimal-github-obsidian.md`, `full-linear-slack.md`).
2. Remove any personal values (IDs, paths, channel names).
3. Add a short comment at the top describing your setup.
4. Submit a PR.

## Adding a New Skill

1. Create a new directory at the repo root (e.g., `yearlyplan/`).
2. Follow the same structure as existing skills: `SKILL.md` + optional `references/`.
3. Add a Step 0 config-loading block (copy from any existing skill).
4. Update README.md to include the new skill in the Skills table.
5. Submit a PR.

## PR Guidelines

- One change per PR (one source, one skill, one config example).
- Test your skill before submitting — run it with Claude Code and confirm it works.
- Keep descriptions short and practical.

## Questions?

Open an issue on GitHub.
