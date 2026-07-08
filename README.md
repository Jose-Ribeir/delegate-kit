# delegate-kit

**A curated bench of 4 specialist subagents for Claude Code — and the playbook for when to delegate to which.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Claude Code plugin](https://img.shields.io/badge/Claude%20Code-plugin-8A2BE2)
![Zero setup](https://img.shields.io/badge/setup-zero--config-brightgreen)

Most agent collections give you a hundred role-experts (`react-expert`, `sql-expert`, …). **delegate-kit is the opposite: a small orchestration layer.** Four subagents organized not by domain but by *reasoning depth and context cost*, plus a `/subagents` skill that teaches the main agent **when to hand work off, and to whom** — so your expensive main context stays clean and your hard thinking gets the strongest model.

---

## Why you want this

Every long Claude Code session drowns in cheap work: fifteen greps to trace a flow, a dozen reads to gather facts, a build, a commit. Doing it inline burns the one thing you can't get back — your main context window. delegate-kit fixes the workflow, not the model:

- **Offload the grind.** Multi-hop, low-judgment work goes to a cheap agent; the tool output lands in *its* context, not yours.
- **Reserve depth for decisions.** Hard trade-offs go to a pure-reasoning agent on the strongest model, with no tool noise.
- **Map before you touch.** Understand a system via a flow report with `path:line` anchors instead of pasting whole files.

## The roster

| Agent | Model | Best at |
|-------|-------|---------|
| **super-thinker** | fable | Top-tier pure reasoning for the hardest, highest-stakes calls — subtle trade-offs, intricate plans, where depth beats speed. No tools. |
| **thinker** | opus | Everyday deep reasoning over context you already have — trade-offs, planning, debugging-by-reasoning. No tools, pure thought. |
| **researcher** | sonnet | Mapping how a system works end-to-end. Never dumps whole files; returns an anchored flow report. |
| **simple-tasks** | haiku | Mechanical chores (commits, pushes, builds, file ops) **and** cheap multi-hop context-saving work. |

The **`/subagents`** skill is the playbook: it gives the main agent the roster, briefing rules for each agent, and a decision guide for picking the right one — including the classic chain **researcher maps → thinker decides → simple-tasks executes & commits**.

## Install

```
/plugin marketplace add Jose-Ribeir/delegate-kit
/plugin install delegate-kit
```

Then reload plugins (`/reload-plugins`) if needed. You now have:

- the **`/delegate-kit:subagents`** skill (the playbook), and
- four spawnable agents: `thinker`, `super-thinker`, `researcher`, `simple-tasks`.

> **Prefer the bare `/subagents` command?** Copy `skills/subagents/` into `~/.claude/skills/` and `agents/*.md` into `~/.claude/agents/`. Manual (non-plugin) skills aren't namespaced, so the command is `/subagents`.

## Usage

Ask the main agent to route work, or invoke the playbook directly:

```
/delegate-kit:subagents
```

Or just delegate in natural language and let Claude pick:

- *"Trace how a request flows from the API route to the DB write."* → **researcher** returns an anchored flow report.
- *"Given that flow, should we cache at the route or the service layer? Reason it through."* → **thinker** weighs the trade-off.
- *"Now apply the change, run the tests, and commit it."* → **simple-tasks** executes and owns the commit.

## Two modes — zero-config, or supercharged

**Standard mode (default, nothing to install).** `researcher` and `simple-tasks` navigate with built-in `Grep` / `Glob` / `Read` / `Bash`. Works everywhere, immediately.

**Power mode (optional).** If your project has either of these, the agents automatically prefer them for cheaper, symbol-precise navigation:

- **[Serena](https://github.com/oraios/serena)** — an MCP server for symbol-level code navigation (read one symbol, not a whole file).
- **graphify** — flow/structure knowledge graphs of your codebase (`graphify query/path/explain`).

No flags to flip — the agents use whatever is present and fall back cleanly when it isn't.

## Customizing

- **Models** are pinned per agent in `agents/*.md` frontmatter (`model:`). Change any to a model you have access to — e.g. if you don't have `fable`, set `super-thinker` to `opus`.
- **Add your own agents** alongside these and reference them from your own copy of the skill.

## How it's structured

```
delegate-kit/
├── .claude-plugin/
│   ├── marketplace.json     # marketplace registry (one plugin, source ./)
│   └── plugin.json          # plugin manifest
├── agents/                  # the 4 subagent definitions
│   ├── thinker.md
│   ├── super-thinker.md
│   ├── researcher.md
│   └── simple-tasks.md
└── skills/
    └── subagents/
        └── SKILL.md         # the /subagents delegation playbook
```

## Contributing

Issues and PRs welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Good first contributions: a new specialist agent that fills a real gap, or sharper briefing guidance in the skill.

## License

[MIT](LICENSE) © José Ribeiro
