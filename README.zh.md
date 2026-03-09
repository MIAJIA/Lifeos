# Life OS — Claude Code 规划 Skill 套件

> 为 [Claude Code](https://claude.ai/claude-code) 设计的配置驱动型日/周/月计划 skill 套件。

[English](./README.md) | 中文

## 这是什么

一套 Claude Code skill，帮你生成今日计划、周计划和月度回顾 —— 自动拉取日历、GitHub、Slack、Linear 等你已在用的工具数据。每个数据源都是可选的，按需启用即可。

## 包含的 Skill

| Skill | 触发词 | 功能 |
|-------|--------|------|
| `today` | `/today` | 并行拉取 7+ 个数据源，AI 按紧迫度排序，输出今日计划到终端 + Obsidian 日记 |
| `close-today` | `/close-today` | 下班回顾：自动检测已完成项，生成统计数据和教练反馈 |
| `weeklyplan` | `/weeklyplan` | 周回顾 + RPM 规划（脑倾卸 → 周 Outcome → 2 个 Project → 不可妥协项）|
| `monthlyplan` | `/monthlyplan` | 月度回顾 + RPM 循环，含时间统计和任务清理 |

## 环境要求

- [Claude Code](https://docs.anthropic.com/claude-code)（Claude Code CLI）
- `gh` CLI（用于 GitHub 数据源）
- 根据启用的数据源，可能需要对应的 MCP（见下方数据源表格）

## 安装

1. Clone 本仓库：
   ```bash
   git clone https://github.com/MIAJIA/Lifeos.git
   cd Lifeos
   ```

2. 复制配置模板：
   ```bash
   cp life-os-config.md ~/.claude/life-os-config.md
   ```

3. 编辑 `~/.claude/life-os-config.md` —— 填写你的 Obsidian vault 路径，启用你用到的数据源。

4. 复制 skill 到 Claude skills 目录：
   ```bash
   cp -r today close-today weeklyplan monthlyplan ~/.claude/skills/
   ```

5. 重启 Claude Code（或开一个新会话）。

## 配置

打开 `~/.claude/life-os-config.md`，设置以下内容：

- `OBSIDIAN_VAULT` — Obsidian vault 根目录路径（用于写入日记/周记/月记）
- `SOURCE_*` — 各数据源开关，`enabled` 或 `disabled`
- `STOP_TIME` — 每日下班时间（用于计划容量计算）
- `LANGUAGE` — 输出语言：`zh` 中文，`en` 英文

最简配置示例（只用 GitHub + Obsidian）：
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
LANGUAGE: zh
```

## 数据源

| 数据源 | 需要 | 配置项 |
|--------|------|--------|
| Google Calendar | Google Drive MCP | `SOURCE_CALENDAR` |
| GitHub | `gh` CLI | `SOURCE_GITHUB` |
| Obsidian | 本地 vault 路径 | `SOURCE_OBSIDIAN` |
| Slack | Slack MCP | `SOURCE_SLACK` |
| Linear | Linear MCP | `SOURCE_LINEAR` |
| Timing App | macOS + Timing App | `SOURCE_TIMING_APP` |
| Notion | Notion MCP | `SOURCE_NOTION` |
| 新闻 | WebSearch / WebFetch | `SOURCE_NEWS` |
| Second Brain | 本地 markdown 文件 | `SOURCE_SECOND_BRAIN` |

## 贡献

在用其他工具？在 `community/sources/` 里添加一个数据源模块，分享给大家。
详见 [CONTRIBUTING.md](./CONTRIBUTING.md)。
