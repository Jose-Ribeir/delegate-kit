---
name: subagents
description: "Roster of the specialized subagents the main agent can spawn, and when to use each. Four types: thinker (pure reasoning, no tools), super-thinker (pure reasoning, top-tier model, no tools — for the hardest calls), researcher (flow-mapping, never dumps whole files), simple-tasks (mechanical chores incl. commits/pushes AND cheap multi-hop context-saving work). Use when deciding how to delegate work — pick the right agent, give it the right brief. Offload many-hop low-judgment work to simple-tasks to keep the expensive main context clean."
---

# /subagents

The roster of specialized subagents you can spawn, what each is for, and how to brief it. Use this to route work to the right agent instead of defaulting to a generic one. Each is a real `subagent_type` (defined in this plugin's `agents/` directory) with its tools and model already pinned — you don't set the model, just pass `subagent_type` and a good prompt.

## The roster

| Agent | Model | Tools | Use it for |
|-------|-------|-------|------------|
| **thinker** | opus | none | Deep reasoning over context you already have: analysis, trade-offs, planning, debugging-by-reasoning, hard decisions. |
| **super-thinker** | fable | none | Same as thinker, but the top tier — reach for it when the reasoning is hardest or the call is highest-stakes and you want maximum depth. |
| **researcher** | sonnet | Read/Grep/Glob/Bash (+ Serena & graphify when installed) | Understanding how something works: the full flow through the system and the nodes involved. |
| **simple-tasks** | haiku | Read/Grep/Glob/Bash (+ Serena when installed) | Mechanical chores (commits, pushes, commands, builds, file ops) **and** multi-hop low-judgment work — chains of dependent steps that would otherwise burn the expensive main context. |

> **Two modes for researcher & simple-tasks.** They work with **zero setup** using built-in `Read`/`Grep`/`Glob`/`Bash` (Standard mode). If the project has [Serena](https://github.com/oraios/serena) (symbol-precise navigation) and/or graphify (flow/structure graphs), they automatically prefer those for cheaper, more precise navigation (Power mode). Nothing to configure either way.

---

## thinker — the reasoning engine

**What it is:** pure thought. It reasons only over the context in your prompt and has **no tools** — it cannot read files, search, or run anything.

**Use when** the hard part is *thinking*, not *finding*: weighing trade-offs, untangling a tricky decision, reasoning through a bug from facts you already have, designing an approach in your head.

**Do NOT use it** to gather information — it can't. If facts are missing, get them (researcher) first, then hand them to thinker.

**Briefing rule — pack the context.** thinker only knows what you tell it. Put every relevant fact, constraint, code snippet, and option into the prompt. A thin brief gets thin reasoning. Ask it for a conclusion *and* the reasoning path.

## super-thinker — the top-tier reasoning engine

**What it is:** the same pure-thought engine as thinker, but running on **Fable** — the strongest reasoning model. Identical constraints: **no tools** (can't read, search, or run), reasons only over the context you hand it. (No access to Fable? Change the agent's `model:` field to one you have.)

**Use when** the reasoning is genuinely hard or the call is high-stakes and you want maximum depth: a subtle architectural trade-off, an intricate multi-factor decision, a gnarly bug you must reason out from the facts, a plan where a wrong call is expensive. When plain thinker would do, use thinker; reach for super-thinker when the extra reasoning depth is worth it.

**Briefing rule — same as thinker, and it matters even more here.** super-thinker only knows what you tell it, so pack every relevant fact, constraint, snippet, and option into the prompt. The deeper the model, the more it rewards a complete brief — and the more a missing fact quietly caps the quality. Ask for a conclusion *and* the reasoning path.

## researcher — the flow mapper

**What it is:** the research specialist. It maps how a system works — the end-to-end flow and the symbols/modules/services (nodes) involved.

**The defining constraint:** it **never dumps a whole file.** In Power mode it navigates with **graphify** (`query` / `path` / `explain`) for flow and structure and **Serena symbol tools** for targeted reads; in Standard mode it uses **Grep** to locate and **Read** for targeted line ranges only. Either way it locates precisely and reads just what it needs, which keeps research cheap and focused.

**Use when** you need to understand something before acting: "how does X flow end to end", "where does Y live and what touches it", "trace the path from A to B".

**Briefing rule — name the target and the question, and include the project root.** Tell it exactly what to understand and what to return, and pass the project root (e.g. `cwd: /path/to/project`) so it scopes its navigation correctly (and, in Power mode, points graphify/Serena at the right project). It comes back with a structured flow report: ordered path, key nodes, connections, and `path:line` anchors for everything — so you (or the next agent) can act from the report alone.

## simple-tasks — the executor

**What it is:** the cheap, fast hands. It executes mechanical, well-defined work and reports back condensed but complete. It runs all the shell/CLI work and **fully owns git, including commits and pushes.**

**Use it for two kinds of work:**
1. **Clear mechanical chores** — commit & push, run a build, move files, run a script. The thinking is already done; this is execution.
2. **Multi-hop, context-saving work** — tasks that take many sequential, dependent tool calls (read a value → follow it to the next file → gather a fact across a dozen places → run a command and act on its output). This is the *main reason to reach for it even when the route isn't fully known*: each hop's tool output lands in **its** context, not yours, so the expensive main context stays clean. Anything low-judgment that would otherwise have you make 15 reads/greps in a row is a candidate to offload here.

**The line that matters:** simple-tasks does *finding and doing*, never *deciding*. Navigation, collection, and mechanical execution toward a clear goal — yes. Design calls, correctness judgments, choosing between approaches, or guessing intent — no; for those it must stop and report. If the hard part is *thinking*, that's **thinker**; if it's *understanding a flow to brief someone*, that's **researcher**.

**Briefing rule — give it the goal, the route, and the project root.** For pure chores, hand it a complete ordered route with exact paths/commands — the more precise, the more reliable. For multi-hop work, give it a crisp goal, the starting point, and the stopping condition (what "done" looks like, what to bring back); it will chase the trail itself. For commits, give the exact message (verbatim, with any required trailer) and what to stage. Pass the project root (e.g. `cwd: /path/to/project`) so it operates in the right place. Always tell it to stop and report if it hits a real decision point.

**Reporting contract:** it reports condensed but with all the important content — commands run, exit status, commit SHA, files changed, and for gather tasks the **findings with exact `path:line` anchors** so you can act without re-walking the trail. It pastes **verbatim output for anything that failed**, and must never claim success for a command it didn't actually run and verify.

---

## Picking the right one

- Need to **think** about what you already know → **thinker** (or **super-thinker** on Fable when the call is hardest / highest-stakes).
- Need to **find out / understand** how something works, with a flow report to brief the next step → **researcher**.
- Need to **do** a clear mechanical task, **or** grind through many low-judgment hops to keep your own context clean → **simple-tasks**.

**Default to offloading.** If a task is low-judgment but would cost *you* (the expensive main agent) a long string of reads/greps/commands, hand it to **simple-tasks** with a clear goal and stopping condition — even if you can't pre-write every step. The hops burn its cheap context instead of yours, and you get back a condensed report. Reserve your own context for the decisions only you can make.

**researcher vs. simple-tasks for multi-hop:** pick **researcher** when the output is *understanding* — a structured flow map of how a system works. Pick **simple-tasks** when the output is *a concrete result or collected facts* and the path is mechanical (gather all X, apply Y everywhere, run Z and report).

A common chain: **researcher** maps the flow → **thinker** reasons over those findings to decide the approach → **simple-tasks** executes and commits. Spawn each with a brief sized to its job; pass concrete pointers (paths, anchors, the exact route) so it doesn't re-discover what you already know.
